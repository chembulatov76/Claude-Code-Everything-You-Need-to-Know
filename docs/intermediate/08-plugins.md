# Install and manage plugins

<sub>**Intermediate** · Lesson 8 of 8 · about 20 minutes · Needs: [Turn a repeated workflow into a skill](01-first-skill.md), [Enforce a rule with a hook](05-hooks.md), [Delegate to a custom subagent](06-subagents.md), [Connect a tool with MCP](07-mcp.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого урока вы сможете устанавливать плагины из каталогов Anthropic в подходящей области видимости, проверять состав добавляемых ими возможностей, а также отключать, обновлять и удалять плагины. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can install a plugin from one of Anthropic's marketplaces at the scope that fits, see what it adds, and turn it off, update it or remove it.</details>

## You need

- Репозиторий из [урока 4](04-permissions-and-sandbox.md) с файлом `.claude/settings.json`, и учебная копия шаблона с Node.js LTS (проверка запускается из неё).
- Расход токенов: минимальный (light).

## The idea

[Плагин (plugin)](https://code.claude.com/docs/en/plugins/overview) объединяет компоненты предыдущих уроков (навыки, субагенты, хуки и MCP-серверы), позволяя устанавливать их единым пакетом из каталогов-маркетплейсов. Anthropic публикует три основных каталога ([Anthropic's marketplaces](https://code.claude.com/docs/en/plugins/anthropic-marketplaces)): <details style="display:inline"><summary><code>[en]</code></summary>A plugin packages the pieces of the last lessons, skills, subagents, hooks and MCP servers, so they install as one unit from a marketplace.</details>

- `claude-plugins-official` - официальный каталог, автоматически подключаемый при первом старте интерактивной сессии;
- `claude-community` - плагины сторонних авторов, прошедшие регистрацию в каталоге Anthropic;
- `claude-code-plugins` - демонстрационный набор плагинов.

Установка выполняется в формате `<plugin>@<marketplace>` с указанием области действия (scope), определяющей файл настроек ([Choose an install scope](https://code.claude.com/docs/en/plugins/install#choose-an-install-scope)):

| Scope | Кому доступен | Запись `enabledPlugins` в |
|---|---|---|
| User | Вам во всех проектах | `~/.claude/settings.json` |
| Project | Всем участникам репозитория | `.claude/settings.json`, в git |
| Local | Вам лично в текущем репозитории | `.claude/settings.local.json` |

Коммит записи проектного уровня активирует плагин для ваших коллег, однако сам плагин может ещё не быть скачан на их машинах, поэтому каждому достаточно один раз выполнить `claude plugin install <plugin>@<marketplace> --scope project`. Включённый плагин участвует в каждой сессии: описания его навыков находятся в контексте, а его хуки и серверы MCP выполняются от имени вашего пользователя.

## Worked example

1. В сессии вашего репозитория откройте плагин `commit-commands` из официального каталога (Anthropic):

   ```text
   /plugin install commit-commands@claude-plugins-official
   ```

   На этом шаге ничего не устанавливается. Панель показывает описание плагина и в блоке **Will install** перечисляет добавляемые команды. Всегда изучайте этот список перед выбором.
2. Выберите вариант **Install for you, in this repo only (local scope)**. Итог установки завершится строкой `Plugin is now active.`
3. Введите `/commit-commands:`. Команды плагина отобразятся под его именем (например, `/commit-commands:commit`).
4. Управляйте плагинами из терминала ([Manage plugins from your shell](https://code.claude.com/docs/en/plugins/install#manage-plugins-from-your-shell)):

   ```bash
   claude plugin list
   claude plugin disable commit-commands@claude-plugins-official
   claude plugin uninstall commit-commands@claude-plugins-official --scope local
   ```

   Команда `claude plugin list` выводит версию плагина, область действия (`Scope: local`) и текущий статус. Команда `disable` выключает плагин, сохраняя установленные файлы; команда `uninstall` удаляет его полностью.

## Your turn

Включите плагин `claude-code-setup` (Anthropic) для всех разработчиков вашего репозитория и установите его по сценарию нового коллеги.

1. Добавьте следующий блок в `.claude/settings.json` рядом с ключами из предыдущих уроков, заменив маркер `TODO` полным идентификатором плагина `<plugin>@<marketplace>`:

   ```json
   "enabledPlugins": {
     "TODO: claude-code-setup at the official marketplace": true
   }
   ```

2. Закоммитьте обновлённый `.claude/settings.json`.
3. Установите плагин так, как это сделал бы коллега после получения вашего коммита:

   ```bash
   claude plugin install claude-code-setup@claude-plugins-official --scope project
   ```

   Команда выведет: `Successfully installed plugin: claude-code-setup@claude-plugins-official (scope: project)`. Выполните `git status`: если установка обновила `.claude/settings.json`, закоммитьте эти изменения.
4. Запустите `claude` и отправьте запрос: `Recommend Claude Code automations for this project.` Claude задействует навык плагина только для чтения ([его README](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-code-setup/README.md)) и предложит подходящие хуки, навыки, субагентов и MCP-серверы.
5. Сохраните список плагинов для проверки в корне репозитория: `claude plugin list --json > .practice/i-8-plugins.json`.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченные настройки включают плагин для проекта: `git show HEAD:.claude/settings.json | jq .enabledPlugins` содержит `"claude-code-setup@claude-plugins-official": true`.
- [ ] Плагин установлен в project scope и активирован: `jq '.[] | select(.id == "claude-code-setup@claude-plugins-official") | {scope, enabled, projectEnabled}' .practice/i-8-plugins.json` возвращает `"project"`, `true` и `true`.
- [ ] Self-check (not tested): вы можете объяснить, что сохраняет `disable`, но удаляет `uninstall`, и почему коллеге может потребоваться выполнить `claude plugin install` после стягивания настроек.

Запустите `npm run check -- i-8 --dir <your repo>` из учебной копии, либо `npm run check -- i-8` в самой копии.

Если первый пункт не проходит, проверьте название после символа `@`: это имя каталога `claude-plugins-official`, а не URL репозитория. Если не проходит второй пункт, выполните команду установки в самом репозитории и повторно экспортируйте JSON-список.

## Watch out

- Хуки и MCP-серверы плагина выполняются с полными правами вашего пользователя ОС, а сторонние каталоги не модерируются компанией Anthropic. Всегда изучайте список **Will install** перед подтверждением ([Plugin security and trust](https://code.claude.com/docs/en/plugins/security)).
- Официальный каталог содержит также плагин `chrome-devtools-mcp` (Vendor: Google), зафиксированный на коммите репозитория сервера ([его каталог](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json)). На 2026-10-06 [его манифест](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/1cec9cd1a3bbf1895c98fa4b4e0e2da5a36e4075/.claude-plugin/plugin.json) запускал сервер из [урока 7](07-mcp.md) более старой версии и без защитных флагов, из-за чего телеметрия остаётся включённой по умолчанию ([upstream README](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/chrome-devtools-mcp-v1.10.1/README.md)). Предпочитайте конфигурацию `.mcp.json` из урока 7.
- Плагины из `claude-plugins-official` обновляются автоматически; запущенная сессия использует загруженную версию до команды `/reload-plugins` или перезапуска. Плагины из каталога сообщества не обновляются автоматически, если вы явно не включите автообновление ([Keep plugins updated](https://code.claude.com/docs/en/plugins/install#keep-plugins-updated)).

Third-party: it can run code on your machine or give Claude new tools. Review it before installing; this guide does not audit it.

Listings checked against the listing bar on 2026-10-06.

## Go further

- [Plugins overview](https://code.claude.com/docs/en/plugins/overview): архитектура и устройство плагинов Claude Code.
- [Anthropic's marketplaces](https://code.claude.com/docs/en/plugins/anthropic-marketplaces): структура официальных каталогов и правила публикации.
- [Тестирование и публикация плагина](../advanced/electives/publish-a-plugin.md): факультатив уровня Advanced по разработке и распространению собственного плагина.

---

<sub>Sources: [Plugins overview](https://code.claude.com/docs/en/plugins/overview) · [Install and manage plugins](https://code.claude.com/docs/en/plugins/install) · [Anthropic's marketplaces](https://code.claude.com/docs/en/plugins/anthropic-marketplaces) · [Plugin security and trust](https://code.claude.com/docs/en/plugins/security) · [Chrome DevTools MCP README](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/chrome-devtools-mcp-v1.10.1/README.md) · [claude-plugins-official manifest](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) · [chrome-devtools plugin manifest](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/1cec9cd1a3bbf1895c98fa4b4e0e2da5a36e4075/.claude-plugin/plugin.json)</sub>

<sub>← [Connect a tool with MCP](07-mcp.md) · [Intermediate index](README.md) · [Intermediate capstone](capstone.md) → · Topic: [Plugins](../topics/plugins.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-8)</sub>
