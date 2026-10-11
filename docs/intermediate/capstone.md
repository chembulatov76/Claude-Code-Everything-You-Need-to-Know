# Intermediate capstone: get a repository ready for a new teammate

<sub>**Intermediate** · Capstone · 30 to 60 minutes · Needs: the eight [Intermediate lessons](README.md), or their skills · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

Это итоговое задание объединяет все навыки уровня Intermediate. Здесь нет пошагового разобранного примера: техническое задание формулирует потребности команды, а конкретную реализацию вы выбираете сами. Раздел [Intermediate Exit](README.md#by-the-end-of-this-level-you-can) перечисляет умения, подтверждаемые по итогам этого уровня. Скрипт проверки оценивает закоммиченные файлы, а оставшиеся пункты вы отмечаете самостоятельно в разделе [What the check can't see](#what-the-check-cant-see). <details style="display:inline"><summary><code>[en]</code></summary>This task puts the whole Intermediate level together. It has no worked example: the brief says what the team needs, and you choose how to build it. The Intermediate Exit is the list of what you can do by the end of this level.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений и Node.js LTS. Итоговое задание выполняется исключительно в учебном шаблоне. Если `git status --porcelain` показывает файлы, закоммитьте их или отложите через `git stash push --include-untracked`.
- Пакеты `jq` и `python3` для скриптов хуков, а также Google Chrome для запуска MCP-сервера.
- macOS, Linux или WSL 2: песочнице требуется одна из этих сред ([Set up on Windows](https://code.claude.com/docs/en/setup#set-up-on-windows)), а скрипты хуков написаны на bash.
- Расход токенов: умеренный (moderate).

## The task

В понедельник к команде присоединяется новый разработчик. Техническое задание находится в файле `capstone/intermediate-brief.md` вашей учебной копии: настройте репозиторий так, чтобы сессии Claude Code нового сотрудника работали по стандартам команды, и закоммитьте конфигурацию в git, поскольку коллега получит только то, что зафиксировано в репозитории. <details style="display:inline"><summary><code>[en]</code></summary>A teammate starts on Monday. The brief is in capstone/intermediate-brief.md in your practice copy: set the repository up so their Claude Code sessions work the way the team's do, and commit all of it, because they get only what's in git.</details>

Реализуйте каждый элемент согласно урокам уровня:

1. **Навык (skill)**, который запускает проверку ссылок для указанной папки и объясняет каждую неработающую ссылку. Навык запускается по имени и подгружается автоматически при вопросе о проверке ссылок ([урок 1](01-first-skill.md)).
2. **Память проекта:** `CLAUDE.md` импортирует `package.json`, а правила тестирования оформлены отдельным файлом в `.claude/rules/` с областью действия, ограниченной файлами в `test/` ([урок 2](02-organize-memory.md)).
3. **Настройки команды** в закоммиченном `.claude/settings.json`: запрет чтения `.env` и `.env.*`, разрешение команды `npm test` и включение песочницы ([урок 4](04-permissions-and-sandbox.md)). Команда `/sandbox` сохраняет настройки в локальный файл, недоступный коллегам, поэтому добавьте `"sandbox": {"enabled": true}` в общий файл.
4. **Хук (hook)**, блокирующий инструменты Edit и Write внутри каталога `samples/` и только там: тесты зависят от неизменности этих файлов ([урок 5](05-hooks.md)).
5. **Субагент-ревьюер** исключительно с инструментами для чтения ([урок 6](06-subagents.md)).
6. **Chrome DevTools MCP** на уровне проекта с зафиксированной версией пакета и отключённым сбором телеметрии, CrUX и проверкой обновлений ([урок 7](07-mcp.md)).
7. **Плагин `claude-code-setup`**, включённый на уровне проекта для всех разработчиков и установленный ([урок 8](08-plugins.md)).

Затем в сессии протестируйте каждый компонент: запустите навык по имени, попросите Claude отредактировать файл в `samples/` через Edit, делегируйте задачу ревьюеру, подтвердите подключение MCP-сервера и выполните один из его инструментов. Подбирайте модель и уровень усилий под каждую задачу, как показано в [уроке 3](03-model-and-effort.md). Закоммитьте все изменения и убедитесь, что `npm test` завершается успешно.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный навык проекта запускает утилиту проверки ссылок по имени и может загружаться автоматически.
- [ ] Закоммиченный исполняемый `PreToolUse` хук для Edit и Write блокирует доступ к `samples/` и ничему больше.
- [ ] Закоммиченный субагент-ревьюер содержит только инструменты для чтения.
- [ ] Закоммиченный `.mcp.json` запускает chrome-devtools-mcp точной версии с выключенной статистикой, CrUX и проверками обновлений.
- [ ] Закоммиченный `.claude/settings.json` активирует плагин из каталога Anthropic.
- [ ] Закоммиченный `.claude/settings.json` запрещает чтение `.env` и разрешает точечную команду тестов.
- [ ] Закоммиченный `.claude/settings.json` включает песочницу, а файл `.claude/settings.local.json` отсутствует в git.
- [ ] Закоммиченный `CLAUDE.md` импортирует существующий файл через `@path`.
- [ ] Закоммиченное правило в `.claude/rules/` имеет параметр `paths:`, и каждое правило сопоставляется с файлами в репозитории.
- [ ] Тесты `npm test` проходят, а `git status --porcelain` ничего не выводит.

Запустите `npm run check -- i-capstone` в вашей учебной копии. Каждый непройденный пункт выведет указания по исправлению.

## What the check can't see

Автоматический скрипт не может проверить следующие умения Intermediate Exit. Отметьте их самостоятельно и честно:

- [ ] Я могу обосновать выбор между запуском задачи в основном диалоге и делегированием субагенту.
- [ ] Я могу определить, какая модель и уровень усилий требуются задаче, и выставить их на нужном уровне видимости.
- [ ] Я умею расследовать причины несрабатывания правил памяти через проверку загрузки контекста.
- [ ] Я понимаю границы изоляции песочницы: что изолируется, а что выполняется вне её контура.

Когда все пункты отмечены, вы готовы к переходу на уровень [Advanced](../advanced/README.md).

---

<sub>Sources: the [Intermediate lessons](README.md), which cite the official pages for each step.</sub>

<sub>← [Install and manage plugins](08-plugins.md) · [Intermediate index](README.md) · [Advanced index](../advanced/README.md) → · [Stuck on this capstone?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-capstone)</sub>
