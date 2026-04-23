# Чистая архитектура (Robert C. Martin)

Правила организации кода по принципам Clean Architecture и SOLID: направление зависимостей, слои, модульная структура.

---

## Навигация

- [Таблица правил](#таблица-правил)
- [Слои и направление зависимостей](#слои-и-направление-зависимостей-clar-001)
- [Domain](#domain-ядро-без-зависимостей-на-фреймворк-clar-002006)
- [Application](#application-оркестрация-use-cases-clar-007013)
- [Infrastructure](#infrastructure-детали-реализации-clar-014016)
- [SOLID](#solid-clar-017021)
- [Модульная организация](#модульная-организация-module-first-clar-022023)
- [Структура каталогов](#структура-каталогов)
- [Пример](#пример)
- [Версионирование](#версионирование)

---

## Таблица правил

| ID | Правило |
|----|---------|
| CLAR-001 | Зависимости строго внутрь: `Infrastructure → Application → Domain` |
| CLAR-002 | Domain содержит только бизнес-правила: Entities, VO, Events, Repository interfaces, Domain Services |
| CLAR-003 | Repository-методы `create`/`update` принимают доменную модель целиком (не скаляры, не DTO, не массивы) |
| CLAR-004 | Domain: запрет зависимостей на Symfony, Doctrine, PSR-контейнер, HTTP |
| CLAR-005 | Domain: → см. PHP-002, PHP-003, PHP-004 (применяются без исключений) |
| CLAR-006 | Domain: запрет интерфейсов, реализация которых требует обращения к инфраструктуре |
| CLAR-007 | Application: каждая бизнес-операция — отдельный UseCase, одно действие (SRP) |
| CLAR-008 | UseCase — единственный публичный метод `execute(XxxInput): XxxOutput\|void` |
| CLAR-009 | `Input` и `Output` — `readonly`-объекты без логики |
| CLAR-010 | UseCase зависит только на Domain-интерфейсы, не на конкретные реализации |
| CLAR-011 | Каждый UseCase живёт в собственном каталоге `Application/UseCase/OperationName/` |
| CLAR-012 | Application: запрет прямых вызовов Doctrine, PDO, HTTP-клиентов |
| CLAR-013 | Application: запрет бизнес-логики (инварианты, расчёты — в Domain) |
| CLAR-014 | Infrastructure: репозиторий персистирует ровно один агрегат |
| CLAR-015 | Infrastructure: запрет бизнес-логики |
| CLAR-016 | Infrastructure: контроллер не зависит на Domain-объекты напрямую, только через Application |
| CLAR-017 | SOLID/SRP — один класс, одна причина меняться |
| CLAR-018 | SOLID/OCP — расширение через новые классы, не модификацию существующих |
| CLAR-019 | SOLID/LSP — реализация полностью соблюдает контракт интерфейса |
| CLAR-020 | SOLID/ISP — интерфейсы узкие и сфокусированные |
| CLAR-021 | SOLID/DIP — верхние слои зависят на абстракции, объявленные в Domain |
| CLAR-022 | Структура Module-first: группировка по доменным модулям, не по техническим слоям |
| CLAR-023 | Межмодульные зависимости — только через Domain-интерфейсы |

---

## Слои и направление зависимостей [CLAR-001]

Зависимости направлены **строго внутрь**: `Infrastructure → Application → Domain`.
Внутренние слои ничего не знают о внешних.

```text
┌─────────────────────────────────┐
│         Infrastructure          │  Frameworks, DB, HTTP, CLI, Queue
│  ┌───────────────────────────┐  │
│  │       Application         │  │  Use Cases, Commands, Queries, DTO
│  │  ┌─────────────────────┐  │  │
│  │  │       Domain        │  │  │  Entities, Value Objects, Events,
│  │  │                     │  │  │  Domain Services, Repository interfaces
│  │  └─────────────────────┘  │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

---

## Domain (ядро, без зависимостей на фреймворк) [CLAR-002..006]

Содержит **бизнес-правила**: валидацию, форматирование кодов, формулы расчётов, инварианты.

- **[CLAR-002]** **Entities** — бизнес-объекты с идентичностью; иммутабельные свойства через `readonly`. **Value Objects** — объекты без идентичности, всегда иммутабельны. **Domain Events** — факты домена, только данные. **Domain Services** — операции над несколькими агрегатами.
- **[CLAR-003]** **Repository interfaces** — контракты (`interface`) для доступа к данным; реализация — в Infrastructure. Методы `create` и `update` принимают **доменную модель целиком** — передавать отдельные скалярные аргументы, DTO или массивы **запрещено**.

**Запрещено в Domain:**
- **[CLAR-004]** Любые зависимости на Symfony, Doctrine, PSR-контейнер, HTTP и т.п.
- **[CLAR-005]** → см. PHP-002, PHP-003, PHP-004 (применяются без исключений).
- **[CLAR-006]** Интерфейсы и сервисы, чья реализация требует обращения к инфраструктуре (БД, очереди, HTTP) — они принадлежат Application.

---

## Application (оркестрация, use cases) [CLAR-007..013]

Содержит **правила приложения**: оркестрацию шагов, генераторы (сервисы, чья реализация обращается к инфраструктуре), интерфейсы таких сервисов.

**[CLAR-007]** Каждая бизнес-операция — отдельный UseCase. Один UseCase — одно действие (SRP).

### Структура UseCase

| Класс | Суффикс | Назначение |
|-------|---------|------------|
| `ApproveRequirementUseCase` | `UseCase` | Реализация; `final class` |
| `ApproveRequirementUseCaseInterface` | `UseCaseInterface` | Контракт; зависимость контроллера — на интерфейс |
| `ApproveRequirementInput` | `Input` | Иммутабельный входной DTO (`readonly`) |
| `ApproveRequirementOutput` | `Output` | Иммутабельный выходной DTO (`readonly`); если нет данных — `void` |

### Правила UseCase

- **[CLAR-008]** Единственный публичный метод: `execute(XxxInput $input): XxxOutput|void`.
- **[CLAR-009]** `Input` и `Output` — `readonly`-объекты без логики.
- **[CLAR-010]** UseCase зависит только на **Domain**-интерфейсы, не на конкретные реализации.
- **[CLAR-011]** Каждый UseCase живёт в собственном каталоге: `Application/UseCase/ApproveRequirement/`.

**Запрещено в Application:**
- **[CLAR-012]** Прямые вызовы Doctrine, PDO, HTTP-клиентов — только через Domain-интерфейсы.
- **[CLAR-013]** Бизнес-логика (инварианты, расчёты) — она принадлежит Domain.

---

## Infrastructure (детали реализации) [CLAR-014..016]

- **Repository implementations** — реализуют Domain-интерфейсы через Doctrine/PDO/etc.
- **Controllers** — принимают HTTP-запрос, вызывают Application-хендлер, возвращают ответ.
- **Persistence (Entities mapping)** — Doctrine mapping-конфигурации.
- **External services** — HTTP-клиенты, очереди, файловая система.

**Запрещено в Infrastructure:**
- **[CLAR-015]** Бизнес-логика.
- **[CLAR-016]** Прямые зависимости контроллера на Domain-объекты, минуя Application.
- **[CLAR-014]** Репозиторий персистирует ровно один агрегат. Всё необходимое (сгенерированные значения, коды, внешние id) передаётся в него как готовый аргумент из UseCase.

---

## SOLID [CLAR-017..021]

| ID | Принцип | Правило |
|----|---------|---------|
| **CLAR-017** | **SRP** — Single Responsibility | Один класс — одна причина меняться. Handler обрабатывает ровно одну команду. |
| **CLAR-018** | **OCP** — Open/Closed | Расширять поведение через новые классы (новый Handler, новый Specification), не модифицируя существующие. |
| **CLAR-019** | **LSP** — Liskov Substitution | Реализация интерфейса полностью соблюдает его контракт; не бросает неожиданных исключений, не игнорирует параметры. |
| **CLAR-020** | **ISP** — Interface Segregation | Интерфейсы узкие и сфокусированные. `UserRepository` не содержит методов `OrderRepository`. |
| **CLAR-021** | **DIP** — Dependency Inversion | Верхние слои зависят на абстракции (интерфейсы), объявленные в Domain. Infrastructure подставляет конкретные реализации через DI-контейнер. |

---

## Модульная организация (Module-first) [CLAR-022..023]

**[CLAR-022]** Весь код группируется по **доменным модулям**, а не по техническим слоям.

**Запрещено** — плоская структура по слоям:

```text
src/
├── Domain/
├── Application/
└── Infrastructure/
```

**Обязательно** — каждый модуль содержит свои слои:

```text
src/
└── {DomainModule}/
    ├── Domain/
    ├── Application/
    └── Infrastructure/
```

**[CLAR-023]** Каждый модуль — самостоятельный bounded context. Межмодульные зависимости — только через Domain-интерфейсы, не через прямые вызовы классов соседнего модуля.

---

## Структура каталогов

```text
src/
└── {DomainModule}/               # например: Requirement, User, Project
    ├── Domain/
    │   ├── Model/          # Entities, Value Objects
    │   ├── Event/          # Domain Events
    │   ├── Repository/     # Interfaces
    │   └── Service/        # Domain Services
    ├── Application/
    │   └── UseCase/
    │       ├── ApproveRequirement/   # Input, Output, Interface, UseCase
    │       └── GetRequirement/       # Input, Output, Interface, UseCase
    └── Infrastructure/
        ├── Persistence/    # Doctrine Repositories, Migrations
        ├── Http/           # Controllers
        └── External/       # HTTP-клиенты, очереди
```

---

## Пример

```php
// Domain — интерфейс репозитория
namespace App\Requirement\Domain\Repository;

interface RequirementRepositoryInterface
{
    public function findById(RequirementId $id): Requirement;
    public function save(Requirement $requirement): void;
}

// Application — UseCase
namespace App\Requirement\Application\UseCase\ApproveRequirement;

final class ApproveRequirementInput
{
    public function __construct(
        public readonly string $requirementId,
        public readonly string $approvedBy,
    ) {}
}

interface ApproveRequirementUseCaseInterface
{
    public function execute(ApproveRequirementInput $input): void;
}

final class ApproveRequirementUseCase implements ApproveRequirementUseCaseInterface
{
    public function __construct(
        private readonly RequirementRepositoryInterface $requirements,
    ) {}

    public function execute(ApproveRequirementInput $input): void
    {
        $requirement = $this->requirements->findById(
            new RequirementId($input->requirementId)
        );
        $requirement->approve($input->approvedBy);
        $this->requirements->save($requirement);
    }
}

// Infrastructure — реализация репозитория
namespace App\Requirement\Infrastructure\Persistence;

final class DoctrineRequirementRepository implements RequirementRepositoryInterface
{
    public function __construct(
        private readonly EntityManagerInterface $em,
    ) {}

    public function findById(RequirementId $id): Requirement
    {
        return $this->em->find(Requirement::class, $id->value)
            ?? throw new RequirementNotFound($id);
    }

    public function save(Requirement $requirement): void
    {
        $this->em->persist($requirement);
    }
}
```

---

## Версионирование

| Версия | Дата | Задача | Описание изменений |
|--------|------|--------|--------------------|
| 1.0.0  | 2026-04-23 | — | Начальное создание |
| 1.0.1  | 2026-04-23 | — | Приведено к стандарту DOC-012, DOC-014, DOC-016, DOC-022 |
