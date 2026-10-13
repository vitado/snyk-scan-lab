# Snyk Scan Lab

Параметризованный стенд для ручного запуска Snyk-сканов на **произвольных** публичных репозиториях или образах. Результаты скачиваются как артефакт прогона (`snyk.json` + `snyk.sarif`). Пригоден для переска: тот же workflow с другими параметрами.

## Предварительные условия

Добавьте секрет `SNYK_TOKEN` в репозиторий (бесплатный Snyk всё равно требует авторизацию):

```bash
gh secret set SNYK_TOKEN -R vitado/snyk-scan-lab
```

Токен можно получить на [app.snyk.io/account](https://app.snyk.io/account) (General → API Token).

## Как запустить

**Через UI:**  
Actions → "Snyk scan" → Run workflow → выбрать `type` / заполнить `target` / выбрать `severity`.

**Через CLI:**

```bash
# SCA-скан репозитория
gh workflow run snyk-scan.yml \
  -f type=sca \
  -f target=epam/ai-dial-chat \
  -f severity=high

# SAST-скан конкретной ветки
gh workflow run snyk-scan.yml \
  -f type=code \
  -f target=epam/ai-dial-core \
  -f ref=main \
  -f severity=medium

# Container-скан образа
gh workflow run snyk-scan.yml \
  -f type=container \
  -f target=alpine:3.19 \
  -f severity=critical
```

## Параметры

| Параметр   | Значения                             | По умолчанию | Описание |
|------------|--------------------------------------|--------------|----------|
| `type`     | `sca` / `code` / `container`         | `sca`        | Тип скана |
| `target`   | строка, обязательный                 | —            | **owner/repo** для sca/code; **image:tag** для container |
| `ref`      | ветка/тег/SHA                        | `""` (default branch) | Git-реф для sca/code |
| `severity` | `low` / `medium` / `high` / `critical` | `low`      | Минимальный уровень severity для репорта |

> **Важно:** для типов `sca` и `code` параметр `target` должен быть слагом вида `owner/repo`
> (например `epam/ai-dial-chat`), потому что используется `actions/checkout`.  
> Для типа `container` — полный image ref (`alpine:3.19`, `ghcr.io/org/image:tag`).

## Где смотреть результаты

- **Run → Artifacts → `snyk-results-<type>`** — архив с `snyk.sarif` и `snyk.json`
  (для sca/container). Скачай и открой: `.json` — читаемый список уязвимостей,
  `.sarif` — для импорта в другие инструменты.
- **Лог шага скана** в прогоне — текстовый вывод Snyk с найденными проблемами.

Специально **не** публикуем в Security → Code scanning: стенд сканирует чужие
произвольные таргеты, их уязвимости не относятся к этому репо и на вкладке Security
только путали бы. Code scanning уместен, когда репо = то, что сканируешь.

## Ограничения (free-tier Snyk)

- `code` (SAST) требует включённого **Snyk Code** в настройках организации
  (app.snyk.io → Settings → Snyk Code). Иначе — `SNYK-CODE-0005 / 403`.
- Есть месячный лимит тестов на free-плане.

## Примечание

Этот репозиторий — *инструмент сканирования*, а не объект скана. Он не обязан содержать сканируемый код.
