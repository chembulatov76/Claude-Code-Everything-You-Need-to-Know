# Permissions, settings scopes and the sandbox

<sub>**Intermediate** · Lesson 4 of 8 · about 20 minutes · Needs: [Organize project memory and see what loaded](02-organize-memory.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого урока вы сможете размещать правила разрешений в конфигурационных файлах соответствующей области видимости, проверять действующие ограничения и включать песочницу (sandbox), чтобы выполняемые через Claude команды оболочки не могли записывать данные за пределами каталога проекта, временной папки и явно добавленных директорий. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can put a permission rule in the settings file whose scope fits it, see which rules are in force, and turn on the sandbox so the shell commands Claude runs can't write outside your project, a temporary folder and any folders you've added.</details>

## You need

- Ваша учебная копия репозитория для выполнения разобранного примера с Node.js LTS (проверка запускается из неё). Один из ваших рабочих репозиториев для раздела Your turn (или копия шаблона для обеих частей).
- macOS, Linux или WSL 2. В Linux и WSL 2 песочнице требуются пакеты `bubblewrap` и `socat` ([Set up Linux and WSL2](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2)); в GitHub Codespaces учебного шаблона оба пакета уже установлены.
- Расход токенов: минимальный (light).

## The idea

Claude Code считывает параметры из четырёх файлов конфигурации, и выбранный файл определяет, на кого распространяется правило ([Settings](https://code.claude.com/docs/en/settings#settings-files-and-who-they-affect)): <details style="display:inline"><summary><code>[en]</code></summary>Claude Code reads settings from four files, and the file a rule goes in decides who it affects</details>

| Файл | На кого влияет | Что размещать здесь |
|---|---|---|
| `~/.claude/settings.json` | Вы лично во всех проектах | Персональные предпочтения |
| `.claude/settings.json`, в git | Все участники проекта | Правила команды: файлы, которые нельзя читать, общие команды |
| `.claude/settings.local.json`, вне git | Вы лично в текущем проекте | Локальные исключения. Ответ "Yes, and don't ask again" сохраняется сюда |
| Managed settings | Все пользователи организации | Корпоративные политики, которые локальные настройки не могут переопределить |

Если один и тот же ключ задан на нескольких уровнях, приоритет имеет более высокий уровень: Managed, затем аргументы командной строки, затем local, затем shared project, затем user ([Settings precedence](https://code.claude.com/docs/en/settings#settings-precedence)). Списки правил (например, `permissions.allow`) объединяются со всех уровней. Внутри правил сначала проверяются запреты (deny), затем запросы (ask), затем разрешения (allow); первое совпадение определяет исход ([Configure permissions](https://code.claude.com/docs/en/permissions#manage-permissions)).

Правила доступа управляют вызовами инструментов Claude. [Песочница (sandbox)](https://code.claude.com/docs/en/sandboxing) создаёт изолированный контур на уровне операционной системы вокруг запускаемых команд оболочки: по умолчанию запись разрешена только внутри проекта, в выделенной временной папке пользователя и в директориях, добавленных через `--add-dir` или `/add-dir`. Доступ к сети возможен только через прокси, проверяющий адреса по разрешённому списку. По умолчанию песочница выключена. В режиме auto-allow команды внутри песочницы выполняются без запроса подтверждений.

## Worked example

В учебной копии:

1. Запустите `claude` и выполните `/status`. Строка `Setting sources` перечислит файлы настроек, загруженные текущей сессией.
2. Сохраните командные правила в файле `.claude/settings.json`. Это стандартные правила документации для защиты файлов `.env` ([Exclude sensitive files](https://code.claude.com/docs/en/settings-reference#exclude-sensitive-files)), дополненные командой тестов:

   ```json
   {
     "permissions": {
       "allow": ["Bash(npm test)"],
       "deny": ["Read(./.env)", "Read(./.env.*)"]
     }
   }
   ```

   Claude Code автоматически перечитывает файл настроек при его изменении ([When edits take effect](https://code.claude.com/docs/en/settings#when-edits-take-effect)). Выполните `/permissions`: в блоке allow появится `Bash(npm test)`, а в deny - оба правила `Read`.
3. Попросите Claude: `Run npm test`. Тест выполнится без запроса разрешений: его покрывает разрешающее правило.
4. Включите песочницу. Выполните `/sandbox` и на вкладке Mode выберите вариант auto-allow. Claude Code сохранит эту настройку в `.claude/settings.local.json`, ваш персональный файл ([Get started](https://code.claude.com/docs/en/sandboxing#get-started)).
5. Попросите Claude выполнить запись за пределами проекта:

   ```text
   Run this exact command and show me its output: touch ~/sandbox-probe
   ```

   Команда запустится без запроса подтверждения и завершится с ошибкой: `Operation not permitted` на macOS или `Read-only file system` на Linux и WSL 2 ([Confirm commands run inside the sandbox](https://code.claude.com/docs/en/sandboxing#confirm-commands-run-inside-the-sandbox)). Если Claude предложит повторить запуск вне песочницы, ответьте отказом.

## Your turn

Настройте такое же разделение конфигурации в вашем собственном репозитории.

1. Сохраните шаблон файла как `.claude/settings.json` и замените каждый маркер `TODO` правилом. Разрешите только одну точную команду запуска ваших тестов (например, `Bash(npm test)` или `Bash(pytest)`):

   ```json
   {
     "permissions": {
       "allow": ["TODO: the command that runs your tests"],
       "deny": ["TODO: reading .env", "TODO: reading .env.local and the other .env.* files"]
     }
   }
   ```

2. Закоммитьте файл в git: это общекомандные правила для всех разработчиков репозитория.
3. В сессии репозитория включите песочницу командой `/sandbox` в режиме auto-allow и повторите шаг 5 из разобранного примера.
4. Выполните `git status`. Файл `.claude/settings.local.json` отсутствует в списке: Claude Code автоматически скрывает его от git при создании ([Keep personal settings out of a repository](https://code.claude.com/docs/en/settings#keep-personal-settings-out-of-a-repository)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный `.claude/settings.json` запрещает чтение `.env` и `.env.*` и разрешает одну команду запуска тестов: `git show HEAD:.claude/settings.json | jq .permissions` показывает эти правила.
- [ ] Песочница включена, а персональный файл не попадает в git: `jq .sandbox .claude/settings.local.json` выводит `"enabled": true`, а `git ls-files .claude/settings.local.json` ничего не выводит.
- [ ] Self-check (not tested): команда Claude `touch ~/sandbox-probe` завершилась с ошибкой доступа без предварительного запроса подтверждения.

Запустите `npm run check -- i-4 --dir <your repo>` из учебной копии, либо `npm run check -- i-4` в самой копии.

Если первый пункт не проходит из-за слишком широкого правила, замените его точной командой: маска `Bash(npm *)` позволила бы Claude выполнять любые произвольные команды npm без запроса разрешений. Если `/sandbox` показывает только вкладку Dependencies, установите пакеты `bubblewrap` и `socat`, перезапустите Claude Code и снова выполните `/sandbox`.

## Watch out

- Песочница изолирует только команды оболочки (shell). Собственные инструменты Claude для работы с файлами и вебом выполняются вне песочницы и руководствуются правилами permissions; хуки и локальные MCP-серверы выполняются с полными правами текущего пользователя ([What runs outside the sandbox](https://code.claude.com/docs/en/sandboxing#what-runs-outside-the-sandbox)). Если песочница не может стартовать, Claude Code запускает команды без изоляции, если параметр `sandbox.failIfUnavailable` не установлен в `true`.
- Сам по себе запрет `Read` блокирует инструменты Claude и распознаваемые файловые команды оболочки (`cat`), но не защищает от запускаемых скриптов, читающих файлы напрямую ([Read and Edit](https://code.claude.com/docs/en/permissions#read-and-edit)). При включённой песочнице Claude Code передаёт пути запретов `Read` в параметр песочницы [`filesystem.denyRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-denyread), делая эти файлы недоступными для любых изолированных процессов.
- В среде Codespaces или других контейнерах, если команды песочницы падают с ошибкой `Can't mount proc on /newroot/proc`, обратитесь к разделу [Bubblewrap fails to start inside a container](https://code.claude.com/docs/en/sandboxing#bubblewrap-fails-to-start-inside-a-container).

## Go further

- [How we built Claude Code auto mode](https://www.anthropic.com/engineering/claude-code-auto-mode): инженерный обзор Anthropic о классификаторе auto mode, его надёжности и граничных случаях.
- [Making Claude Code more secure and autonomous with sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing): принципы изоляции файловой системы и сетевого контура.
- [Security](https://code.claude.com/docs/en/security): целостная модель безопасности Claude Code.

---

<sub>Sources: [Settings](https://code.claude.com/docs/en/settings) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [Settings reference](https://code.claude.com/docs/en/settings-reference) · [How we built Claude Code auto mode](https://www.anthropic.com/engineering/claude-code-auto-mode)</sub>

<sub>← [Pick the model and effort](03-model-and-effort.md) · [Intermediate index](README.md) · [Enforce a rule with a hook](05-hooks.md) → · Topic: [Permissions and safety](../topics/permissions-and-safety.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-4)</sub>
