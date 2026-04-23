# CQRS

Правила разделения операций чтения и записи: два отдельных интерфейса репозитория на один агрегат.

---

## Навигация

- [Таблица правил](#таблица-правил)
- [Принцип](#принцип)
- [Подробное описание](#подробное-описание)
- [Структура каталогов](#структура-каталогов)
- [Пример](#пример)
- [Версионирование](#версионирование)

---

## Таблица правил

| ID | Правило |
|----|---------|
| CQRS-001 | `*WriteRepositoryInterface` объявляется в Domain, содержит только мутации |
| CQRS-002 | WriteRepository: запрет методов чтения (`findById` и др.) |
| CQRS-003 | `*ReadRepositoryInterface` объявляется в Application, возвращает только read-DTO |
| CQRS-004 | ReadRepository: запрет мутаций |
| CQRS-005 | UseCase-запросы (Query) зависят только на `*ReadRepositoryInterface` |
| CQRS-006 | Реализации Write и Read — отдельные классы в Infrastructure |

---

## Принцип

Операции чтения и записи разделены на уровне интерфейсов репозиториев. Один агрегат — два контракта:

| Интерфейс | Слой | Назначение |
|-----------|------|------------|
| `TaskWriteRepositoryInterface` | Domain | Работает с доменной моделью: `create`, `update`, `delete` |
| `TaskReadRepositoryInterface` | Application | Возвращает read-DTO напрямую из SQL |

## Подробное описание

- **[CQRS-001]** **`*WriteRepositoryInterface`** объявляется в **Domain**. Методы принимают и возвращают доменные объекты (`Task`, `TaskId`). Только мутации и генерация идентификаторов.
- **[CQRS-002]** WriteRepository: **методы чтения (`findById` и др.) запрещены** — если команде нужно загрузить агрегат, она использует `*ReadRepositoryInterface`.
- **[CQRS-003]** **`*ReadRepositoryInterface`** объявляется в **Application** — это контракт чтения, реализация которого обращается к инфраструктуре. Методы возвращают read-DTO (`*View`, `*Detail`, `*Summary`).
- **[CQRS-004]** ReadRepository: никаких мутаций.
- **[CQRS-005]** **UseCase-запросы** зависят только на `*ReadRepositoryInterface`.
- **[CQRS-006]** Реализации обоих интерфейсов — **отдельные классы** в Infrastructure-слое.

## Структура каталогов

```text
src/{Module}/
├── Domain/
│   └── Repository/
│       └── TaskWriteRepositoryInterface.php   # write — доменная модель
├── Application/
│   ├── Repository/
│   │   └── TaskReadRepositoryInterface.php    # read — контракт DTO
│   └── UseCase/
│       ├── GetTask/          # Query — зависит на TaskReadRepositoryInterface
│       └── UpdateTask/       # Command — зависит на TaskWriteRepositoryInterface
└── Infrastructure/
    └── Persistence/
        ├── PdoTaskWriteRepository.php         # реализует TaskWriteRepositoryInterface
        └── PdoTaskReadRepository.php          # реализует TaskReadRepositoryInterface
```

## Пример

```php
// Domain — write
interface TaskWriteRepositoryInterface
{
    public function create(Task $task): void;
    public function update(Task $task): void;
    public function delete(TaskId $id): void;
}

// Application — read
interface TaskReadRepositoryInterface
{
    public function findById(int $id): TaskDetailView;
    /** @return list<TaskSummaryView> */
    public function listByStoryId(int $storyId): array;
}

// Application — Command UseCase
final class UpdateTaskUseCase implements UpdateTaskUseCaseInterface
{
    public function __construct(
        private readonly TaskWriteRepositoryInterface $tasks,
    ) {}
}

// Application — Query UseCase
final class GetTaskUseCase implements GetTaskUseCaseInterface
{
    public function __construct(
        private readonly TaskReadRepositoryInterface $tasks,
    ) {}
}

// Infrastructure — write
final readonly class PdoTaskWriteRepository implements TaskWriteRepositoryInterface
{
    public function create(Task $task): void { ... }
    public function update(Task $task): void { ... }
    public function delete(TaskId $id): void { ... }
}

// Infrastructure — read
final readonly class PdoTaskReadRepository implements TaskReadRepositoryInterface
{
    public function findById(int $id): TaskDetailView { ... }
    public function listByStoryId(int $storyId): array { ... }
}
```

---

## Версионирование

| Версия | Дата | Задача | Описание изменений |
|--------|------|--------|--------------------|
| 1.0.0  | 2026-04-23 | — | Начальное создание |
| 1.0.1  | 2026-04-23 | — | Приведено к стандарту DOC-012, DOC-014, DOC-016, DOC-022 |
