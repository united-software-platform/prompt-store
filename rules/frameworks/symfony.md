# Symfony-правила

Правила конфигурации Symfony-приложения: только PHP-файлы, без YAML и XML.

---

## Навигация

- [Таблица правил](#таблица-правил)
- [Конфигурация](#конфигурация-symf-001003)
- [Версионирование](#версионирование)

---

## Таблица правил

| ID | Правило |
|----|---------|
| SYMF-001 | Все конфигурации — только в `.php`-файлах |
| SYMF-002 | Запрет YAML-конфигов (`.yaml`, `.yml`) |
| SYMF-003 | Запрет XML-конфигов (`.xml`) |

---

## Конфигурация [SYMF-001..003]

- **[SYMF-001]** **Только PHP** — все конфигурации пишутся исключительно в `.php`-файлах (`config/*.php`, `config/packages/*.php`, `config/routes/*.php`).
- **[SYMF-002]** **Запрещены** любые конфигурационные файлы в формате YAML (`.yaml`, `.yml`) — включая `services.yaml`, `routes.yaml`, `packages/*.yaml` и любые другие.
- **[SYMF-003]** **Запрещены** любые конфигурационные файлы в формате XML (`.xml`).

### Пример

```php
// config/packages/framework.php
use Symfony\Config\FrameworkConfig;

return static function (FrameworkConfig $framework): void {
    $framework->secret('%env(APP_SECRET)%');
    $framework->httpMethodOverride(false);
};
```

```php
// config/services.php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $services = $container->services()
        ->defaults()
            ->autowire()
            ->autoconfigure();

    $services->load('App\\', '../src/')
        ->exclude('../src/{DependencyInjection,Entity,Kernel.php}');
};
```

```php
// config/routes.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import('../src/Controller/', 'attribute');
};
```

---

## Версионирование

| Версия | Дата | Задача | Описание изменений |
|--------|------|--------|--------------------|
| 1.0.0  | 2026-04-23 | — | Начальное создание |
| 1.0.1  | 2026-04-23 | — | Приведено к стандарту DOC-012, DOC-014, DOC-022 |
