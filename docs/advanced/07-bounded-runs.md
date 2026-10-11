# Put bounds on autonomous runs

<sub>**Advanced** · Lesson 7 of 7 · about 20 minutes, plus run time · Needs: [Script Claude Code with `claude -p`](04-headless.md), [Share your setup with a team](06-share-your-setup.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете ограничивать автономный запуск `claude -p` правилом наименьших привилегий, песочницей, лимитами раундов и расходов, этапом проверки человеком и предварительной оценкой стоимости. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can bound an unattended `claude -p` run with least privilege, the sandbox, turn and spend limits, a review gate and a cost estimate.</details>

## You need

- Репозиторий, для которого вы писали скрипты в [Уроке 4](04-headless.md) и настраивали общую конфигурацию в [Уроке 6](06-share-your-setup.md), с настроенной командой запуска тестов, а также копия учебного шаблона с Node.js LTS и проверками уровня Advanced ([Практика](README.md#practice)): проверка вызывается из копии. Настройте папку `.practice/` в соответствии с описанием в [Уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если еще не сделали этого.
- bash и `jq` в среде macOS, Linux или WSL 2. На Linux и WSL 2 для работы песочницы требуются утилиты `bubblewrap` и `socat` ([Set up Linux and WSL2](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2)).
- Расход токенов: умеренный (moderate). Каждый запуск останавливается по достижении установленных лимитов. Экономный вариант: в разделе Ваша очередь задайте лимиты по результатам пробного запуска среза и выполните скрипт еще для одного небольшого среза вместо всей задачи целиком.

## The idea

Процесс, за которым никто не следит в реальном времени, выполняет все, что было разрешено заранее. Поэтому разрешайте минимум и жестко ограничивайте остальное:

- **Наименьшие привилегии.** Режим `dontAsk` отклоняет любые действия, требующие интерактивного подтверждения ([dontAsk mode](https://code.claude.com/docs/en/permission-modes#allow-only-pre-approved-tools-with-dontask-mode)), поэтому разрешайте доступ только к тем папкам, которые задача должна менять, и строго к одной команде, которую она должна запускать.
- **Песочница.** Операционная система строго ограничивает области записи и сетевой доступ разрешенных команд ([Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing)). Включайте строгий режим и запрещайте старт без песочницы.
- **Лимиты.** Флаг `--max-turns` ограничивает раунды вызова инструментов агентом, а флаг `--max-budget-usd` ограничивает расчетный бюджет расходов ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)).
- **Этап проверки человеком (Review gate).** Запретите `git commit` и `git push`: запрещающие правила действуют во всех режимах ([Available modes](https://code.claude.com/docs/en/permission-modes#available-modes)). Режим `dontAsk` отклоняет вариации команд, не попавшие под шаблон (например, `git -C . push`), если они не были явно разрешены ([What a Bash rule doesn't match](https://code.claude.com/docs/en/permissions#bash-rule-limits)). Вы лично проверяете diff и коммитите только одобренные правки.
- **Оценка стоимости.** Сначала выполните один небольшой срез задачи (slice) и задайте финансовый лимит на основе его реальной стоимости.

## Worked example

Работайте в вашем репозитории. Папки `src/` и `test/`, а также команда `npm test` приведены в качестве примера: замените их на пути и команды вашего проекта во всех местах ниже.

1. Сохраните ограничения в файле `.claude/bounded-run.json`:

   ```json
   {
     "permissions": {
       "allow": ["Edit(./src/**)", "Edit(./test/**)", "Bash(npm test *)"],
       "deny": ["Agent", "Read(./.env)", "Read(./.env.*)", "Bash(git commit *)", "Bash(git push *)"]
     },
     "sandbox": {
       "enabled": true,
       "failIfUnavailable": true,
       "allowUnsandboxedCommands": false,
       "autoAllowBashIfSandboxed": false
     }
   }
   ```

   - `allow`: редактирование в двух целевых папках и запуск `npm test` с любыми аргументами. Правило `Edit` охватывает все встроенные инструменты изменения файлов, а путь с `./` отсчитывается от стартовой директории запуска ([Read and Edit](https://code.claude.com/docs/en/permissions#read-and-edit)).
   - `deny`: защита файлов `.env`; блокировка `git commit` и `git push` (где завершающий ` *` также соответствует вызову без аргументов, [Wildcard patterns](https://code.claude.com/docs/en/permissions#wildcard-patterns)); и запрет инструмента `Agent`, исключающий неконтролируемое делегирование субагентам ([Built-in subagents](https://code.claude.com/docs/en/sub-agents#built-in-subagents)), которые могли бы работать со своими режимами разрешений ([Permission modes](https://code.claude.com/docs/en/sub-agents#permission-modes)).
   - `sandbox`: активирована; завершает работу на старте, если песочницу невозможно поднять ([`sandbox.failIfUnavailable`](https://code.claude.com/docs/en/settings-reference#sandbox-failifunavailable)); никогда не повторяет запуск вне изоляции ([Strict sandbox mode](https://code.claude.com/docs/en/sandboxing#turn-off-the-retry-with-strict-sandbox-mode)); и проверяет изолированные команды по вашим правилам вместо автоматического одобрения ([`sandbox.autoAllowBashIfSandboxed`](https://code.claude.com/docs/en/settings-reference#sandbox-autoallowbashifsandboxed)).

   Инструменты редактирования файлов Claude не могут изменить этот конфигурационный файл во время работы: в режиме `dontAsk` запись в защищенную директорию `.claude/` безусловно отклоняется, и разрешающие правила не могут это отменить ([Protected paths](https://code.claude.com/docs/en/permission-modes#protected-paths)).
2. Создайте запускающий скрипт `scripts/bounded-run.sh`:

   ```bash
   #!/bin/bash
   # Runs one unattended task inside the bounds in .claude/bounded-run.json.
   set -u
   if [ $# -ne 2 ]; then
     echo 'Usage: scripts/bounded-run.sh "<task>" <result.json>' >&2
     exit 2
   fi
   before=$(git rev-parse --short HEAD)
   claude -p "$1" \
     --setting-sources project \
     --settings .claude/bounded-run.json \
     --permission-mode dontAsk \
     --model sonnet \
     --max-turns 15 \
     --max-budget-usd 2 \
     --output-format json > "$2"
   status=$?
   jq '{subtype, num_turns, total_cost_usd, denied: [.permission_denials[]? | .tool_input.command // .tool_name]}' "$2"
   echo "HEAD before: $before, after: $(git rev-parse --short HEAD)"
   git status --short
   exit "$status"
   ```

   - `--setting-sources project`: исключает пользовательские и локальные файлы настроек из сессии ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)), благодаря чему сохраненные ранее интерактивные разрешения из `.claude/settings.local.json` не попадут в автономный запуск ([Permission system](https://code.claude.com/docs/en/permissions#permission-system)).
   - `--settings`: указывает на файл ограничений, применяемый Claude Code поверх настроек самого проекта ([Change a setting for one session](https://code.claude.com/docs/en/settings#change-a-setting-for-one-session)). Разрешающие правила передаются здесь, так как запуск `-p` игнорирует разрешающие правила из `.claude/settings.json` в репозиториях без предварительного подтверждения доверия ([What runs before you trust a folder](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder)).
   - `allowUnsandboxedCommands: false`: передача через `--settings` предотвращает повтор команды вне песочницы, даже если в проекте указано `true` ([Strict sandbox mode](https://code.claude.com/docs/en/sandboxing#turn-off-the-retry-with-strict-sandbox-mode)). Начиная с v2.1.285 Claude Code также игнорирует настройки репозитория, ослабляющие песочницу ([Repository settings under an admin-required sandbox](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)).
   - `--model sonnet`: Sonnet отлично справляется с большинством задач программирования при заметно меньшей стоимости по сравнению с Opus ([Choose the right model](https://code.claude.com/docs/en/costs#choose-the-right-model)).
   - `--max-turns` и `--max-budget-usd`: настраиваемые ограничения, подбираемые под объем задачи.
3. Сделайте скрипт исполняемым и закоммитьте оба файла, чтобы вся команда работала в одинаковых границах безопасности:

   ```bash
   chmod +x scripts/bounded-run.sh
   git add scripts/bounded-run.sh .claude/bounded-run.json
   git commit -m "Add a bounded runner for unattended tasks"
   ```

   Если git сообщает об игнорировании файла, команда `git check-ignore -v <path>` покажет правило; добавьте файл принудительно через `git add -f`.
4. Проверьте, как лимит останавливает процесс. Запустите те же ограничения с намеренно крошечным бюджетом:

   ```bash
   claude -p "Run npm test and summarize the result" \
     --setting-sources project --settings .claude/bounded-run.json \
     --permission-mode dontAsk --model sonnet \
     --max-budget-usd 0.01 --output-format json > .practice/a-7-capped.json
   echo "exit $?"
   jq '{subtype, total_cost_usd}' .practice/a-7-capped.json
   ```

   Команда вернет ненулевой код завершения, затем `"subtype": "error_max_budget_usd"`: процесс прервался по лимиту, сохранив запись в JSON ([Handle the result](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result)). Поле `total_cost_usd` может немного превышать заданный лимит, так как оно включает ответ модели, на котором был зафиксирован перерасход ([Track costs on failed conversations](https://code.claude.com/docs/en/agent-sdk/cost-tracking#track-costs-on-failed-conversations)).
5. Оцените стоимость на небольшом срезе (slice) задачи:

   ```bash
   scripts/bounded-run.sh "Use your Edit tool to add a test for one untested function in src/<file>. Then run npm test as a command on its own, and then commit the test." .practice/a-7-slice.json
   ```

   Сводка покажет `"subtype": "success"`, количество раундов, расчетную стоимость `total_cost_usd`, а также список заблокированных вызовов в `denied` (например, попытку вызова `git commit`). Значение HEAD остается неизменным до и после выполнения, а `git status` показывает незакоммиченный тест. Оценка `total_cost_usd` рассчитывается на стороне клиента и является приблизительной ([Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking#estimates-not-billing)); на тарифах Claude команда `/usage` отображает лимиты тарифного плана ([Commands](https://code.claude.com/docs/en/commands#all-commands)). Чтобы задать границы для всей задачи, пропорционально масштабируйте стоимость среза с разумным запасом.
6. Выступите в роли контролирующего ревьюера. Внимательно просмотрите все изменения через `git status` и `git diff`, включая сам файл ограничений, самостоятельно выполните `npm test` и закоммитьте только проверенный результат. Для дополнительной проверки запустите интерактивную сессию `claude` и поручите субагенту в чистом контексте выполнить критический анализ изменений на наличие пропущенных краевых случаев ([Add an adversarial review step](https://code.claude.com/docs/en/best-practices#add-an-adversarial-review-step)).

## Your turn

Ограничьте реальную задачу в вашем репозитории объемом больше одного среза: например, написание тестов для всех непокрытых функций в директории.

Критерии приемки:

- Скрипт `scripts/bounded-run.sh "<task>" <result.json>` запускает задачу: один вызов `claude -p` в режиме `dontAsk`, с выводом JSON по указанному пути и передачей файла ограничений через параметр `--settings`.
- Скрипт вызывает `claude` напрямую по имени команды (не по абсолютному пути и не через `npx`) и не изменяет переменную `PATH`: это позволяет скрипту проверки подставить тестовую заглушку.
- Каждое разрешающее правило указывает строго одну папку или одну конкретную команду: никаких общих `Bash` или `Edit`, и никаких правил на весь проект или программу целиком вроде `Bash(npm *)`. Закоммиченный файл `.claude/settings.json` также учитывается: в доверенной папке его правила объединяются с правилами файла `--settings` ([Lists merge instead of overriding](https://code.claude.com/docs/en/settings#lists-merge-instead-of-overriding)). Это же относится к `allowed-tools` в навыках репозитория при их вызове ([Pre-approve tools for a skill](https://code.claude.com/docs/en/skills#pre-approve-tools-for-a-skill)).
- Запрещающие правила блокируют `git commit` и `git push`, каждое с обязательным пробелом и звездочкой на конце ` *`: правило `Bash(git push)` без звездочки сопоставляется только с вызовом без аргументов.
- Песочница включена в строгом режиме, не использует auto-allow, пропускает изолированные команды через ваши правила, и запуск завершается ошибкой, если песочница недоступна.
- Заданы лимит раундов и лимит бюджета выше стоимости среза с учетом масштаба задачи.
- Скрипт имеет права на исполнение, а сам скрипт и файл ограничений закоммичены в git.
- Вы выполняете запуск, после чего самостоятельно проверяете diff и запускаете тесты перед созданием коммита.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Скрипт `scripts/bounded-run.sh` закоммичен и имеет права на исполнение, принимает текст задачи и путь для результата (`scripts/bounded-run.sh "<task>" <result.json>`), и запускает `claude -p` в режиме `dontAsk` с выводом в JSON и закоммиченным файлом `--settings`.
- [ ] В скрипте установлены лимит раундов и лимит бюджета.
- [ ] Каждое разрешающее правило указывает строго одну команду или папку, а запрещающие правила блокируют `git commit` и `git push`.
- [ ] Файл настроек включает песочницу в строгом режиме без auto-allow и запрещает старт без песочницы.
- [ ] Файл `.practice/a-7-slice.json` содержит пробный запуск среза с затратами ниже лимита бюджета.
- [ ] Файл `.practice/a-7-capped.json` содержит запуск, остановленный по достижении лимита.
- [ ] Самопроверка (не тестируется): вы лично просмотрели diff и запустили тесты перед коммитом изменений.

Запустите команду `npm run check -- a-7 --dir <your repo>` из вашей копии учебного шаблона или `npm run check -- a-7` в самом шаблоне. Проверка запускает скрипт с заглушкой вместо реального Claude, поэтому токены не тратятся.

Если первый пункт завершается ошибкой, убедитесь, что права на исполнение установлены (`git ls-files -s scripts/bounded-run.sh` начинается с `100755`), вызов `claude` идет напрямую без путей и `npx`, и `PATH` не переопределяется. Если не проходит разрешающее правило, сузьте его до конкретной директории или команды. Если проверка песочницы указывает на ключ, добавьте его в файл настроек: без `failIfUnavailable` сессия при недоступности песочницы продолжит работу без изоляции ([`sandbox.enabled`](https://code.claude.com/docs/en/settings-reference#sandbox-enabled)). Если не проходит файл среза, убедитесь, что скрипт направляет в файл результата только чистый JSON вывода `claude` (stdout), и повторите шаг 5; если срез завершился ошибкой `error_max_budget_usd`, ваш лимит бюджета ниже стоимости даже одного среза, увеличьте его.

## Watch out

- Ваши границы не охватывают все, что загружается в сессии. Хуки и локальные серверы MCP исполняются вне песочницы с вашими полными правами доступа ([What runs outside the sandbox](https://code.claude.com/docs/en/sandboxing#what-runs-outside-the-sandbox)). Как предупреждал [Урок 4](04-headless.md#watch-out), запуск `-p` подключает хуки и серверы из `.mcp.json` без подтверждения, а также применяет `allowed-tools` навыков проекта ([What runs before you trust a folder](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder)). Внимательно проверяйте хуки, серверы и навыки перед запуском, либо добавьте `"disableAllHooks": true` в файл ограничений для полного отключения хуков.
- Запрещающее правило сопоставляется с командой, которую пишет Claude, а не с тем, что делает запущенная программа ([What a Bash rule doesn't match](https://code.claude.com/docs/en/permissions#bash-rule-limits)). Разрешенная команда `npm test` исполняет тестовые файлы, которые Claude может редактировать. В песочнице этот код может писать в любые места рабочего каталога, кроме защищенных путей (внутри `.git` защищены только `hooks` и `config`, [Protected paths](https://code.claude.com/docs/en/sandboxing#protected-paths)). Однако отправка `git push` из изолированной команды не сможет связаться с remote: в режиме `dontAsk` песочница отклоняет подключения к неразрешенным хостам ([Hosts outside your allowed domains](https://code.claude.com/docs/en/sandboxing#hosts-outside-your-allowed-domains), [Turn off permission prompts in unattended runs](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs)). Именно поэтому скрипт сравнивает HEAD до и после работы, а ваше ревью должно покрывать каждый измененный файл.
- Динамический воркфлоу запускается через собственный инструмент `Workflow`. В автономном запуске он стартует только при явном одобрении правилом allow или хуком `PreToolUse`: правило `Workflow` одобряет любой воркфлоу, а `Workflow(<name>)` разрешает конкретный сохраненный воркфлоу ([Approve the plan before it runs](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs)). Агенты воркфлоу следуют общим правилам разрешений, а их расходы учитываются в `--max-budget-usd`. При их использовании добавьте `"workflowSizeGuideline": "small"` в файл ограничений ([`workflowSizeGuideline`](https://code.claude.com/docs/en/settings-reference#workflowsizeguideline)).

## Go further

- [Выбор среды песочницы](https://code.claude.com/docs/en/sandbox-environments): использование контейнеров и виртуальных машин для запусков, выходящих за рамки возможностей песочницы Bash.
- [Инженерия изоляции Claude во всех продуктах](https://www.anthropic.com/engineering/how-we-contain-claude): публикация команды Anthropic об архитектуре ограничения потенциального ущерба от агентов в Claude Code и других сервисах.
- [Предотвращение уязвимостей в процессе написания кода](https://code.claude.com/docs/en/security-guidance): плагин security-guidance, поручающий Claude анализировать собственные правки на наличие уязвимостей прямо во время работы.

---

<sub>Sources: [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Extend Claude with skills](https://code.claude.com/docs/en/skills) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Settings files and precedence](https://code.claude.com/docs/en/settings) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [Run Claude Code programmatically](https://code.claude.com/docs/en/headless) · [How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop) · [Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) · [Manage costs effectively](https://code.claude.com/docs/en/costs) · [Commands](https://code.claude.com/docs/en/commands) · [Best practices](https://code.claude.com/docs/en/best-practices) · [Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows)</sub>

<sub>← [Совместное использование настроек в команде](06-share-your-setup.md) · [Advanced index](README.md) · [Практикум: перенос кодовой базы с динамическим воркфлоу](capstone.md) → · Topic: [Разрешения и безопасность](../topics/permissions-and-safety.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-7)</sub>
