# From Jupyter Notebook to a Production ML System

*Why 90% of models never become production solutions*

## Introduction

The story of most ML projects is remarkably repetitive. A Jupyter notebook appears in the data science department. The model shows 98% accuracy on a test set, management is impressed, and the project is declared a success. A few months pass, and it turns out the model never left the notebook: the business isn't using it, it isn't generating revenue, and it is gradually forgotten along with the virtual environment it was trained in — even though its creation consumed months of engineering time. What remains, in the end, is sunk cost. The problem isn't that the model was bad — its metrics proved it worked. The problem is that the cost of moving it into a production loop turned out to be far too high.

Industry experience, accumulated over many years, shows that only a small fraction of models developed in research mode ever reach production — a figure repeated so often in the professional community that it has become something of an aphorism. The reason isn't the quality of the models or the competence of the data scientists; the reason is that a model is only one small part of a much larger system. Google's now-classic paper "Hidden Technical Debt in Machine Learning Systems" demonstrated this most vividly: the model code itself occupies only a small fraction of the overall architecture, surrounded by a mass of infrastructure components — data verification, feature extraction, serving infrastructure, monitoring, and dozens of others, each accumulating its own technical debt [6].

A real ML system isn't a model — it's dozens of services, architectural decisions, and an enormous amount of engineering work. The path from the first experiment to a production AI platform is rarely a straight line, but it almost always passes through the same stages of maturation. In this article, we'll walk that path through the eyes of an architect — from a Jupyter notebook to a platform that unifies classical machine learning, LLMs, and AI agents.

The nine stages described below aren't a rigid methodology or a standard — they're a generalization of a recurring pattern seen in companies of every size, from a startup with a single model to a corporation running dozens of teams. Some organizations move through these stages in a few months; others stretch the transition over years; still others compress several adjacent stages into a single sprint. But the order in which the architectural questions arise is surprisingly stable: first you have to prove the model works at all, and only then that it can work reliably, reproducibly, and at scale. Trying to reverse that order — designing an enterprise platform before a single working model exists — is a classic case of over-engineering: it burns time building architecture for requirements that aren't yet known.

## Stage 1. The Jupyter Notebook

At the start there isn't much: a notebook, a dataset, and a trained model. Formally, everything works — you can run the cells top to bottom and get a prediction.

```text
CSV -> Pandas -> Notebook -> Model
```

This is the stage of pure research, and its value shouldn't be underestimated: it's where the basic hypothesis gets tested — can the model solve the problem at all?

At this stage, there is no architect as such. There's a researcher, whose main job is to prove the idea works and holds up on real data. Any attempt to design an architecture before a working model exists is effort spent on a problem that doesn't yet exist. The path from Python code in a notebook to a production-grade ML system begins precisely by recognizing that this stage calls not for architecture, but for experimental discipline [2].

The most common mistake here is ignoring the reproducibility of experiments. After a week of active work, it's already impossible to reconstruct which version of the data was used, which hyperparameters produced the best result, or why one model was chosen over another. This is where the first layer of the hidden technical debt that Sculley and co-authors describe gets laid down: the confusion between code, data, and configuration doesn't arise in production — it arises right here, during uncontrolled experimentation [6].

It's worth dispelling a common misconception here: reproducibility at the notebook stage isn't about Docker or pipelines — it's about basic discipline. Fixing a seed, saving the dataset version, and logging the hyperparameters and metrics of every run is enough to avoid having to reconstruct the logic of an experiment from memory a month later. The absence of this discipline isn't critical as long as the model lives in the head of a single researcher, but it becomes costly the moment other people join the process at the next stage.

One common solution at this stage is adopting an ML experiment tracking system — MLflow, Weights & Biases, DVC, or something similar. Doing so is a sign of a researcher's maturity and their intent to keep the experiment at the right standard. These systems let you track hyperparameters, datasets, and model metrics, and to store and retrieve the best model by a chosen metric. Using one can be extremely valuable even for a single engineer, and becomes essential the moment more than one person starts working on the model.

A notebook can train a model, but the business wants predictions delivered automatically, not on request from a data scientist. This is where the first genuinely architectural task appears: how to expose the model's output to the outside world.

## Stage 2. The First Inference Service

The model stops being a research artifact and becomes part of the product — it now has to answer requests in real time.

```text
Notebook -> Saved Model -> REST API -> Application
```

The first real circuit of architectural decisions appears: how to serialize the model, where to store it, how to load it when the service starts, how to update it without downtime, and how to scale it under load. The typical stack at this stage is FastAPI for serving requests, MLflow for tracking and versioning the model, and Docker for packaging the environment. Practical experience building a fully automated CI/CD pipeline on top of MLflow, GitHub Actions, FastAPI, and Docker illustrates well what even a minimal production wrapper around a model consists of [1]. It's also worth noting that changing the model is only one of three axes of change in an ML system, alongside code and data; the CD4ML methodology explicitly states that all three axes need to be versioned and deployed reproducibly at the same time [8].

The most common problem at this stage is a model that gets reloaded on every single request, which kills the service's performance. Close behind are the absence of model version control, the absence of input schema validation, and, as a consequence, an API that breaks after the very first model update because nobody warned consumers that the contract had changed.

The question of the data contract deserves special attention. Real, often messy traffic from external systems now hits the service's input — not a neatly prepared test dataset. Without an explicit input schema, the model will sooner or later receive a request it was never trained for, and will either return a meaningless prediction or crash on an unhandled exception. Even at this stage, it makes sense to build basic input validation as a separate layer ahead of the model call, rather than as part of the model's own code.

This stage typically marks the transition from a model in a Jupyter notebook to code organized as separate classes. The data preprocessor, the training code, the inference code, input/output validation, and error handling all split apart. The code stops being a research notebook where the order in which cells are executed affects the result, and takes on the character of a finished project ready for automation.

Request and response handling is introduced, and the first data contract takes shape.

The model works and answers requests. But the next question arises: how do you retrain it as new data appears?

## Stage 3. Training Automation

Manual training runs give way to a pipeline — a reproducible sequence of steps from raw data to a registered model.

```text
Data -> Validation -> Training -> Evaluation -> Registry
```

At this stage we stop thinking of the model as a single artifact and start designing a process. This is where the role of "architect" first emerges — someone who begins to make sense of what's happening and to plan how to embed the resulting microservice into the existing system, or how to design a new one around it.

Questions start to arise: who runs training and when, how do you roll back to a previous version in case of regression, where are training artifacts stored, and how do you guarantee that training can be reproduced a month or a year from now?

Google Cloud formalizes this transition through its MLOps maturity model: level 0 is a manual, fully human-driven process; level 1 adds continuous model training control through an automated pipeline; and level 2 includes full CI/CD for the ML pipeline itself, including data validation at every step [7]. The CD4ML methodology describes the same requirements slightly differently — as the need to make model training a deterministic, repeatable step in a single delivery pipeline, rather than a one-off action performed by a researcher [8].

If training is still triggered manually, the team inevitably loses visibility: nobody can say for certain which model is currently running in production, why that particular model was chosen, or whether the training process that produced it can even be repeated.

The model registry deserves its own mention here — a component that, at this stage, stops being a convenient file store and becomes the source of truth for which model version is used where and why. It's the registry that ties together the training metadata (which data, which code, which metrics), the model artifact itself, and its promotion status across stages, from staging to production. Without that link, the registry degenerates into a dumping ground of files named things like `model_final_v2_new.pkl`, which undoes all the effort put into automation.

The pipeline trains the model reproducibly. But where does the pipeline get its data from, and can that data's quality be trusted?

## Stage 4. Data Becomes a System

Up to this point, data was just a file. Now it's full-fledged infrastructure with its own components and owners.

```text
Sources -> ETL -> Feature Store -> Training -> Serving
```

Here the architect runs into what is arguably the most underestimated problem in machine learning — not the model, not the volume of data, but the consistency of features between training and inference. You need a single set of features shared by training and serving, no leakage of the target variable, identical data-processing logic in both loops, and continuous quality control over incoming data. It's precisely to solve this problem that the industry developed the Feature Store pattern — a shared feature repository accessible both to the training pipeline and to the inference service.

The emergence of managed platforms such as Google Vertex AI, with a dedicated Feature Store as one of their central components, is direct confirmation that this problem is systemic rather than isolated [5]. Sculley and co-authors separately emphasize that it's data, not code, that is the source of the most invisible and most expensive forms of technical debt in ML systems — through hidden feedback loops and undocumented dependencies between data sources [6].

The classic mistake at this stage is duplicating feature engineering: the same logic, with small variations, gets written twice — once for training, once for production. After a month in operation, these two implementations inevitably diverge, and the model in production starts behaving differently from the model that was validated on historical data.

Data contracts are another element that shifts, at this stage, from optional practice to necessity. The team that produces data (a transactional system, say, or a frontend event system) and the team that consumes it for model training rarely sit within the same circle of responsibility. If the data format or the semantics of a field change without warning — and in practice this happens routinely — the training pipeline either fails outright or, worse, keeps running and trains the model on corrupted data. An explicit contract, captured as a schema and validated automatically at the ETL entry point, turns this hidden dependency into a managed, observable process.

So: the data is organized, the features are consistent. But the model still serves only a limited number of test requests — what happens once real users start relying on it?

## Stage 5. Production Arrives

The model goes live for real users, and with that comes an entirely new class of requirements — not about prediction quality, but about service reliability.

```text
Users -> API Gateway -> Model Service -> Monitoring -> Logs
```

The focus now shifts to SLAs (Service Level Agreements), fault tolerance, horizontal scaling, load balancing, and access security. The inference service stops being a standalone application and becomes part of the company's broader production infrastructure, with its typical control plane, serving plane, and observability plane, deployed on managed compute and orchestrated through Kubernetes with proper user access control [1].

The most painful mistake at this stage is viewing system reliability purely through the lens of model quality. In practice, production systems fail for entirely different reasons: a message queue runs out of space, a cache fills up, an API can't handle a traffic spike, or Kubernetes recreates the pod running the model at the worst possible moment, wiping out a warmed-up cache.

!!! warning "Infrastructure matters more than the model"
    At the production stage, infrastructure decisions — not algorithmic ones — most often determine the quality of the system and whether it survives real load. An architect who keeps optimizing accuracy instead of p99 latency and fault tolerance at this stage is solving the wrong problem.

The service is stable and serving real traffic. But is the model actually delivering the value it was expected to?

## Stage 6. Monitoring

At this stage, it's no longer just servers that come under scrutiny. The central question shifts: not "is the service available," but "is the model still actually useful."

You now have to measure model quality over time, track model drift, data drift, and concept drift, monitor the distribution of input features, watch response latency, and even track the cost of a single inference call. Google Cloud explicitly ties this discipline to the earlier maturity stages: without continuous training, without input data validation, and without model drift monitoring, a process simply cannot be called production-grade [7]. In an ecosystem built around MLflow and containerized serving, observability is typically implemented through Prometheus and Grafana on top of a dedicated monitoring dashboard that collects metrics separately from the service's business logic [1].

The most common — and most expensive — mistake at this stage is watching only CPU and memory. A model can respond in milliseconds and produce zero infrastructure-level errors, while its real-world quality on new data has already dropped by half, and the business won't notice until, at best, the next quarterly report.

It's important to distinguish between three types of drift that need to be tracked simultaneously.

Data drift is a change in the distribution of input features relative to what the model was trained on — for example, the age profile of shoppers changes in an online store.

Concept drift is a trickier problem: the relationship between the features and the target variable itself changes over time, even if the input distribution stays the same.

Model drift is the umbrella term for an observed drop in model quality, regardless of which of the first two causes triggered it. Without distinguishing between these, a team typically reacts to the symptom rather than the cause, and retrains the model when what actually needs to change is the feature set.

Degradation is caught in time. The next question is how to respond to it without manual intervention every single time.

## Stage 7. Continuous Learning

Every model gradually becomes outdated as the world around it changes, so it needs a full lifecycle built around it — one closed into a continuous loop.

```text
Production -> Monitoring -> Retraining -> Validation -> Canary -> Production
```

Here a different order of questions arises: how do you know it's time to retrain, how do you properly compare a new model version against the old one, how do you safely switch users over to it, and how do you roll back quickly if something goes wrong?

To switch safely, the industry uses canary deployment and shadow deployment — gradually rolling the new model out to a portion of traffic, or running it in parallel without affecting users at all, which lets you compare metrics before a full release. This is a direct continuation of the continuous-training principles Google Cloud describes [7], and of CD4ML's ideas about continuous, reproducible, and safely reversible delivery of model changes [8].

The most common mistake is retraining "on a schedule" — once a week or once a month — without any real understanding of whether the new model actually improved on the previous one, or whether the retraining was worth the resources spent on it at all. This approach creates the illusion of automation without answering the one question that matters: did the retraining actually help?

The difference between canary and shadow deployment is often a source of confusion in practice, even though it's fundamental. In a canary deployment, a portion of real traffic is genuinely routed to the new model, and its mistakes are visible to users — the risk is offset by keeping that traffic share small and tightly controlled. In a shadow deployment, the new model receives a copy of the traffic in parallel with the old one, but its responses are never shown to users — only logged for comparison — which lets you evaluate the new version with zero risk, though it tells you nothing about how users would actually react to it. A mature platform typically uses both patterns in sequence: shadow first, to gather statistics, then canary, to roll out gradually into live operation.

The lifecycle of a single model is now in place. But what happens when there isn't just one model anymore, but hundreds — and not one team managing them, but dozens?

## Stage 8. Enterprise ML Platform

Once an organization has hundreds of models and dozens of teams building them, one-off solutions stop scaling. They're replaced by a platform with shared services: identity, projects, feature store, registry, pipelines, serving, monitoring, security, cost control, and governance.

The architect's role now takes on a central function: they design not a model, not even a single service, but the platform itself, its processes, team structure, security standards, and the operating cost of the whole enterprise.

The focus shifts to multi-tenancy, so that different teams can safely share common infrastructure; GitOps as the single way changes get delivered; Zero Trust and RBAC/ABAC access models; reproducibility at the level of the entire platform; and compliance and FinOps.

The emergence of managed platforms at the level of Google Vertex AI — unifying Feature Store, Experiments, Pipelines, and Continuous Monitoring in a single ecosystem — is essentially industrial confirmation that this exact set of services is what an organization operating ML at scale actually needs [5]. Governance stops being a formality at this stage: the experience of government programs such as the Algorithm Charter for Aotearoa New Zealand, where more than twenty government agencies voluntarily committed to transparent, accountable, and fair use of algorithms with the support of a dedicated Data Ethics Advisory Group, shows that governance isn't bureaucratic overhead — it's a necessary element of a mature platform, especially where model decisions directly affect people [3].

The main danger here is trying to solve organizational and process problems purely with technical means — scaling up infrastructure without changing access-management processes, quality standards, or accountability for the decisions the model makes.

The platform can now serve classical ML models at industrial scale. But the industry has already posed the next question to the architect: how do you fold large language models and agents into this same platform?

## Stage 9. AI Platform

The next level of maturity goes beyond classical machine learning. Large language models, RAG, AI agents, the MCP protocol, vector databases, orchestration, state machines, and policy engines all get added to the familiar set of components.

Businesses today increasingly see generative AI not as a threat, but as a source of productivity growth — this shift in business sentiment toward AI is directly noted in the business press covering companies adopting generative models [4]. It's exactly this demand that pushes the architect beyond the traditional ML stack. Now they're designing not a standalone ML system, but an intelligent enterprise platform, where LLMs and AI agents aren't isolated experiments but full-fledged components of the overall architecture, subject to the same principles of versioning, observability, and governance developed at earlier stages.

The mistake at this stage mirrors the mistake of the first one: just as a Jupyter notebook gets pushed into production without any architecture around it, LLMs and agents are often bolted onto the platform without bringing along the discipline of prompt versioning, token cost control, and generation quality monitoring — repeating, in effect, a journey classical ML already took ten years to complete.

Here, the evolutionary cycle of the architecture doesn't end — it begins again, at a new level of complexity, but following the same principles.

## Conclusion

The central idea of this article is simple: each stage follows logically from the one before it — it doesn't appear by accident or by fashion. An architect who skips a stage — say, trying to build an Enterprise Platform without working Monitoring in place — is usually forced to return to the skipped level later, but now under the pressure of a production incident rather than at the calm pace of deliberate design.

| Stage | The architect's central question |
| --- | --- |
| Notebook | Does the idea even work? |
| Inference API | How do you deliver a prediction to the user? |
| Training Pipeline | How do you reproduce training? |
| Data Platform | How do you guarantee data quality? |
| Production | How do you ensure service reliability? |
| Monitoring | How do you know the model has degraded? |
| Continuous Learning | How do you improve the model automatically? |
| Enterprise Platform | How do you scale development across dozens of teams? |
| AI Platform | How do you unify classical ML, LLMs, and AI agents into a single architecture? |

A model that shows impressive accuracy in a notebook is only the beginning of the journey, not the finish line. Between an experiment and a production system stand nine stages of architectural maturation, and most models never reach production precisely because someone tries to skip over one or more of them.

The architect's job isn't to artificially speed up this path, but to walk it deliberately — knowing, at every step, which question needs to be answered next.

!!! note "Bottom line"
    The path from a Jupyter notebook to an AI platform isn't a linear checklist — it's a sequence of architectural decisions, each one becoming the foundation for the next.

## Sources

1. Shwet Prakash. *The Ultimate Guide to Production-Grade MLOps: A Fully Automated CI/CD Pipeline with MLflow, GitHub Actions, AWS EC2 and AWS ECS*. Medium. [https://medium.com/@shwet.prakash97/the-ultimate-guide-to-production-grade-mlops-a-fully-automated-ci-cd-pipeline-with-mlflow-github-54956235bd71](https://medium.com/@shwet.prakash97/the-ultimate-guide-to-production-grade-mlops-a-fully-automated-ci-cd-pipeline-with-mlflow-github-54956235bd71)
2. Shailesh Kumar Khanchandani. *A Practical MLOps Roadmap: From Python Code to Production-Grade ML Systems*. Medium. [https://medium.com/@skk.jodhpur/a-practical-mlops-roadmap-from-python-code-to-production-grade-ml-systems-2ee18181d681](https://medium.com/@skk.jodhpur/a-practical-mlops-roadmap-from-python-code-to-production-grade-ml-systems-2ee18181d681)
3. *Algorithm Charter for Aotearoa New Zealand*. data.govt.nz (NZ Government). [https://data.govt.nz/blog/algorithm-charter-for-aotearoa-new-zealand](https://data.govt.nz/blog/algorithm-charter-for-aotearoa-new-zealand)
4. Hamish Cardwell. *Businesses urged to view artificial intelligence as an opportunity, not a threat*. RNZ. [https://www.rnz.co.nz/news/business/487847/businesses-urged-to-view-artificial-intelligence-as-an-opportunity-not-a-threat](https://www.rnz.co.nz/news/business/487847/businesses-urged-to-view-artificial-intelligence-as-an-opportunity-not-a-threat)
5. Staff Writer. *Google Cloud launches Vertex AI, an all-new ML platform*. ITBrief New Zealand. [https://itbrief.co.nz/story/google-cloud-launches-vertex-ai-an-all-new-ml-platform](https://itbrief.co.nz/story/google-cloud-launches-vertex-ai-an-all-new-ml-platform)
6. D. Sculley et al. *Hidden Technical Debt in Machine Learning Systems*. NeurIPS 2015. [https://proceedings.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf)
7. Google Cloud Architecture Center. *MLOps: Continuous delivery and automation pipelines in machine learning*. Google Cloud. [https://docs.cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning](https://docs.cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
8. Danilo Sato, Arif Wider, Christoph Windheuser. *Continuous Delivery for Machine Learning*. martinfowler.com. [https://martinfowler.com/articles/cd4ml.html](https://martinfowler.com/articles/cd4ml.html)
