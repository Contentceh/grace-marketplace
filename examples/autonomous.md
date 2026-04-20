# GRACE Autonomy Guide: `grace lint --profile autonomous`

Этот документ — краткий практический гайд по тому, как работает `grace lint --profile autonomous` и как повысить автономность GRACE-проекта.

## Что это за команда

`grace lint --profile autonomous` — это **gate готовности к длительному автономному исполнению**, а не «режим автопилота».

Команда проверяет, что:
- проект имеет достаточные shared-артефакты;
- верификация модулей исполнима и наблюдаема;
- handoff между агентами формализован через operational packets;
- лог/trace evidence реально присутствует в runtime-коде.

Опорные места в коде CLI:
- `src/grace-lint.ts` — профиль `standard | autonomous`
- `src/lint/core.ts` — `lintAutonomousReadiness(...)`
- `src/lint/catalog.ts` — коды проблем + remediation
- `src/query/core.ts` — индекс модулей/файлов/верификации

## Что проверяет профиль `autonomous`

### 1) Operational packets
Требуется `docs/operational-packets.xml` с обязательными секциями:
- `ExecutionPacketTemplate`
- `GraphDeltaTemplate`
- `VerificationDeltaTemplate`
- `FailurePacketTemplate`
- `CheckpointReportTemplate`

Также ожидаются поля для автономного цикла:
- `assumptions`
- `stop-conditions`
- `retry-budget`
- `checkpoint-fields`

### 2) Технологический якорь
Требуется `docs/technology.xml`.
Желательно (warning при отсутствии):
- `<PreferredAgentStack>`
- `<AutonomyPolicy>`

### 3) Модульный минимум
Для каждого shared module из plan/graph:
- должен быть `V-M-*` entry в `docs/verification-plan.xml`;
- должны быть linked runtime-файлы (не только тесты);
- шаги в `docs/development-plan.xml` должны иметь `verification="V-M-..."`.

### 4) Полнота verification entry
Для каждого `V-M-*` требуются:
- `test-files`
- `module-checks`
- `scenarios` (success/failure)
- `required-log-markers` или `required-trace-assertions`

Дополнительно:
- test file должен существовать на диске;
- governed test file должен быть linked к модулю;
- module-check желательно явно ссылается на test file/dir (warning);
- `wave-follow-up` и `phase-follow-up` — рекомендуются (warning при отсутствии).

### 5) Реальная наблюдаемость evidence
Проверка не засчитывает:
- маркеры, встречающиеся только в тестах;
- «мертвые» строковые литералы в runtime без лог/emit/trace вызова.

Если marker формата `...[BLOCK_X]`, то должен существовать соответствующий semantic block (`START_BLOCK_X ... END_BLOCK_X`) в linked runtime-файлах.

## Как повысить автономность проекта

1. Пройти gate без ошибок:

```bash
grace lint --profile autonomous --path . --remediate --fail-on warnings
```

2. Держать `docs/operational-packets.xml` как стабильный контракт handoff.

3. Для каждого модуля поддерживать полноценный `V-M-*`:
- test-files
- module-checks
- scenarios
- observable evidence
- wave/phase follow-up

4. Дублировать доказательства в runtime, а не только в тестах:
- стабильные лог-маркеры;
- привязка к `BLOCK_*`.

5. Жестко поддерживать ownership через `LINKS: M-...` в `MODULE_CONTRACT` у runtime и governed test-файлов.

6. Явно задавать execution policy в `technology.xml`:
- preferred runtime/test stack;
- retry budget;
- stop/replan правила.

## Важный вывод

`grace lint --profile autonomous` не «включает автономность» сам по себе.

Он делает проект **менее двусмысленным, более проверяемым и наблюдаемым**, что и повышает реальную автономность агентного выполнения.
