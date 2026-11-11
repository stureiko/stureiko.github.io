# Cost-Driven Infrastructure

*Why renting GPUs often turns out to be excessive, and how to design a payment model with Infracost*

## Introduction

A typical task looks like this: an ML platform team gets a budget and, to avoid depending on capacity availability, rents a GPU cluster for a year. The decision seems reasonable: the hourly price is lower than on-demand, there are no queues, the engineers are calm, and the cost has been forecast and included in the budget. Six months later someone from audit sees that the cluster is busy about a third of the time, and the rest of the time the nodes sit powered on, train nothing, and cost exactly the same. The bill covers every hour of the month, while useful work was done in only some of them.

Nobody in this story made a technical mistake. The cluster is configured correctly, jobs run, models train. The mistake is architectural: infrastructure cost was treated as a fixed line in the budget, not as a function of the real workload profile. We are used to designing reliability through SLAs, redundancy and fault tolerance, and security through threat models and access control. Cost, too often, remains a side effect that is discovered after the fact.

I suggest treating cost as a designed property of the system. The thesis is simple: GPU load during model training is fundamentally uneven, so renting capacity permanently under long commitments is, in most cases, needlessly expensive. It is better to plan no further than a month ahead, commit only to the minimum necessary baseline, and buy peaks on demand. The decision is made by calculation, not by intuition, and the calculation runs on infrastructure described as code. Infrastructure is managed through controlled changes, so we always have a baseline for calculating cost and for controlling cost changes retrospectively. Infracost is one of the most popular tools for this kind of control.

I call this approach cost-driven infrastructure, and the process of revising cost and its forecast should become as regular a project process as tech-debt assessment or metric-drift control.

## Why GPUs Sit Idle

Model development is not like running a web service with a more or less smooth stream of requests. It is a "data - experiment - evaluation - debugging" cycle, and each phase needs compute capacity to a different degree. Data preparation and cleaning run mostly on CPU and require first of all storage volume. An experiment needs a GPU, but only for the duration of the run itself. Then comes evaluating results, analyzing errors and fixing code, and for all that time the expensive accelerators do nothing. Before a release, demand jumps: several variants must run at once, metrics must be rechecked, training must be repeated on corrected data. After the release, things go quiet again.

This picture is supported not only by practice but also by public research. A team at Shanghai AI Lab analyzed traces from the Acme clusters, which run 4,704 A100 GPUs, and described them in a paper published at NSDI 2024 [1]. Median GPU utilization in LLM development jobs was very high, about 99%, but the load distribution is bimodal: accelerators are either almost fully busy or almost idle, as happens during model evaluation and debugging. The same paper notes that network adapters are idle more than 60% of the time. In other words, "average load" describes such a system poorly, because it has no typical state. There are two extreme modes, and the system switches between them.

The second observation concerns job size. The authors of SAKURAONE, who describe workload dynamics in a single-tenant LLM development environment, show that large jobs use GPUs well, while small jobs on one or two nodes have a median utilization of only 17–23% [2]. They also give a figure that should make us stop and think: canceled jobs account for 73.5% of all occupied GPU time. This does not mean cancellation is always a mistake. For a researcher, stopping a failed experiment quickly is exactly the right thing to do. But the infrastructure pays for it, and pays at a rate calculated for useful work.

A third example shows that even a busy GPU is not always doing useful work. According to a Backblaze publication based on Adobe's data, at first profiling about two thirds of GPU time went to waiting for data [3]. After the input pipeline was optimized, utilization rose to about 80%, which is close to the practical ceiling. That is a separate topic, solved by engineering inside the pipeline rather than by purchasing capacity, but it is a reminder to measure utilization before deciding to buy capacity.

This leads to a practical distinction between two modes. Large pretraining runs occupy many nodes for long periods without pauses, their utilization is high, and long commitments can be justified for them. Small experimental jobs, by contrast, are short, irregular and often canceled, and their utilization is low. These two modes cannot be paid for in the same way. If we buy round-the-clock capacity for experimental work, we pay the "continuous operation" rate for work that is never continuous.

## Three Ways to Pay for GPUs

To reason about a mixed model, we need a clear view of the payment options. I will use AWS as the best-documented case. Other providers work similarly, with different names.

**On-demand.** Hourly payment with no commitment. This is the most expensive rate per hour, but also the most flexible: a node starts when needed and stops costing money once it is stopped. The main limitation is not price but availability. Plain on-demand gives no guarantee that the required number of top-tier GPUs will be found at the right moment in the right region.

**Short-term reservation.** In AWS this is Capacity Blocks for ML. A block can be booked for one to 14 days, or for a period in multiples of seven days up to 182 days, and booking is possible up to eight weeks in advance [4]. Payment is made at booking time, a block cannot be canceled, and up to 64 instances are available in one block. The P4d, P4de, P5, P5e, P5en and P6 families are supported. The price depends on demand, is published per accelerator-hour, and usually sits between on-demand and a one-year reservation [5]. The main advantage of a block is that it guarantees availability for a specific window, which is exactly the problem on-demand does not solve. Many other providers have an analogue in the form of monthly committed plans.

**Long-term commitments.** Reserved Instances and Savings Plans for one or three years. The provider gives a discount in exchange for your paying for the capacity whether you use it or not. For p4d.24xlarge in us-east-1 with a one-year reservation and no upfront payment, price aggregators give a reference of about $13.92 per hour, which is roughly a 37% discount to on-demand [6].

!!! note "Check prices against the official price list"
    The reservation prices in this article come from aggregators and refer to 2026. They are a reference for illustrating the method, not data for a budget. Before planning a purchase, check the figures against the provider's current official price list and verify the terms of the specific plan.

Here are the three options in one table.

| Criterion | On-demand | Short-term reservation | Long-term commitment (1–3 years) |
| --- | --- | --- | --- |
| Price per hour | Highest | Between on-demand and a one-year reservation | Lowest |
| Flexibility | Full | Limited to the booking window | Low |
| Availability guarantee | None | Yes, for the block period | Depends on the reservation type |
| Planning horizon | Not needed | Days and weeks, up to 8 weeks ahead | Years |
| Underuse risk | Practically none | You pay for the whole block, no cancellation | You pay for the whole term regardless of load |

The table shows that cheap and flexible never come together, and they are two sides of the same coin. So the question is not "which rate" but "which part of the load to which rate". It can only be answered by calculation, and calculation needs a convenient tool.

## Cost as Code: Infracost

If infrastructure is described in Terraform, its cost can be calculated before it is applied. This is the task Infracost solves [7]. It reads the configuration code, finds resources, requests prices from the provider and produces a monthly estimate broken down by resource. This turns "how much will it cost" from a guess into a verifiable number that you can get at review time, not at the end of the month.

Here is a minimal example: one node with eight A100 GPUs.

```hcl
resource "aws_instance" "train_node" {
  ami           = var.dlami_id
  instance_type = "p4d.24xlarge"

  tags = {
    team    = "nlp-research"
    project = "ranker-v3"
    env     = "training"
  }
}
```

For this configuration, Infracost returns an on-demand estimate of $21.96 per hour, or $16,029 per month at 730 hours. Storage, network and other resources are counted as separate lines. For reference, here are the prices of several GPU instances obtained the same way (AWS, us-east-1, Linux, 730 hours per month).

| Instance | GPU | $/hour | $/month |
| --- | --- | --- | --- |
| p5.48xlarge | 8× H100 | 55.04 | 40,179 |
| p4d.24xlarge | 8× A100 40 GB | 21.96 | 16,029 |
| g5.12xlarge | 4× A10G | 5.67 | 4,141 |
| g6.12xlarge | 4× L4 | 4.60 | 3,359 |
| g5.xlarge | 1× A10G | 1.01 | 734 |

Running it in CI comes down to one command.

```bash
infracost scan
```

The practical value is not in the number itself but in where it appears. If you run the scan in CI, every pull request shows the cost delta: a change that adds two more p4d nodes will show the bill growth right in the code discussion, while it can still be rejected. On top of this, Infracost supports budgets and policies that can block a change if it goes beyond acceptable limits. Tags let you split costs by team and project, which we will need at the end of the month to compare plan and actual.

The current version of Infracost calculates on-demand rates. Reservation and Capacity Blocks prices come from the provider's price lists, not from the tool's estimate. Older versions let you specify reservations through a usage file [8], but I would not rely on that for a mixed scheme. So the "baseline plus peaks" calculation is done on top of the Infracost estimate: the hourly on-demand price comes from the tool, the commitment price from the provider's price list, and the arithmetic itself is done in a simple spreadsheet or a small script. This is not a flaw in the approach but a division of responsibility: Infracost handles what it knows well, namely resource descriptions and current rates, and the decision on commitment structure stays with a person.

## The "Baseline Plus Peaks" Model

The core of this method fits in one formula. For each node, we need to find out whether it is cheaper to hold it under a commitment or to take it on demand. A commitment costs a fixed amount for the whole month, while on-demand costs in proportion to hours used. So there is a utilization threshold below which the commitment loses. Call it the break-even utilization:

`u* = commitment_price / on-demand_price`

Both prices are per hour. If a node is needed for more than u* of the month, it is cheaper to take it under a commitment. If less, it is cheaper to take it on demand and pay only for the hours actually used. For p4d.24xlarge we get 13.92 / 21.96 ≈ 63%. The threshold is set by prices, not by the size of our team: the bigger the commitment discount, the lower u*, and the sooner a reservation starts to pay off. With a 37% discount, a node must be loaded almost two thirds of the time to avoid overpaying.

Consider an end-to-end example. Suppose a team needs one p4d.24xlarge node all the time: regular fine-tuning and serving of experimental models take 730 hours a month. A second node is needed about 200 hours a month for large weekly runs. Two more nodes are needed 100 hours a month each for peak experiments before a release. The actual demand is 730 + 200 + 100 + 100 = 1,130 node-hours per month, and peak simultaneous demand is four nodes. Consider four ways to do it.

| Scenario | Composition | Cost per month |
| --- | --- | --- |
| A. "Rent with headroom" | 4 nodes on-demand 24/7: 4 × 16,029 | $64,116 |
| B. "Reserve everything" | 4 nodes on a one-year commitment: 4 × 10,162 | $40,648 |
| C. "On-demand only when needed" | 1,130 h × 21.96 | $24,812 |
| D. "Baseline plus peaks" | 1 node on commitment ($10,162) + 400 hours on-demand (400 × 21.96 = $8,783) | $18,945 |

The hourly rate in the tables is rounded to the cent; the totals are calculated from the exact Infracost rate. Scenario A is the simplest organizationally and the most expensive. Scenario B lowers the price of an hour, but its utilization is 1,130 / 2,920 = 39%, well below the threshold u*: we pay for three nodes that sit idle for most of the month. Scenario C looks attractive, but it demands strict shutdown discipline and carries the risk that no on-demand nodes will be available at the moment of a peak.

Scenario D gives a total cost estimate of $18,945 and is 53% cheaper than B and 70% cheaper than A. But the most interesting comparison is with C: the mixed scheme is cheaper even than "ideal" on-demand, by about $5,900 a month. The reason is that the baseline node is 100% loaded and clears the threshold u* ≈ 63% with room to spare, so for it a commitment beats hourly payment. The second node is loaded 200 / 730 ≈ 27%, which is below the threshold, so reserving it does not pay. The right decision here is therefore one node in the baseline and the rest on demand. This is not a general rule of "reserve a third". It is the result of a calculation on a specific load profile and specific prices. If prices change or the profile shifts, the answer may change, which is why the calculation should be repeated regularly.

## Monthly Planning

The model only works when its inputs are kept current. So I recommend using a monthly cycle with three roles: the architect, who owns the structure of the solution and the calculation; FinOps or the finance director, who owns the budget and commitment terms; and the ML team leads, who know their planned hardware load. The principles of FinOps, that is, shared responsibility of engineering, finance and the business for cloud spend, are well described in the FinOps Framework [9], and this approach is essentially its application to GPU workloads.

The cycle consists of five steps.

**First**: each team sends its training calendar for the month: which runs are planned, how many nodes are needed and for how many hours. Precision to the hour is not needed; an honest estimate is enough, along with the acknowledgment that some runs may not happen.

**Second**: the architect prepares two or three Terraform variants, for example "baseline only" and "baseline plus peaks", and runs them through Infracost to estimate cost.

**Third**: based on the results, a decision is made on commitments: how many nodes to move into the baseline and whether to change existing commitments.

**Fourth**: peaks are booked in advance. If large runs are known weeks ahead, it makes sense to order Capacity Blocks or request quotas, rather than waiting until the nodes are needed urgently.

**Fifth**: at the end of the month the plan is compared with actuals by tags, and the discrepancies become input for the next cycle.

So that "peaks on demand" does not turn into manual work, you need a mechanism that makes this mode automatic. In practice this is a job queue with autoscaling: Kubernetes with Karpenter, which brings up nodes for queued pods and shuts them down when the work is done, or Slurm with an elastic pool. Then engineers do not need to remember to shut things down, and the baseline configuration is simply always used first, and only the overflow goes to the on-demand configuration. Without such automation, scenario C from the example above stays theoretical: money is saved only if the extra nodes really do shut down.

Roles matter more here than tools. If the architect calculates but has no authority over purchasing, the calculation remains a reference note. If FinOps buys commitments without a load profile, the result is scenario B. If team leads do not send their plans, the baseline is set from last month, and we are always one step behind. A regular cycle is needed precisely so that these three parties regularly synchronize their plans and decisions.

## Limitations and Risks

The approach has limits, and they are worth keeping in mind.

**First**: on-demand GPU availability is not guaranteed, especially for top-tier accelerators. A scenario in which peaks depend entirely on whether free nodes can be found at the moment of the request carries a real risk of missed deadlines. So important peaks are better covered by short-term reservations, leaving on-demand for the less critical part of the load.

**Second**: prices change. In 2026 AWS raised Capacity Blocks prices: by about 15% in January, and then several more times during the year [5]. So the threshold u* is not a constant, and a calculation made six months ago may give the wrong answer. The figures in this article should also be seen as an illustration for a specific moment in time and must be re-checked before a project starts.

**Third**: GPUs are not the only cost item. Storage for datasets and checkpoints, network traffic, and egress when moving data between regions or outside also cost money, and in some projects they are comparable to compute. Optimizing one item while ignoring the others gives deceptive savings, so the calculation should include all the infrastructure, not just the GPUs.

**Fourth**: long continuous pretraining runs remain a case where a one-year commitment really does win. If a cluster is occupied by one large job for weeks without pauses, its utilization is close to 100%, clearly above u*, and the commitment discount works in full. The mixed model does not reject long-term commitments. It only requires that they be applied to the part of the load that deserves them.

**Fifth**, and perhaps the most important: organizational risk. If nobody owns this decision, meaning there is no person who recalculates the baseline once a month, things fairly quickly return to the familiar "rent with headroom". It is the safest choice for each individual participant and the most expensive for the company as a whole. To prevent this, the decision needs an owner with clear responsibility and a regular update cycle.

## Conclusion

We started with a cluster rented for a year and loaded to a third. Its problem is not the hardware or the provider but that load and cost were not designed. GPU load in model development is bimodal, small jobs use accelerators poorly, and paying the "continuous operation" rate for such a load usually means overpaying.

The alternative is a mixed model. The minimum baseline, loaded above the threshold u*, is covered by a commitment, and everything else is taken on demand or through short-term reservations booked in advance. In the four-node example, this scheme cost $18,945 a month against $64,116 for renting with headroom. This result is not universal, but the method presented lets you compare a node's utilization with the threshold and decide by calculation, not by feel.

Infracost makes this approach practical because it brings cost into the infrastructure development process. When the estimate is visible in the pull request, and budgets and policies guard against accidental decisions, cost becomes as verifiable a property of the system as reliability or security. It is visible before it turns into a bill, and that is the main idea of cost-driven infrastructure.

## References

1. Hu, Q. et al. *Characterization of Large Language Model Development in the Datacenter*. NSDI 2024. [https://arxiv.org/abs/2403.07648](https://arxiv.org/abs/2403.07648)
2. *SAKURAONE: An Open Ethernet-Based AI HPC System and Its Observed Workload Dynamics in a Single-Tenant LLM Development Environment*. arXiv:2604.13600, 2026. [https://arxiv.org/abs/2604.13600](https://arxiv.org/abs/2604.13600)
3. Gigazine. *Why are GPUs unnecessarily idle during AI training, and what measures has Adobe taken to address this?* (based on a Backblaze publication), 2026. [https://gigazine.net/gsc_news/en/20260826-gpu-idle/](https://gigazine.net/gsc_news/en/20260826-gpu-idle/)
4. AWS. *Amazon EC2 Capacity Blocks for ML — User Guide*. [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-blocks.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-blocks.html)
5. AWS. *Capacity Blocks pricing*. [https://aws.amazon.com/ec2/capacityblocks/pricing/](https://aws.amazon.com/ec2/capacityblocks/pricing/)
6. Vantage. *EC2 instance pricing comparison (p4d.24xlarge)*. [https://instances.vantage.sh/aws/ec2/p4d.24xlarge](https://instances.vantage.sh/aws/ec2/p4d.24xlarge)
7. Infracost. *Documentation*. [https://www.infracost.io/docs/](https://www.infracost.io/docs/)
8. Infracost. *Usage-based resources (EC2 reserved instance usage parameters)*. [https://www.infracost.io/docs/features/usage_based_resources/](https://www.infracost.io/docs/features/usage_based_resources/)
9. FinOps Foundation. *FinOps Framework*. [https://www.finops.org/framework/](https://www.finops.org/framework/)
