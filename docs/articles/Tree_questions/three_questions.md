# The Three Fundamental Questions of Architecture: Why, What, and How

There are three questions that any large IT system's architecture cannot afford to leave unanswered without becoming inefficient and poorly scalable: *Why*, *What*, and *How*. These questions are not arbitrary — a similar structure already appears in the [Zachman Framework](https://medium.com/datacrat/zachman-framework-73cbb960a4eb), the classic enterprise-architecture model in which rows represent perspectives on the system (What, How, Where, Who, When, Why) and columns represent levels of detail, from business context down to finished implementation. The three questions discussed in this article are a simplified but practical projection of that same principle onto an architect's day-to-day work.

It is useful to map these questions onto three levels of architectural work: Enterprise Architect, Solution Architect, and Software Architect. Each answers a different question:

* **Why?** — Why are we building the system in the first place? This is the Enterprise Architect's area of responsibility.
* **What?** — What exactly should the system be? This is where the Solution Architect operates.
* **How?** — How will we implement it? This is the Software Architect's question.

## Introduction

Almost every architecture discussion starts from the wrong end. The reason lies in most architects' backgrounds: they grew up as technical specialists, so the question "How" feels the most familiar and comfortable — there is an immediate urge to work out implementation details, pick a stack, or sketch a class diagram. As a system and an architect's experience grow, the architect's thinking gradually becomes more abstract, and the focus shifts to "What," the question that defines the overall architecture of the solution.

It helps to picture a solution as a building. If pipes are sticking out of the walls with water leaking from them and half the windows are missing, that is poor work by the developers — but if the pipes were routed to the wrong rooms altogether, that is poor work by the Software Architect.

Let's go further: suppose the house has been built, everything inside works, but the building itself looks like it was assembled from mismatched pieces. It looks awkward and unattractive. This is where we see the answer to the question "What" we built. All the requirements were met, yet we did not end up with a coherent solution. The solution cannot function and perform its purpose as a unified system — the Solution Architect has failed at the task.

And the third level is "Why." Returning to our analogy, imagine we have erected a beautiful, functional, well-built structure — but in the wrong place, somewhere nobody needs it. The solution exists, but it does not fulfill the ultimate business function; it does not solve the problem the customer actually asked to have solved.

It is precisely the "Why" question that deserves the most attention from the very start, because it is the one that determines how the future solution will address the customer's real needs — that is, how it implements the business logic and solves the specific problems the project was undertaken to solve in the first place.

In his well-known essay [“Who Needs an Architect?”](https://martinfowler.com/ieeeSoftware/whoNeedsArchitect.pdf), Martin Fowler describes an architect not as someone who single-handedly makes every technical decision, but as someone responsible for ensuring that the important decisions — the ones that are expensive to change later — are made consciously and in the right order. It is precisely this ordering of questions that gets overlooked most often: teams argue about technologies before they have worked out why those technologies are needed at all.

A typical scene looks like this — people arguing about:

* Kubernetes or ECS?
* PostgreSQL or DynamoDB?
* Python or Go?
* Kafka or RabbitMQ?

Yet none of these questions is actually the first one to ask. Real architecture begins much earlier, with three fundamental questions: Why, What, and How. It is the answers to these questions, taken in the right sequence, that determine the quality of the entire architecture.

## Why Architects Keep Arguing With Each Other

A telling scenario plays out on almost every large project. The Enterprise Architect says: "We need digital transformation." The Solution Architect replies: "Let's break the system into microservices." The Software Architect adds: "We should use Event Sourcing."

Formally, all three are right. But each of them is answering the question that belongs to their own level, not the question the others actually raised: the Enterprise Architect is talking about Why, the Solution Architect about What, and the Software Architect about How. This is exactly why such conversations feel like the architects are speaking different languages — they genuinely are discussing different layers of the same problem, simply without saying so explicitly.

## Why — Enterprise Architecture

The central question at this level is: why does the system exist at all? Here the discussion revolves around business, strategy, value, ROI, processes, users, risks, and constraints. An architect working at this level talks about technology very little — their language is much closer to the language of business than to the language of engineering.

The idea of breaking any initiative down into "why — how — what" will be familiar from [Simon Sinek's Golden Circle model](https://simonsinek.com/golden-circle): strong organizations and leaders start precisely with Why, because the reason a product exists motivates and persuades people far more powerfully than a list of features. The same principle applies in architecture: before designing a system, you need to clearly articulate the purpose it is being built for.

The Enterprise Architect answers three key questions: what problem are we solving; why is the company investing money in this at all; and what does success look like. Typical artifacts at this level include the Capability Map, Business Process, Value Stream, Business Goals, and Roadmap.

The Enterprise Architect role is not some abstract "architecture philosopher" position. It is formally defined even in government competency frameworks: for example, [guidance from the New Zealand government](https://www.digital.govt.nz/standards-and-guidance/strategy-and-planning/digital-capability-public-service-workforce/align-your-roles-to-sfia) defines this role through the SFIA framework, and [New Zealand's Government Enterprise Architecture (GEA-NZ)](https://www.digital.govt.nz/standards-and-guidance/technology-and-architecture/government-enterprise-architecture) is built on The Open Group's TOGAF methodology and applied nationwide — demonstrating that the Why question, and the work of answering it, can be formalized just as rigorously as technical architecture.

The main outcome of work at this level is a clear understanding of why the system exists.

## What — Solution Architecture

The next question is: what exactly needs to be built? At this level, services, components, integrations, data flows, APIs, and system boundaries come into the picture. This is where the first architecture diagram typically appears, but the architect is still not discussing code — they are designing the structure of the system, not its internal workings.

[Simon Brown's C4 model](https://c4model.com/) is a convenient tool for this level: its top two layers — Context and Container — describe exactly what major parts the system consists of and how those parts interact with one another, without descending into code-level detail. Typical Solution Architecture artifacts include the Context Diagram, Container Diagram, Component Diagram, Integration Diagram, and Sequence Diagram.

The main outcome of this level is clarity about what parts the system is made of.

## How — Software Architecture

The final question is: how exactly will this be implemented? Here the discussion covers languages, frameworks, databases, patterns, CI/CD, testing, security, and, finally, the code itself. Continuing the C4 model analogy, this corresponds to the Component and Code levels — the detail within individual containers, which only becomes meaningful once the system's boundaries have already been defined at the What level.

Typical decisions at this level include CQRS, Event Sourcing, DDD, Clean Architecture, and Hexagonal Architecture. The main outcome is a clear understanding of exactly how developers will build the system.

## Why You Cannot Skip Levels

A very common mistake is jumping straight into the How level, bypassing Why and What. A team starts by saying, "Let's use Kubernetes," even though nobody has yet answered why this service is needed in the first place. Another example: a team enthusiastically discusses Kafka, but no one has defined what events actually exist, who produces them, and who consumes them.

In both cases, the result is technology without architecture. The industry has a name for this mistake — the ["Shiny Nickel"](https://medium.com/@srinathperera/a-deeper-look-at-software-architecture-anti-patterns-9ace30f59354) anti-pattern: a team picks a trendy, "shiny" technology before it has understood the problem that technology is supposed to solve. A technology is never, by itself, an architectural decision — it only becomes one once the Why and What questions have been answered.

## How the Questions Are Nested Within Each Other

The three questions do not exist independently — they are nested within one another, and each subsequent level is meaningful only because the previous one has already been defined. This can be represented as a sequence of increasingly specific questions:

```
Why  — Enterprise architect : Why are we building the system at all?
  ↓
What — Solution architect   : What system needs to be built to achieve the goal?
  ↓
How  — Software architect   : How do we implement this system in practice?
```

Or as a set of nested dolls, where each inner layer can only be opened once the one before it has been removed:

* **Why**
    * **What**
        * **How**

!!! tip "Practical implication"
    If a meeting is discussing a technology (How) and the team cannot state, in a single sentence, which business goal (Why) it serves, that is a signal to step back up a level rather than argue about implementation details.

## A Real-World Example: an AI Platform

Let's walk through a real case: building an AI platform for a company.

**Why.** The company wants to cut ML model development time in half. This is a concrete, measurable business goal, and it is the one the Enterprise Architect brings to the other levels.

**What.** To achieve this goal, the platform needs the following components:

* ML pipeline orchestration;
* experiment tracking and model versioning;
* a single source of features for training and inference — a Feature Store;
* a catalog and lifecycle management for models — a Model Registry;
* delivery of models into production — Serving;
* observability into model quality in production — Monitoring.

**How.** Only now, once the goal and the composition of the solution have been defined, do concrete implementation technologies naturally emerge:

* Kubernetes — the container orchestration platform;
* Kubeflow — ML pipeline orchestration;
* MLflow — experiment tracking and model versioning;
* Feast — the single source of features for training and inference;
* Harbor — the catalog and lifecycle management for models;
* ArgoCD — GitOps-based delivery;
* KServe — the model serving layer;
* Kafka — streaming of data and events;
* Prometheus — observability into model quality in production;
* Ceph — object storage;
* Terraform — infrastructure as code;
* AWS — the cloud platform.

The order here matters. Technologies at the How level emerge as a consequence of decisions made at the What level, which in turn follow from the goal set at the Why level. Had the team started by choosing Kubernetes and Kafka without first articulating why the platform was needed and what components it should consist of, the result would have been an expensive pile of infrastructure with no clear connection to the business goal.

## Where Each Architect's Responsibility Ends

| Role | Key Question | Primary Outcome |
|---|---|---|
| Enterprise Architect | Why | Business value and goals |
| Solution Architect | What | Solution architecture |
| Software Architect | How | Technical implementation |

It is worth emphasizing that these are not three independent roles but three levels of decision-making. In small companies, one person may cover all three levels at once, whereas in large organizations they are typically distributed across different specialists and teams.

## Conclusion

Architecture is not a matter of choosing technologies. Technologies only emerge after three fundamental questions have been answered:

1. **Why** — why is the system being created, and what business value should it deliver?
2. **What** — what should the solution architecture be in order to achieve that goal?
3. **How** — how should this architecture be implemented, using which specific technologies and engineering practices?

The more complex the system, the more expensive it becomes to start with the How question. Mature architecture teams always work top-down: from business goals, to solution architecture, and only then to implementation. It is this sequence — not the choice of a particular stack — that distinguishes an architect from an engineer who picks technologies based on familiarity, and it is precisely this discipline that makes it possible to build systems that not only work today but also remain resilient to future changes in both business and technology.

## Sources

1. Align your roles to SFIA — New Zealand Digital government — [https://www.digital.govt.nz/standards-and-guidance/strategy-and-planning/digital-capability-public-service-workforce/align-your-roles-to-sfia](https://www.digital.govt.nz/standards-and-guidance/strategy-and-planning/digital-capability-public-service-workforce/align-your-roles-to-sfia) — the official definition of the Enterprise Architect role through the SFIA framework.
2. Zachman Framework — Janhavie, Datacrat / Medium — [https://medium.com/datacrat/zachman-framework-73cbb960a4eb](https://medium.com/datacrat/zachman-framework-73cbb960a4eb) — a breakdown of the 6×6 Zachman matrix and its "primitive questions" (What/How/Where/Who/When/Why).
3. Enterprise architecture (GEA-NZ) — New Zealand Digital government — [https://www.digital.govt.nz/standards-and-guidance/technology-and-architecture/government-enterprise-architecture](https://www.digital.govt.nz/standards-and-guidance/technology-and-architecture/government-enterprise-architecture) — New Zealand's government enterprise architecture, built on TOGAF/The Open Group.
4. The C4 model for visualising software architecture — Simon Brown — [https://c4model.com/](https://c4model.com/) — the official description of the C4 model (Context, Container, Component, Code).
5. The Golden Circle — Simon Sinek — [https://simonsinek.com/golden-circle](https://simonsinek.com/golden-circle) — the Why–How–What Golden Circle model.
6. Who Needs an Architect? — Martin Fowler, IEEE Software, 2003 — [https://martinfowler.com/ieeeSoftware/whoNeedsArchitect.pdf](https://martinfowler.com/ieeeSoftware/whoNeedsArchitect.pdf) — the classic essay on the architect's role.
7. A Deeper Look at Software Architecture Anti-Patterns — Srinath Perera, Medium — [https://medium.com/@srinathperera/a-deeper-look-at-software-architecture-anti-patterns-9ace30f59354](https://medium.com/@srinathperera/a-deeper-look-at-software-architecture-anti-patterns-9ace30f59354) — anti-patterns, including "Shiny Nickel" (choosing a technology before understanding the problem).
