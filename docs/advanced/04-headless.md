# Script Claude Code with `claude -p`

<sub>**Advanced** · Lesson 4 of 7 · about 20 minutes · Needs: [Run and save a dynamic workflow](03-dynamic-workflows.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете писать скрипты для запуска Claude Code в полностью автономном режиме без участия человека за клавиатурой, считывать результат в формате JSON и код завершения, а также жестко ограничивать доступные инструменты и число раундов диалога под требования задачи. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can write a script that runs Claude Code with no one at the keyboard, reads its JSON result and exit code, and limits it to the tools and turns the job needs.</details>

## You need

- Один из ваших репозиториев с настроенной командой запуска тестов, а также копия учебного шаблона с Node.js LTS: скрипт проверки запускается из копии. Чтобы практиковаться непосредственно в копии шаблона, используйте ее для обеих целей. Настройте папку `.practice/` в соответствии с описанием в [Уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если еще не сделали этого.
- Оболочка bash и утилита `jq` в среде macOS, Linux или WSL 2.
- Расход токенов: минимальный (light). Каждый запуск `claude -p` представляет собой одну короткую сессию.

## The idea

Команда `claude -p "<prompt>"` выполняет один запрос без запуска интерактивного интерфейса, выводит ответ в терминал и завершается с кодом 0 (или ненулевым кодом при сбое) ([Run Claude Code programmatically](https://code.claude.com/docs/en/headless)). Она читает стандартный поток ввода stdin, поэтому скрипты могут передавать вывод diff прямо через конвейер.

С флагом [`--output-format json`](https://code.claude.com/docs/en/headless#get-structured-output) команда возвращает один JSON-объект с полями `result`, `is_error`, `num_turns`, `session_id` и `permission_denials` ([`SDKResultMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage)). Поле `result` содержит текст ответа или сообщение об ошибке при внутреннем сбое ([Basic usage](https://code.claude.com/docs/en/headless#basic-usage)). Поле `subtype` принимает значение `success` либо причину досрочной остановки (например, `error_max_turns`; в таком случае поле `result` отсутствует) ([Handle the result](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result)). Всегда проверяйте код завершения процесса и флаг `is_error` перед использованием содержимого `result`.

Поскольку подтвердить запрос разрешений некому, всегда явно указывайте режим: сессия без указания режима может запуститься в режиме auto ([Auto-approve tools](https://code.claude.com/docs/en/headless#auto-approve-tools)). Три флага устанавливают границы: `--permission-mode dontAsk` отклоняет любые вызовы, требующие подтверждения, `--allowedTools` явно указывает разрешенные инструменты, а `--max-turns` ограничивает максимальное число раундов вызова инструментов ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)).

## Worked example

В вашем репозитории:

1. Задайте один вопрос и сохраните структурированный ответ в JSON:

   ```bash
   claude -p "In one sentence, what does this repository do?" --output-format json --permission-mode dontAsk > .practice/a-4-first.json
   echo "exit $?"
   jq '{is_error, subtype, num_turns, session_id}' .practice/a-4-first.json
   jq -r .result .practice/a-4-first.json
   ```

   Команда выведет `exit 0`, затем `"is_error": false` и `"subtype": "success"` с количеством раундов и идентификатором сессии, а затем одно предложение с описанием репозитория. Чтение файлов в рабочей директории не требует подтверждения, поэтому режим `dontAsk` позволил Claude прочитать нужные файлы ([dontAsk mode](https://code.claude.com/docs/en/permission-modes#allow-only-pre-approved-tools-with-dontask-mode)).
2. Запросите действие, требующее подтверждения. Флаг `--setting-sources user` исключает файлы настроек проекта из сессии ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)), поэтому ни разрешающие правила, ни режим auto-allow песочницы из [Урока 4 уровня Intermediate](../intermediate/04-permissions-and-sandbox.md) не одобрят запуск тестов ([Auto-allow mode](https://code.claude.com/docs/en/sandboxing#auto-allow-mode)). Это также исключает правила запрета проекта (например, защиту `.env`), поэтому шаги со 2 по 4 передают их заново через `--disallowedTools` ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)):

   ```bash
   claude -p "Run the test suite and say how many tests pass" --output-format json --permission-mode dontAsk --setting-sources user --disallowedTools "Read(./.env)" "Read(./.env.*)" > .practice/a-4-denied.json
   jq -c '.permission_denials[] | {tool_name, command: .tool_input.command}' .practice/a-4-denied.json
   jq -r .result .practice/a-4-denied.json
   ```

   Каждая строка лога представляет собой вызов инструмента, отклоненный режимом `dontAsk` ([`SDKPermissionDenial`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkpermissiondenial)), например `{"tool_name":"Bash","command":"npm test 2>&1 | tail -20"}`. Результат сообщит, что Claude не смог запустить тесты.
3. Разрешите команду из шага 2 (в данном случае `npm test`) и ограничьте число раундов:

   ```bash
   claude -p "Run the test suite and say how many tests pass" --output-format json --permission-mode dontAsk --setting-sources user --disallowedTools "Read(./.env)" "Read(./.env.*)" --allowedTools "Bash(npm test *)" --max-turns 3 | jq '{subtype, num_turns, denied: (.permission_denials | length)}'
   ```

   Вывод покажет `"subtype": "success"` и `"denied": 0`: на этот раз Claude запустил тесты. Пробел перед `*` имеет значение, при этом ` *` на конце также соответствует вызову команды без параметров ([Wildcard patterns](https://code.claude.com/docs/en/permissions#wildcard-patterns)). Правило также покрывает конвейеры вида `npm test 2>&1 | tail -20`: Claude Code проверяет каждую часть конвейера независимо, а `tail` входит в список команд только для чтения, работающих без запроса разрешений ([Compound commands](https://code.claude.com/docs/en/permissions#compound-commands), [Read-only commands](https://code.claude.com/docs/en/permissions#read-only-commands)).
4. Достигните установленного лимита раундов. Задача требует последовательного выполнения нескольких действий, а флаг `--max-turns 1` разрешает только один раунд вызова инструментов:

   ```bash
   claude -p "Read the file that defines this project's test command, then run that command, then summarize any failures" --output-format json --permission-mode dontAsk --setting-sources user --disallowedTools "Read(./.env)" "Read(./.env.*)" --allowedTools "Read,Bash(npm test *)" --max-turns 1 > .practice/a-4-capped.json
   echo "exit $?"
   jq '{subtype, has_result: has("result")}' .practice/a-4-capped.json
   ```

   Команда вернет ненулевой код завершения, затем `"subtype": "error_max_turns"` и `"has_result": false`: сессия, остановленная по достижении лимита, не содержит поля `result`. Параметр `--max-turns` считает только раунды с вызовом инструментов ([Turns and messages](https://code.claude.com/docs/en/agent-sdk/agent-loop#turns-and-messages)).

## Your turn

Напишите скрипт автоматического ревью, который можно запускать перед каждым коммитом: `scripts/review-staged.sh` передает подготовленные к коммиту изменения в Claude, выводит рецензию и сохраняет JSON-ответ. Этот скрипт повторно используется в выпускном задании уровня Advanced.

Критерии приемки:

- Скрипт bash по пути `scripts/review-staged.sh`, закоммиченный и с правами на исполнение (`chmod +x`).
- Он передает через конвейер вывод команды `git diff --cached` (подготовленные изменения, [git diff](https://git-scm.com/docs/git-diff)) в один вызов `claude -p`. Claude не требуется инструмент Bash для чтения diff, так как данные поступают через stdin ([Add Claude to a build script](https://code.claude.com/docs/en/headless#add-claude-to-a-build-script)).
- Промпт в вызове передается сразу после аргумента `-p`, как в официальных примерах [Pipe data through Claude](https://code.claude.com/docs/en/headless#pipe-data-through-claude) и [Add Claude to a build script](https://code.claude.com/docs/en/headless#add-claude-to-a-build-script), а также передаются флаги `--output-format json`, `--permission-mode dontAsk`, `--allowedTools` с перечислением только `Read`, `Grep` и `Glob`, и флаг `--max-turns` со значением от 1 до 10. В macOS, Linux и WSL инструменты Glob и Grep по умолчанию скрыты, а указание любого из них в `--allowedTools` делает доступными оба ([Glob tool behavior](https://code.claude.com/docs/en/tools-reference#glob-tool-behavior)).
- Скрипт вызывает `claude` напрямую по имени команды (не по абсолютному пути и не через `npx`) и не модифицирует переменную `PATH`: скрипт проверки подставляет тестовую заглушку через стандартное имя.
- Скрипт сохраняет результат JSON по пути из своего первого аргумента либо в `.practice/a-4-result.json`, если аргумент не передан, предварительно создавая папку при необходимости.
- Он выводит текст поля `result` и завершается с ненулевым кодом при сбое `claude` или если флаг `is_error` равен `true`.
- Если в git нет подготовленных изменений, скрипт сообщает об этом и завершается с ненулевым кодом, не вызывая Claude. Команда `git diff --cached --quiet` завершается с кодом 0, когда изменений нет ([git diff](https://git-scm.com/docs/git-diff)).

Закоммитьте скрипт. Затем подготовьте небольшое изменение в git (например, правку в одну строку) и вызовите `scripts/review-staged.sh` один раз: он создаст файл `.practice/a-4-result.json`. Ознакомьтесь с рецензией перед коммитом изменения.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Скрипт `scripts/review-staged.sh` закоммичен и имеет права на исполнение.
- [ ] Вызов `claude -p` содержит промпт сразу после `-p`, запрашивает вывод в формате JSON, работает в режиме `dontAsk`, предварительно разрешает только `Read`, `Grep` и `Glob`, и ограничивает количество раундов числом 10 или меньше.
- [ ] Скрипт передает подготовленный diff через stdin, сохраняет JSON по указанному пути, выводит результат, завершается с ненулевым кодом при ошибке и не вызывает Claude при отсутствии подготовленных изменений.
- [ ] Файл `.practice/a-4-result.json` содержит успешно выполненный запуск в рамках лимита раундов.
- [ ] Самопроверка (не тестируется): вы видели остановку сессии по достижении лимита `--max-turns` на шаге 4 пошагового примера.

Запустите команду `npm run check -- a-4 --dir <your repo>` из копии учебного шаблона или `npm run check -- a-4` внутри самого шаблона. Проверка запускает ваш закоммиченный скрипт с заглушкой вместо реального `claude`, поэтому токены вашего плана не расходуются.

Если первый пункт не проходит, выполните `chmod +x scripts/review-staged.sh`, добавьте файл через `git add` и закоммитьте снова. Если не проходит второй пункт, изучите подсказку: она назовет отсутствующий флаг либо укажет на вызов `claude` по пути, через `npx` или изменение `PATH`. Сопоставьте строку вызова с критериями из раздела Ваша очередь, начиная с позиции промпта. Если не проходит третий пункт, запустите скрипт вручную сначала без подготовленных изменений, а затем с подготовленными, проверяя `echo $?` после каждого запуска; скрипт проверки тестирует именно закоммиченную версию, поэтому коммитьте правки перед перезапуском проверки. Если не проходит четвертый пункт, подготовьте изменение и запустите скрипт еще раз: сессия, прерванная по лимиту, не засчитывается, поэтому увеличьте лимит раундов или сузьте задачу в промпте.

## Watch out

- Запуск через `-p` не отображает диалог доверия рабочей директории: он запускает хуки из `.claude/settings.json` проекта и подключает серверы из `.mcp.json` даже в директории, которой вы никогда не давали доверия ([Start faster with bare mode](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode)). Перед написанием скриптов в чужом репозитории передавайте флаг `--setting-sources user`, чтобы Claude Code не читал файлы настроек проекта и его `.mcp.json` ([What runs before you trust a folder](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder)).
- Если установлена переменная окружения `ANTHROPIC_API_KEY`, запуск `-p` всегда использует указанный API-ключ вместо вашего тарифного плана ([Authentication precedence](https://code.claude.com/docs/en/authentication#authentication-precedence)). Значение `total_cost_usd` в JSON представляет собой расчетную оценку на стороне клиента, а не фактический счет за использование ([Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking#estimates-not-billing)).
- В репозитории, которому вы еще не подтвердили доверие, запуск `-p` игнорирует разрешающие правила `permissions.allow` из `.claude/settings.json` и выводит предупреждение `this workspace has not been trusted` в stderr; подтверждение доверия для родительской папки не учитывается ([Project allow rules and workspace trust](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust), [Error reference](https://code.claude.com/docs/en/errors#workspace-has-not-been-trusted)). Передавайте необходимые скрипту правила явно в командной строке через `--allowedTools`.

## Go further

- [Программный запуск Claude Code](https://code.claude.com/docs/en/headless): структурированный вывод со схемой `--json-schema`, потоковая передача JSON и продолжение сессии через флаг `--resume` с идентификатором `session_id`.
- [Подтверждение плана перед выполнением](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs): запуск сохраненного в прошлом уроке воркфлоу из скрипта с указанием `Workflow(<name>)` в правилах доступа.
- [Обзор Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview): запуск аналогичного цикла работы агента на Python или TypeScript.

---

<sub>Sources: [Run Claude Code programmatically](https://code.claude.com/docs/en/headless) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop) · [Agent SDK reference - TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) · [Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) · [Authentication](https://code.claude.com/docs/en/authentication) · [Tools reference](https://code.claude.com/docs/en/tools-reference) · [Error reference](https://code.claude.com/docs/en/errors) · [git diff](https://git-scm.com/docs/git-diff)</sub>

<sub>← [Запуск и сохранение динамического воркфлоу](03-dynamic-workflows.md) · [Advanced index](README.md) · [GitHub Actions](05-github-actions.md) → · Topic: [Автоматизация](../topics/automation.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-4)</sub>
