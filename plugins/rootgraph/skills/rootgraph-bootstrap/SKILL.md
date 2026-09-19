---
name: rootgraph-bootstrap
description: "Как довести проект до базового набора документов после подключения к Rootgraph: архитектурная карта, обогащение сгенерированных документов, разбор незавершённой работы и техдолга, черновик конвенций. Загружать при /rootgraph-bootstrap, при подсказке SessionStart про базовые документы, при блоке bootstrap в результатах rootgraph_init/rootgraph_index и при просьбе «собери архитектуру/документацию проекта»."
---

# Инициализация проекта: базовые документы, незавершённая работа, архитектура

## Зачем

Сборка идёт в два слоя.

- **Слой 1 — детерминированный, мгновенный, без LLM.** Клиент сам, в конце
  первой индексации (и при последующих), собирает по фактам репозитория семь
  базовых документов (`project-overview`, `architecture`, `modules`,
  `data-model`, `infrastructure`, `development-guide`, `unfinished-work`) и
  находит незавершённую работу и техдолг: заглушки, отключённые тесты,
  `deprecated`, незакрытые чекбоксы в README/docs, слишком большие файлы и
  функции, файлы без тестов. Находки попадают в задачи со статусом
  `pending_review` и источником `bootstrap`: ничего не принимается молча.
- **Слой 2 — агент.** Ты (с параллельными субагентами) обогащаешь
  сгенерированные части документов смыслом, создаёшь компоненты
  архитектурной карты, разбираешь находки по одной и готовишь черновик
  конвенций.

## Протокол

1. `rootgraph_status`. Если `configured: false` — остановись, нужен `rootgraph_init`.
2. `rootgraph_bootstrap_status`: шаги, документы, счётчики, поле `next`.
3. Если шаг `base_documents` не `done` — вызови `rootgraph_bootstrap_run` (быстро,
   без LLM; сам запускает индексацию, если проект ещё не индексировался).
   Флаги `refresh` и `force` передавай, только если их прямо указал
   пользователь; `force` перезаписывает правки людей и агентов, это решение
   только пользователя.
4. `rootgraph_bootstrap_plan` (можно `phase`: `components`, `documents`,
   `triage`, `conventions`) и выполни поле `instruction` дословно: для КАЖДОГО
   batch отдельный субагент через Task (`general-purpose`), все вызовы Task
   ОДНИМ сообщением, чтобы они шли параллельно. Размеры пачек: модули по 8
   ключей, триаж по 15 задач.
5. Если `estimated_subagents` больше 8, а вход был автоматической подсказкой
   (SessionStart, блок в результате `rootgraph_init`/`rootgraph_index`), а не
   явной командой `/rootgraph-bootstrap` — сначала назови пользователю число
   и спроси, продолжать ли. Явная команда даёт согласие на ≤ 8 субагентов;
   больше 8 — спроси всё равно.
6. После возврата субагентов вызови `rootgraph_bootstrap_mark(step="architecture_review",
   status="done", note)` и `rootgraph_bootstrap_status`. Не больше 2 раундов.
   Пользователю — одна строка итога.

## Язык

Язык проекта задаётся при создании проекта (`en` или `ru`) и приходит в
каждом результате bootstrap-инструментов и `rootgraph_files_undescribed`
(поля `language`, `language_name`, `project_language`). Все тексты для проекта
(тела частей документов, названия и описания задач, описания файлов) пиши на
этом языке.

Всегда английскими остаются и никогда не переводятся: slug документов, ключи
частей, названия документов и частей, id шагов (`index`, `file_descriptions`,
`base_documents`, `components`, `architecture_review`, `tasks_scan`,
`techdebt_scan`, `conventions_draft`, `secrets`), значение источника
`bootstrap`, имена компонентов и символов как в коде. Инструмент
`rootgraph_bootstrap_doc_part_save` отклонит кириллицу в ключах и названиях.

## Правила

- Не перезаписывай части с `edited_by: human` (и `agent`, если пользователь не
  просил): сервер и так оставит их как есть и вернёт `kept_edited`.
- Не выдумывай факты. Не уверен — оставь как есть.
- Никаких секретов и значений из `.env`: в документы попадают только имена
  переменных. Клиент блокирует находки сканера секретов.
- Находки разбирай по одной: принять — `rootgraph_task_update(id, status="open",
  title и description на языке проекта, priority)`; отклонить —
  `rootgraph_task_comment_add(id, «отклонено: причина»)` и `rootgraph_task_close(id)`.
  Массово не закрывай и ничего не пропускай молча.
- Компоненты (`rootgraph_component_add`) — это рантайм-архитектура: сервисы,
  хранилища, хосты, домены, прокси. Каталоги кода — не компоненты, они живут
  в документе `modules`.

## Стиль текста модулей

Разделы Responsibility / Public surface / Dependencies / Dependents: что модуль
делает и зачем, 2–5 предложений; таблицы и Mermaid сохраняй, если верны; без
пересказа кода и без ссылок на номера строк.

## Промпты субагентов (дословно из instruction плана)

[document] «Прочитай текущие части документа {slug}: rootgraph_bootstrap_doc_get(slug, keys). Изучи код проекта (Read/Grep) и перепиши каждую часть из part_keys: убери неточности, добавь смысл (что и зачем), сохрани таблицы и Mermaid, где они верны. Язык текста — {lang_name}. Ключи и названия частей не меняй, всё остаётся на английском в идентификаторах. Не выдумывай: если не уверен, оставь как есть. В конце ОДИН раз вызови rootgraph_bootstrap_doc_part_save(slug, items=[{key, content_md}]) и верни только «сохранено N».»

[modules] «Для каждого ключа из part_keys документа modules прочитай часть (rootgraph_bootstrap_doc_get), открой файлы каталога компонента, перепиши разделы Responsibility/Public surface/Dependencies/Dependents точнее, на языке {lang_name}, сохрани через rootgraph_bootstrap_doc_part_save; верни «сохранено N».»

[triage] «Для каждой задачи из списка открой file_path (Read) вокруг line, реши: реальная проблема — вызови rootgraph_task_update(id, status="open", title и description на языке {lang_name}, priority по важности); ложное срабатывание — rootgraph_task_comment_add(id, «отклонено: причина») и rootgraph_task_close(id). Ничего не пропускай молча; верни «принято A, отклонено B».»

[components] «Используй candidates и файлы конфигурации (compose, CI, k8s): для каждого реального сервиса/хранилища/хоста/домена/прокси вызови rootgraph_component_add(kind, name, description на языке {lang_name}), затем rootgraph_component_link(component, depends_on) по зависимостям и rootgraph_component_link(component, file_path) для 1–3 главных файлов. Затем rootgraph_bootstrap_mark(step="components", status="done", note).»

[conventions] «Запусти навык rootgraph-conventions-wizard в режиме «черновик из фактов»: предзаполни ответы по rootgraph_bootstrap_doc_get(slug="development-guide") и конфигам линтеров, показывай пользователю секцию за секцией и сохраняй через rootgraph_conventions_save только подтверждённое.»

## Когда остановиться

После 2 раундов — не запускай третий. Скажи одной строкой, что осталось
(`rootgraph_bootstrap_status`), и предложи `/rootgraph-bootstrap`. Если сервер
ответил «не поддерживает bootstrap» (`status: unsupported`) — не повторяй,
скажи пользователю обновить сервер.
