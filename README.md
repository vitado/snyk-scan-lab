# Snyk Scan Lab

Параметризованный стенд для ручного запуска Snyk-сканов на **произвольных** публичных репозиториях или образах. Результаты публикуются в Security → Code Scanning и сохраняются как артефакт сборки.

## Предварительные условия

Добавьте секрет `SNYK_TOKEN` в репозиторий (бесплатный Snyk всё равно требует авторизацию):

```bash
gh secret set SNYK_TOKEN
```

Токен можно получить на [app.snyk.io/account](https://app.snyk.io/account).

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

- **Security → Code scanning** — все найденные уязвимости с аннотациями.
- **Run → Artifacts → snyk-sarif** — файл `snyk.sarif` для локального анализа / передачи в другие инструменты.

## Примечание

Этот репозиторий является *инструментом сканирования*, а не объектом скана. Он не обязан содержать сканируемый код.
