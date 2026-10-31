# Team Agent Context Platform

*как построить общую память для команды людей и AI-агентов*

## 1. Введение: AI-агент уже умеет писать код. Следующая проблема — командная работа

### 1.1. От Copilot к участнику команды

За последние несколько лет роль AI в разработке программного обеспечения заметно изменилась. 
Если первые инструменты на базе больших языковых моделей воспринимались прежде всего как интеллектуальное автодополнение к IDE, 
то современные системы постепенно превращаются в самостоятельных участников процесса разработки.

Эту эволюцию можно условно представить следующим образом:

Autocomplete -> Chat Assistant -> Coding Agent -> Autonomous Agent -> Multi-Agent Development

На первом этапе AI фактически выполнял роль продвинутого autocomplete. 
Разработчик оставался непосредственным автором кода, а модель предлагала продолжение текущей строки, функции или небольшого фрагмента программы. 
Контекст взаимодействия при этом был локальным: открытый файл, несколько соседних функций и текущая позиция курсора.

Следующим шагом стали Chat Assistants. 
Разработчик получил возможность обсуждать с моделью код, задавать вопросы об ошибках, просить объяснить незнакомую библиотеку, написать тест или предложить вариант реализации. 
Контекст стал существенно шире, но управление процессом по-прежнему полностью оставалось у человека: человек формулировал каждый следующий запрос, переносил результат в проект и решал, что делать дальше.

Появление Coding Agents изменило саму модель взаимодействия. Теперь разработчик может поставить задачу более высокого уровня:
"Добавь авторизацию для этого API, обнови тесты и проверь, что существующие сценарии продолжают работать".

Агент самостоятельно исследует структуру repository, находит связанные компоненты, читает существующую реализацию, определяет необходимые изменения, модифицирует несколько файлов и запускает проверки.

Вместо последовательности команд человека

- найди файл
- посмотри функцию
- измени код
- добавь тест
- запусти pytest
- исправь ошибку

возникает более высокоуровневое взаимодействие:

Developer (Goal) -> Coding Agent (Explore -> Plan -> Implement -> Test -> Fix -> Report)

Следующий этап - Autonomous Agents. 
Такой агент получает не отдельную инструкцию по изменению кода, а задачу и критерии результата. 
В рамках предоставленных ему полномочий он способен самостоятельно выполнить достаточно длинную последовательность действий: 
исследовать проблему, сформировать план, изменить код, запустить тесты и линтеры, проанализировать ошибки, повторить цикл исправления и подготовить Pull Request.

Современный coding agent уже способен:

* исследовать repository и находить связанные компоненты;
* анализировать существующую архитектуру;
* планировать изменение до начала реализации;
* изменять несколько взаимосвязанных частей системы;
* писать и обновлять тесты;
* запускать build, tests, linters и другие проверки;
* анализировать результаты CI;
* создавать Pull Request;
* проводить первичный code review;
* работать с issue tracker и внешними инструментами;
* выполнять многошаговые задачи, значительно превышающие по длительности обычную chat-сессию.

Это постепенно меняет роль разработчика.

Вместо непосредственного управления каждой операцией появляется модель:

Human (intent - constraints - acceptance criteria) -> Agent (planning - implementation - verification) -> Engineering Artifact -> Human Review

Однако наиболее интересное изменение начинается ещё на следующем уровне — Multi-Agent Development.

Если один агент способен самостоятельно выполнить инженерную задачу, естественным следующим шагом становится специализация:

                 Task
                  │
                  ▼
              Planner Agent
                  │
                  ▼
            Implementation Plan
                  │
          ┌───────┴────────┐
          ▼                ▼
   Developer Agent    Security Agent
          │                │
          └───────┬────────┘
                  ▼
             Reviewer Agent
                  │
                  ▼
                 PR

Один агент исследует задачу и формирует план, другой реализует изменение, третий анализирует безопасность, четвёртый проверяет результат. 
В более сложном случае несколько implementation agents могут параллельно работать над разными компонентами одной системы.

В результате AI постепенно перестаёт быть просто инструментом отдельного разработчика.

Возникает новая организационная модель:

Developer A ── Agent A
Developer B ── Agent B
Developer C ── Agent C
Architect   ── Agent D
                 │
                 ▼
             Project

И именно здесь возникает принципиально новый класс проблем.

Пока AI является autocomplete или персональным chat assistant, достаточно контекста конкретного разработчика. 
Но как только несколько людей и несколько агентов начинают параллельно изменять одну систему, им требуется общий контекст проекта.

Agent A должен знать, почему месяц назад было принято определённое архитектурное решение. 
Agent B должен понимать, что компонент, который он собирается изменить, уже перерабатывается другим разработчиком. 
Agent C не должен предлагать технологию, которую команда ранее исследовала и сознательно отвергла. 
А новый агент должен иметь возможность подключиться к проекту, не получая для этого историю нескольких сотен предыдущих диалогов.

Таким образом, следующий этап развития AI-assisted development связан уже не только с увеличением возможностей самих моделей.

Появляется другая задача:
"Как организовать совместную работу людей и AI-агентов так, чтобы они разделяли единое понимание состояния проекта, принятых решений, текущих задач и накопленного инженерного опыта?"

Именно с этой точки начинается проблема коллективного контекста проекта, которую мы будем рассматривать далее.

### 1.2. Главная проблема — потеря контекста

Когда AI-агент используется одним разработчиком для решения локальной задачи, проблема контекста относительно проста. 
Разработчик знает историю проекта, понимает принятые ранее решения и способен дополнить запрос необходимой информацией. 
Даже если агент чего-то не знает, человек может скорректировать его действия.

Ситуация существенно меняется, когда агенты становятся повседневным инструментом всей команды:

Developer A  ↔  Agent A
Developer B  ↔  Agent B
Developer C  ↔  Agent C

Каждая такая пара постепенно формирует собственный локальный контекст проекта.

Developer A объясняет своему агенту, почему определённый сервис нельзя напрямую подключать к базе данных. 
Developer B вместе со своим агентом исследует варианты организации очередей сообщений. 
Developer C обсуждает с третьим агентом ограничения API Gateway.

Через некоторое время фактическое знание о проекте начинает выглядеть примерно так:

                         Project
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
        Developer A   Developer B   Developer C
              │            │            │
              ▼            ▼            ▼
           Agent A      Agent B      Agent C
              │            │            │
              ▼            ▼            ▼
          Context A     Context B     Context C

Проблема состоит в том, что:

Context A ≠ Context B ≠ Context C

и ни один из них не является полным контекстом проекта.

Знание остаётся внутри локальной сессии

Предположим, Developer A и Agent A при реализации задачи обнаружили важное архитектурное ограничение:

Все обращения к Policy Engine должны проходить через Context Gateway. Прямой вызов OPA из прикладных сервисов запрещён.

Agent A учитывает это ограничение и корректно реализует задачу.

Но если найденное правило осталось только внутри истории разговора:

Developer A
     │
     ▼
Agent A conversation
     │
     ▼
architecture constraint

то с точки зрения остальных участников команды этого знания фактически не существует.

Через неделю Developer B ставит своему агенту похожую задачу.

Agent B видит код и формулировку задачи, но не видит предыдущую сессию Agent A. Поэтому он вполне рационально предлагает:

Application Service
        │
        ▼
       OPA

хотя команда уже определила другую архитектуру:

Application Service
        │
        ▼
 Context Gateway
        │
        ▼
       OPA

Оба агента могут при этом действовать совершенно корректно относительно предоставленного им контекста.
Ошибка возникает не на уровне модели.
Ошибка возникает на уровне организации знания.

Команда начинает повторно оплачивать одни и те же исследования.

Рассмотрим другой пример.
Developer A вместе с агентом исследовал выбор между Kafka и NATS JetStream:
Kafka vs NATS JetStream

Были изучены:

* эксплуатационная сложность;
* latency;
* delivery semantics;
* request/reply;
* масштабирование;
* требования текущей системы.

После исследования команда выбрала NATS JetStream.
Но само исследование и причины выбора остались в conversation history.

Через несколько месяцев Agent B получает задачу, связанную с messaging layer, и обнаруживает Kafka как очевидный кандидат.
Начинается новое исследование:

Agent A:
Kafka vs NATS - research - NATS (несколько месяцев)

Agent B:
Kafka vs NATS - research - ?

Команда второй раз расходует:

* время разработчика;
* время агента;
* LLM tokens;
* вычислительные ресурсы;
* время code review.

Причём второй агент может прийти к другому выводу просто потому, что получил немного другой набор исходных данных.

Проблема заключается уже не только в потере информации, но и в потере причин принятия решений.

Код не содержит всей истории решения.
Можно возразить, что агент способен исследовать repository и восстановить архитектуру по существующему коду.
До определённой степени это действительно так.

Если система использует NATS, агент увидит:
import nats

Но из кода невозможно надёжно определить:
Почему NATS?

Возможно:

* Kafka исследовали и отвергли.
* Kafka планируют внедрить позже.
* NATS используется временно.
* Kafka запрещена инфраструктурной командой.
* NATS выбран из-за request/reply.
* NATS используется только для конкретного bounded context.

Текущее состояние кода показывает прежде всего что реализовано, но значительно хуже отвечает на вопрос почему система стала именно такой.
Именно эта часть знания особенно важна для AI-агента, который должен принимать следующие инженерные решения.

Контекст распределён не только между агентами.

Ситуация ещё сложнее, потому что значительная часть проектного знания вообще находится вне AI-сессий:

                         Project Context
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
         Git                Jira              Email
          │                  │                  │
          ▼                  ▼                  ▼
     Source Code           Tasks            Decisions
          │
          ├──────────────┬───────────────┐
          ▼              ▼               ▼
       Meetings      Telegram/Slack    AI Chats

* Архитектурное решение могло появиться на встрече.
* Уточнение — в Telegram.
* Задача — в Jira.
* Реализация — в Git.
* Причина изменения — в Pull Request.
* А дополнительное исследование — в conversation разработчика с AI-агентом.

Каждый источник содержит корректный фрагмент информации, но единой модели проекта при этом нет.
Появляется временная фрагментация.

Есть ещё одна проблема — время.
Контекст проекта постоянно изменяется.

Например:

* January ADR-12: Use REST between services
* March ADR-27: Introduce asynchronous communication
* June ADR-41: Use NATS JetStream for event transport

Semantic search вполне может найти все три документа.
Но агенту недостаточно знать, что они существуют.

Он должен понимать:

* ADR-12 -> ADR-27 -> ADR-41
* Current state: ADR-41

Иначе большая память способна парадоксальным образом ухудшить результат: чем больше исторической информации доступно агенту, тем больше среди неё устаревших и противоречащих друг другу утверждений.
В итоге можно сформулировать более общую проблему.
Назовём её Context Fragmentation Problem — фрагментацией проектного контекста между людьми, агентами, информационными системами и временем.

             Project Knowledge
     ┌──────────────┼──────────────┐
     │              │              │
     ▼              ▼              ▼
   People          Tools          Agents
     │              │              │
 ┌───┼───┐      ┌───┼────┐     ┌───┼───┐
 ▼   ▼   ▼      ▼   ▼    ▼     ▼   ▼   ▼
 A   B   C     Git Jira Chat   A1  A2  A3
                  +
                 Time
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
       Past     Current    Planned

В результате команда сталкивается сразу с несколькими эффектами:

* разные агенты имеют разные представления об одной системе;
* ранее выполненные исследования повторяются;
* отвергнутые решения возникают снова;
* архитектурные ограничения теряются;
* новые разработчики и агенты долго восстанавливают контекст;
* исторические решения смешиваются с актуальными;
* знания, полученные одним агентом, не становятся автоматически знаниями остальных;
* важная информация остаётся внутри встреч, переписки и локальных AI-сессий.

Особенно опасно, что эта проблема масштабируется вместе с количеством агентов.

Для одного разработчика:

1 Human + 1 Agent

потерю контекста ещё можно компенсировать человеческой памятью.

Но для команды:
10 Developers + 10 Personal Agents + Planner Agents + Review Agents + Security Agents
ручная синхронизация становится всё менее реалистичной.

Мы получаем парадокс: чем больше работы способны выполнить AI-агенты, тем более серьёзной становится проблема координации их контекста.

Поэтому масштабирование agent-assisted development требует решения уже не только задачи:
Как сделать отдельного агента умнее?

Возникает более важный системный вопрос:
Как сделать так, чтобы знания, полученные одним участником проекта — человеком или агентом, — становились доступной, актуальной и проверенной частью общего контекста для всех остальных?

Именно для решения этой проблемы нам потребуется перейти от памяти отдельных агентов к управляемой памяти самого проекта.

## 2. Где сегодня находится память проекта

Если попросить команду показать исходный код проекта, ответ обычно очевиден — Git repository.
Если спросить, где находятся задачи, скорее всего, это будет Jira или другой issue tracker.
С документацией ситуация также выглядит относительно понятной: Confluence, Wiki или каталог `docs` внутри repository.

Но значительно сложнее ответить на другой вопрос:

> **Где находится память проекта?**

То есть где можно получить не отдельный документ или список задач, а целостный ответ на вопросы:

* в каком состоянии проект находится сейчас;
* какие решения уже приняты;
* почему они были приняты;
* какие решения ещё обсуждаются;
* над чем сейчас работает команда;
* кто за что отвечает;
* какие задачи заблокированы и почему;
* какие компоненты изменяются;
* какие риски обнаружены;
* какие исследования уже проводились;
* какие варианты были отвергнуты;
* что должно произойти дальше.

В большинстве реальных проектов такой системы не существует.

Память проекта распределена между множеством источников:

```text
                         PROJECT MEMORY

                               │
        ┌──────────────┬───────┼────────┬──────────────┐
        │              │       │        │              │
        ▼              ▼       ▼        ▼              ▼
       Git            Jira    Email   Meetings     Confluence
        │              │       │        │              │
        ▼              ▼       ▼        ▼              ▼
      Code           Tasks   Threads  Decisions    Documents
        │
        │
        ├──────────────┬───────────────┬──────────────┐
        ▼              ▼               ▼              ▼
   PR discussions   Telegram       Personal       AI
                    Slack/Teams     Notes      Conversations
```

Каждая из этих систем содержит важную часть истории проекта. Но ни одна из них не содержит проект целиком.

### Git: что было реализовано

Git является наиболее надёжным источником информации о фактическом состоянии программной системы.

В нём находятся:

* source code;
* configuration;
* Infrastructure as Code;
* tests;
* commit history;
* branches;
* tags;
* иногда архитектурная документация и ADR.

Git хорошо отвечает на вопрос:

> Что сейчас реализовано?

Но значительно хуже отвечает на вопрос:

> Почему это было реализовано именно так?

Предположим, агент обнаруживает:

```text
services/
    gateway/
    policy/
    mcp-router/
```

и видит, что `gateway` обращается к `policy`, а остальные сервисы не имеют прямого доступа к нему.

Из кода можно восстановить существующую архитектуру:

Service - Gateway - Policy Engine

Но невозможно надёжно определить, является ли это:

* сознательным security constraint;
* временной реализацией;
* исторически сложившейся структурой;
* результатом ограничений старой версии системы;
* архитектурным правилом, которое должно применяться и к новым сервисам.

Git хранит **результат инженерного решения**, но далеко не всегда сохраняет сам процесс принятия решения.

---

### Jira: что команда собирается делать

Issue tracker содержит другую часть памяти:

Epic - Story- Task - Subtask

Здесь находятся:

* backlog;
* sprint;
* assignments;
* priorities;
* statuses;
* acceptance criteria;
* deadlines;
* иногда dependencies.

Jira хорошо отвечает на вопрос:

> Какие задачи существуют?

Но значительно хуже:

> Почему появилась эта задача?

Например:
TASK-417 - Implement ACL for MCP Router

может быть результатом архитектурной встречи, security review или production incident.

Причина могла обсуждаться в другом месте:

Meeting - Security discussion - Decision - Jira TASK-417

В Jira остался только последний элемент этой цепочки.

### Email: договорённости и контекст между системами

Email часто содержит информацию, которой нет больше нигде:

* согласования;
* ограничения;
* требования внешних команд;
* ответы заказчиков;
* технические уточнения;
* сроки;
* обязательства;
* решения.

Например:

From: Security Team

Direct access to the Policy Engine
from application services will not
be approved.

Это фактически архитектурное ограничение.
Но если оно осталось только внутри email thread, то новый разработчик или AI-агент практически не имеет возможности узнать о нём автоматически.

### Telegram, Slack и Teams: оперативная память команды

Значительная часть реальной инженерной работы происходит в корпоративных чатах.

Типичное обсуждение:

Developer A:
Почему здесь используется Redis?

Developer B:
Мы сначала пробовали PostgreSQL,
но latency была слишком большой.

Architect:
Да. Redis оставляем.
Для persistence используется PostgreSQL.

За несколько сообщений возникло важное архитектурное знание:

* Redis - operational state
* PostgreSQL - persistent state

и одновременно зафиксирована причина этого решения.

Но через несколько месяцев найти это обсуждение будет сложно даже человеку, который в нём участвовал.
Для нового AI-агента этого знания практически не существует.

Чаты являются хорошей **оперативной памятью команды**, но плохой долговременной памятью проекта.

### Meetings: место, где принимается большая часть решений

Особенно много контекста теряется на встречах.

Именно там часто происходит переход:

Problem - Discussion - Alternatives - Decision - Assignment

Например:

Architect:
Kafka здесь будет избыточна.

Developer:
Тогда используем NATS?

Architect:
Да. Для MVP берём NATS JetStream.

Tech Lead:
Хорошо. Сергей обновит ADR,
Анна после этого изменит Router.

За несколько минут появились:

**Decision**:
Use NATS JetStream for MVP.

**Task**:
Update ADR.
Owner: Sergey.

**Dependency**:
Router change
depends on ADR update.

**Assignment**:
Router change
Owner: Anna.

Если после встречи был создан только Jira ticket:

**Jira**:
Update Router

большая часть контекста уже потеряна.

### Confluence и Architecture Documents: формализованная память

Документация должна была бы решить эту проблему.
И частично она действительно её решает.

Architecture documents, ADR, RFC и design documents способны сохранить:

* архитектурный контекст;
* alternatives;
* decisions;
* trade-offs;
* constraints.

Это один из наиболее ценных источников долговременной памяти проекта.

Но появляется другая проблема — **актуальность**.

Architecture Document

created: 2025-10

Current System

2026-08

Документ может выглядеть официальным, хорошо структурированным и при этом описывать систему девятимесячной давности.
Поэтому наличие документа ещё не означает, что содержащаяся в нём информация является текущим состоянием проекта.

### Pull Request discussions: память инженерных компромиссов

PR/MR discussions часто содержат особенно ценную информацию.

Например reviewer пишет:

Don't call Repository directly
from the HTTP handler.

Use Service layer because transactions
are managed there.

Автор исправляет код, PR закрывается.

В repository остаётся правильная структура:

HTTP Handler - Service - Repository

Но причина изменения остаётся в комментариях к старому PR.

Следующий разработчик или агент может снова попытаться сделать:

HTTP Handler - Repository

и команда повторит тот же цикл review.

Таким образом, code review создаёт большое количество полезного инженерного знания, которое редко становится частью долговременной памяти проекта.

### Personal Notes: невидимая память

Практически в каждой команде существует ещё один слой:

* notes.md
* Notion
* Obsidian
* Apple Notes
* personal TODO
* local markdown
* paper notebook

Иногда именно там находится лучшее описание определённой части системы.
Но для проекта это знание фактически недоступно.

Возникает важное различие:

Team member knows $\neq$ Team knows

### AI conversations: новый и быстро растущий слой памяти

С распространением coding agents появляется ещё один источник.

Разработчик может провести с агентом довольно глубокое исследование:

Developer - Agent - repository analysis - alternative architectures - experiments - decision - implementation

В рамках одной сессии агент может изучить десятки файлов, проверить несколько гипотез и обнаружить неочевидные ограничения системы.
Это потенциально очень ценное инженерное знание.

Но после завершения сессии большая его часть остаётся в conversation history:
Agent Session #18427

Другой разработчик и другой агент начинают следующую задачу практически с нуля.
По мере распространения AI-разработки эта проблема будет только увеличиваться: значительная часть инженерного анализа начинает происходить не между людьми, а внутри индивидуальных взаимодействий `Developer ↔ Agent`.


### Отдельные системы знают факты, но не знают проект

В результате мы получаем распределённую картину:

| Источник                 | Что он хорошо знает                     |
| ------------------------ | --------------------------------------- |
| Git                      | Что реализовано                         |
| Jira                     | Что запланировано и выполняется         |
| Email                    | Формальные и внешние договорённости     |
| Telegram / Slack / Teams | Текущие обсуждения                      |
| Meetings                 | Решения, аргументы и договорённости     |
| Confluence               | Формализованную документацию            |
| PR discussions           | Причины конкретных инженерных изменений |
| Personal Notes           | Локальный опыт человека                 |
| AI Conversations         | Исследования и опыт конкретного агента  |

Каждая система отвечает на свой вопрос.

Но попробуем задать более сложный:

> **В каком состоянии находится проект прямо сейчас?**

Например:

* Какова текущая архитектура?
* Какие решения являются действующими?
* Какие изменения сейчас выполняются?
* Какие задачи заблокированы?
* Почему они заблокированы?
* Кто должен устранить blocker?
* Какие компоненты затрагиваются?
* Какие риски существуют?
* Какие решения ожидаются?
* Что изменилось после вчерашней встречи?

Где находится ответ?

Обычно — **нигде**.

Ответ приходится каждый раз реконструировать:

(Git + Jira + Meeting notes + Slack + Email + Architecture docs + knowledge of several people) - Human - reconstructed project state

Причём разные участники могут реконструировать его по-разному.

### Информация существует, но состояние приходится вычислять

Здесь возникает важное архитектурное различие.

Существующие инструменты в основном хранят **события и артефакты**:

* commit happened
* message was sent
* meeting happened
* task was created
* PR was merged
* document was changed

Но команде необходимо знать **состояние**:

* Component X is being migrated.
* TASK-17 is blocked.
* ADR-31 is awaiting approval.
* Developer A owns migration.
* Component Y depends on TASK-17.
* Current milestone is at risk.

Можно представить это как:

Project Events - Message - Meeting - Commit - PR - Task - Decision - ??? - Current Project State

Именно отсутствующий слой `???` представляет для нас основной интерес.

### От набора информационных систем к памяти проекта

Следовательно, задача заключается не в создании ещё одного места, куда мы скопируем:

Git + Jira + Email + Chat + Meetings

Так мы получим только ещё одно большое хранилище.

Необходимо построить слой, способный преобразовать: "distributed project information" в: "structured project knowledge + current project state".
То есть перейти от: "Where is the information?" к: "What does the project currently know?" и далее: "What is the current state of the project?"

Именно здесь появляется идея **Project Memory** и связанного с ней **Project Digital Twin** — управляемого представления знаний, решений, задач, зависимостей и текущего состояния проекта.

Но прежде чем строить такую систему, необходимо разобраться с ещё одним очевидным подходом: если информация уже распределена между множеством источников, почему бы просто не проиндексировать всё с помощью embeddings и не дать агенту обычный RAG?

Как мы увидим далее, этого недостаточно.

## 3. Почему просто RAG недостаточно

Рассмотреть очевидное решение:

Everything
   ↓
Embedding
   ↓
Vector DB
   ↓
RAG
   ↓
Agent

Показать его ограничения.

Semantic similarity не отвечает на вопросы:

* актуально ли найденное решение;
* является ли оно утверждённым;
* было ли оно отменено;
* кто является владельцем решения;
* к какому проекту оно относится;
* можно ли этому агенту его видеть;
* является текст решением или просто предложением.

Пример:

"We could migrate to Kafka."
        ≠
"Decision: migrate to Kafka."

Ввести фундаментальное разделение:

Raw Information
       ≠
Validated Knowledge
       ≠
Current Project State

⸻

## 4. Концепция Team Agent Context Platform

Представить TACP.

Основная задача:

создать единое пространство проектного контекста, которым совместно пользуются люди и AI-агенты.

Показать три основных цикла системы.

Organizational Loop

Communication
      ↓
Understanding
      ↓
Project State
      ↓
Tasks
      ↓
People

Engineering Loop

Task
 ↓
Context
 ↓
Agent
 ↓
Code
 ↓
Review
 ↓
Merge

Learning Loop

Work
 ↓
Experience
 ↓
Knowledge
 ↓
Validation
 ↓
Project Memory
 ↓
Next Work

⸻

## 5. Архитектура TACP

Использовать подготовленную C4 L1 System Context Diagram.

Объяснить место системы между:

* разработчиками;
* руководителями;
* Git;
* Jira;
* корпоративными коммуникациями;
* встречами;
* CI/CD;
* LLM providers.

Главный архитектурный тезис:

Communication
     ↓
Organizational Intelligence
     ↓
Project Digital Twin
     ↓
Context Engineering
     ↓
AI Agents
     ↓
Engineering Work
     ↓
Knowledge Promotion
     └────────────► Project Knowledge

⸻

## 6. Organizational Intelligence: превращаем коммуникации в состояние проекта

Первая крупная подсистема TACP.

Источники:

Telegram / Slack / Teams
Email
Meetings
Jira
Git

Pipeline:

Sources
   ↓
Ingestion
   ↓
Normalization
   ↓
Understanding
   ↓
Entity Resolution
   ↓
Project Reasoning

⸻

## 7. Meeting Intelligence

Отдельно разобрать один показательный сценарий.

Meeting
   ↓
Recording
   ↓
Speech-to-Text
   ↓
Speaker Diarization
   ↓
Speaker Identification
   ↓
Transcript
   ↓
LLM Understanding

Из встречи извлекаются:

* topics;
* decisions;
* tasks;
* owners;
* deadlines;
* dependencies;
* risks;
* open questions.

Пример:

Sergey:
Я сначала подготовлю ADR для ACL.
Anna:
После этого я смогу начать Jira integration.

превращается в:

TASK-101
Create ACL ADR
Owner: Sergey
TASK-102
Implement Jira integration
Owner: Anna
TASK-102 depends_on TASK-101

⸻

## 8. От Summarization к Project Reasoning

Показать принципиальную разницу между meeting assistant и TACP.

Meeting assistant отвечает:

Что обсуждали?

TACP должна отвечать:

Что изменилось в проекте после этого обсуждения?

Например:

New Task
   ↓
Project Reasoning
   ↓
Check:
    Architecture
    ADR
    Dependencies
    Existing tasks
    Milestone
    Responsibilities
    Security constraints

⸻

## 9. Human-in-the-loop: почему LLM не должна самостоятельно управлять проектом

Рассмотреть опасную архитектуру:

Meeting
 ↓
LLM
 ↓
Jira Task
 ↓
Agent
 ↓
Code

Показать более безопасную:

Meeting
 ↓
LLM extraction
 ↓
TaskCandidate
 ↓
Project validation
 ↓
TaskProposal
 ↓
Human / Policy confirmation
 ↓
CommittedTask

Главный принцип:

AI предлагает изменение состояния проекта, но domain layer решает, допустимо ли это изменение.

⸻

## 10. Project Digital Twin

Ввести центральное понятие архитектуры.

Digital Twin содержит:

People
Projects
Components
Tasks
Dependencies
Milestones
Risks
Decisions
Responsibilities
Blockers

Пример:

MCP Router
    ↓ blocked by
ACL Design
    ↓ requires
ADR-021
    ↓ owned by
Sergey

Благодаря этому система может рассуждать не только о документах, но и о текущем состоянии проекта.

⸻

## 11. Три разных типа памяти

Один из центральных разделов статьи.

Evidence

Что фактически произошло:

* meeting transcript;
* email;
* Telegram message;
* PR comment.

Knowledge

Что команда считает правильным:

* ADR;
* architecture;
* pattern;
* runbook;
* policy.

State

Что происходит сейчас:

* task;
* blocker;
* milestone;
* responsibility;
* dependency.

Формула:

Evidence
   ≠
Knowledge
   ≠
State

Показать, почему смешивание этих типов приводит к ошибкам AI.

⸻

## 12. Git как canonical engineering memory

Обосновать использование Git для подтверждённого инженерного знания.

Структура:

AGENTS.md
docs/
    architecture/
    adr/
    plans/
    patterns/
    runbooks/
    known-issues/

Показать роль ADR.

Особенно важно сохранять:

не только что решили, но почему приняли решение и какие альтернативы отвергли.

⸻

## 13. Semantic Memory и pgvector

После canonical knowledge появляется поисковый слой:

Git
Documents
Tasks
Evidence
    ↓
Metadata
    +
Embeddings
    ↓
PostgreSQL + pgvector

Объяснить hybrid retrieval:

semantic similarity
        +
metadata
        +
authority
        +
freshness
        +
relationships
        +
permissions

⸻

## 14. Context Engineering как отдельная инженерная дисциплина

Ввести один из наиболее интересных тезисов статьи:

задача состоит не в том, чтобы передать агенту максимальный контекст, а в том, чтобы передать минимальный достаточный контекст.

Pipeline:

Task
 ↓
Scope Detection
 ↓
Hybrid Retrieval
 ↓
Authority Ranking
 ↓
Conflict Detection
 ↓
Token Budget
 ↓
Context Package

⸻

## 15. Task Context Package

Показать пример:

task:
  TASK-371
goal:
  Implement Jira MCP connector
project_state:
  milestone: MCP-MVP
architecture:
  - architecture/mcp.md
decisions:
  - ADR-014
  - ADR-021
dependencies:
  - TASK-355
constraints:
  - SEC-011
related_code:
  - services/mcp-router
evidence:
  - meeting-91#00:31:42

Agent получает это вместо сотен страниц conversation history.

⸻

## 16. Персональный AI-агент разработчика

Объяснить модель:

Developer
    ↓
Personal Agent
    ↓
Personal Work Context
    ↓
Team Context Platform

У агента нет собственной версии истины.

Personal Agent получает:

user
+
role
+
task
+
project state
+
engineering context
+
permissions

⸻

## 17. От встречи непосредственно к работе агента

Использовать end-to-end сценарий как центральный пример статьи.

Meeting
"Anna implements Jira integration
after Sergey finishes ACL ADR."
        ↓
Speech Recognition
        ↓
Task Extraction
        ↓
Project Reasoning
        ↓
TASK-101 → Sergey
TASK-102 → Anna
depends_on TASK-101
        ↓
Human confirmation
        ↓
Sergey Personal TODO
        ↓
Agent creates ADR
        ↓
ADR accepted
        ↓
TASK-101 DONE
        ↓
TASK-102 BLOCKED → READY
        ↓
Anna Personal Agent
        ↓
Context Package
        ↓
Implementation
        ↓
PR

И подчеркнуть:

Agent Anna никогда не видел исходную встречу.

Ему передано уже структурированное состояние проекта.

⸻

## 18. Knowledge Promotion: как проект обучается на собственной работе

После выполнения задачи:

Agent Run
PR
Review
Incident
       ↓
Knowledge Agent
       ↓
KnowledgeCandidate
       ↓
Human Review
       ↓
ADR / Pattern / Runbook
       ↓
Git
       ↓
Knowledge Index

Таким образом появляется organizational learning loop.

⸻

## 19. Может ли такая система действительно работать сегодня?

Здесь статья должна перейти от архитектурного предложения к критической оценке.

Разделить компоненты по зрелости.

Высокая техническая готовность

Сегодня достаточно надёжно реализуются:

* email/chat ingestion;
* meeting transcription;
* semantic search;
* RAG;
* Jira/Git integration;
* summarization;
* task extraction;
* coding agents;
* PR review;
* MCP/tool integration.

Средняя готовность

Работают, но требуют контроля:

* speaker identification;
* decision extraction;
* task ownership extraction;
* dependency extraction;
* knowledge promotion;
* architecture-aware reasoning.

Исследовательская/сложная зона

Наиболее сложны:

* построение достоверного Project Digital Twin;
* определение противоречащих решений;
* автоматическое определение устаревшего знания;
* reasoning по большим dependency graphs;
* autonomous project coordination;
* оценка того, какое знание действительно нужно сохранить.

⸻

## 20. Где TACP применима

Наиболее подходящие команды:

* software development;
* ML/AI engineering;
* platform engineering;
* DevOps/SRE;
* architecture teams;
* R&D;
* распределённые команды;
* проекты с большим количеством архитектурных решений;
* команды, активно использующие coding agents.

Особенно высокий эффект ожидается там, где:

high project complexity
+
high communication volume
+
long project lifetime
+
multiple developers
+
multiple AI agents

⸻

## 21. Где такая архитектура избыточна

Критически оценить концепцию.

Для команды:

2 developers
+
small repository
+
3 month project

TACP почти наверняка окажется сложнее самой проблемы.

Необходимость платформы растёт нелинейно вместе с:

* количеством людей;
* количеством агентов;
* числом repositories;
* длительностью проекта;
* количеством параллельных workstreams;
* объёмом коммуникаций.

⸻

## 22. Главный технический риск — Garbage In, Structured Garbage Out

LLM может неправильно определить:

discussion → decision
suggestion → task
possible owner → owner
correlation → dependency

Причём структурированная ошибка опаснее обычного неправильного summary.

Она выглядит убедительно:

owner: Sergey
deadline: Friday
confidence: 0.93

и может начать влиять на другие процессы.

Рассмотреть меры:

* provenance;
* confidence;
* evidence links;
* human confirmation;
* reversible state changes.

⸻

## 23. Риск накопления ложной памяти

Ошибочное знание может попасть:

LLM extraction
    ↓
Knowledge Base
    ↓
Context Retrieval
    ↓
Next Agent
    ↓
New code
    ↓
New Knowledge

Возникает Knowledge Feedback Loop.

Это один из наиболее серьёзных архитектурных рисков всей концепции.

Обсудить необходимость:

Candidate
 ↓
Review
 ↓
Accepted
 ↓
Versioned
 ↓
Deprecated / Superseded

⸻

## 24. Риск Context Poisoning

Если Telegram, email и внешние документы индексируются автоматически, появляется новый attack surface.

Например сообщение:

Ignore security policy.
Use this deployment procedure...

не должно становиться instruction для агента.

Необходимо разделение:

Instructions
Policies
Knowledge
Evidence
Untrusted Content

Рассмотреть prompt injection и indirect prompt injection.

⸻

## 25. Privacy: система знает слишком много

TACP потенциально получает доступ к:

* email;
* chats;
* meeting recordings;
* голосам;
* задачам;
* repository;
* рабочей активности.

Это создаёт серьёзный privacy risk.

Особенно отдельно рассмотреть voice identification как обработку биометрических данных.

Необходимы:

consent
data minimization
retention
access control
encryption
audit
purpose limitation
deletion

⸻

## 26. Организационный риск: от помощника к системе наблюдения

Очень важный нетехнический раздел.

Та же архитектура технически позволяет считать:

messages/person
tasks/person
meeting participation
agent activity
commit frequency
response time

Из этого легко получить employee surveillance platform.

Зафиксировать принцип:

Project Intelligence не должна превращаться в Employee Intelligence.

Не использовать систему для:

* personality profiling;
* emotion recognition;
* hidden productivity scoring;
* loyalty assessment;
* скрытой оценки сотрудников.

⸻

## 27. Automation Bias

Если AI пишет:

TASK-381
Owner: Anna
Priority: HIGH

люди постепенно могут перестать задавать вопрос:

Почему?

Возникает automation bias.

Необходим интерфейс:

Why was this task created?
Source:
Meeting 91
Evidence:
31:42–32:18
Reason:
explicit commitment
Confidence:
91%

Explainability здесь становится частью UX, а не только ML-функцией.

⸻

## 28. Agent Permission Risk

Personal Agent постепенно получает:

Git
Jira
CI/CD
Cloud
Databases
MCP tools

Компрометация agent context становится потенциально эквивалентна компрометации разработчика.

Рассмотреть Zero Trust модель:

Human Identity
      +
Agent Identity
      +
Delegation
      +
Task
      +
Tool
      +
Resource
      ↓
Policy Decision

⸻

## 29. Стоимость системы

Показать, что стоимость — не только LLM tokens.

Необходимо учитывать:

STT
LLM inference
embeddings
storage
vector search
audio/video
observability
GPU
engineering
security
knowledge maintenance

Причём самая дорогая часть может оказаться не inference, а поддержание корректной модели Project State.

⸻

## 30. Метрики: как понять, что TACP действительно полезна

Не ограничиваться:

summary accuracy

Предложить бизнес/engineering metrics:

Context Recovery Time

Сколько времени человеку или агенту требуется для входа в задачу.

Repeated Discussion Rate

Как часто команда повторно обсуждает уже решённые вопросы.

Repeated Mistake Rate

Повторяет ли новый агент уже исправленную ошибку.

Lost Decision Rate

Сколько решений не оказалось зафиксировано.

Unassigned Action Rate

Сколько договорённостей потеряно после встреч.

Knowledge Reuse Rate

Используются ли накопленные знания в следующих задачах.

Agent Rework Rate

Сколько результатов агента приходится переделывать из-за неправильного контекста.

⸻

## 31. Как проверить гипотезу без строительства всей платформы

Очень практический раздел статьи.

Предложить MVP:

1 project
5–10 developers
1 Git repository
1 Jira project
1 team chat
meeting recordings
PostgreSQL + pgvector
1 Context Service
1 Coding Agent
1 Knowledge Agent

Эксперимент:

Control:
Agent receives repository + task
Treatment:
Agent receives TACP Context Package

Сравнивать:

task completion
rework
architecture violations
context recovery time
token consumption
human interventions

⸻

## 32. Roadmap

Phase 1 — Project Knowledge

Git
ADR
AGENTS.md
pgvector
Context Builder

Phase 2 — Organizational Intelligence

Meetings
Email
Chat
Task extraction
Decision extraction

Phase 3 — Project Digital Twin

Tasks
Dependencies
Components
Milestones
Risks

Phase 4 — Personal Agents

TODO
Context Package
Coding Agent
Reviewer

Phase 5 — Learning Loop

Knowledge Agent
Knowledge Promotion
Automatic documentation maintenance

Phase 6 — Enterprise Governance

Zero Trust
Multi-project
RBAC/ABAC
Audit
Retention
Cross-project knowledge

⸻

## 33. Возможная целевая архитектура

Здесь разместить:

C4 L1 — System Context

затем:

C4 L2 — Container Architecture

и коротко объяснить bounded contexts:

Communication Intelligence
Media Intelligence
Identity & Participants
Project Knowledge
Project Digital Twin
Work Management
Context Engineering
Agent Execution
Knowledge Promotion
Governance & Security

Не уходить в детали каждого микросервиса — это материал для отдельной технической статьи.

⸻

## 34. Главный архитектурный вывод

Архитектура AI-assisted development постепенно может измениться от:

Human → AI Agent → Code

к:
```text
             Project Context
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Human     Human    Human
          │        │        │
          ▼        ▼        ▼
       Agent     Agent    Agent
          │        │        │
          └────────┼────────┘
                   ▼
                  Work
                   │
                   ▼
               Experience
                   │
                   ▼
                Knowledge
                   │
                   └──────► Project Context
```

Центральным элементом становится уже не AI Agent.

Им становится Project Context Platform.

⸻

## 35. Заключение: проект как самостоятельный носитель контекста

Вернуться к исходной проблеме.

Сегодня мы говорим:

«У моего AI-агента есть память».

Более масштабируемая модель:

«У проекта есть память, а люди и агенты получают из неё необходимый контекст».

Именно этот переход позволяет рассматривать AI-агентов не как персональные инструменты отдельных разработчиков, а как полноценных участников общей инженерной системы.

При этом TACP не следует рассматривать как готовый универсальный рецепт.

Это архитектурная гипотеза, ценность которой необходимо проверять экспериментально.

Главные вопросы:

1. Насколько хорошо система отличает обсуждение от решения?
2. Насколько точно она восстанавливает dependencies?
3. Снижает ли общий контекст количество ошибок агентов?
4. Не становится ли поддержание Digital Twin дороже получаемой пользы?
5. Можно ли обеспечить достаточный уровень privacy и security?
6. Не создаёт ли автоматизация новый организационный overhead?
7. Насколько человеку можно доверять сформированной AI моделью состояния проекта?

Если ответы окажутся положительными, следующий этап развития coding agents будет связан не столько с более мощными моделями, 
сколько с появлением инфраструктуры коллективного контекста — общей памяти для команд, состоящих одновременно из людей и AI-агентов.

Самые сильные части здесь — разделы 3, 10–18 и 19–31: сначала мы формулируем проблему и архитектуру, 
а затем сами же пытаемся её «сломать» через применимость, privacy, knowledge poisoning, automation bias, безопасность и стоимость. 
Это сделает материал значительно убедительнее обычного описания очередной multi-agent платформы.

