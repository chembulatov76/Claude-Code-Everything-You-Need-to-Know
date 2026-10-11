# Advanced capstone: add CI review your team can share

<sub>**Advanced** · Capstone · 30 to 60 minutes, plus run time · Needs: the seven [Advanced lessons](README.md), or their skills · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

Это итоговое практическое задание объединяет весь материал уровня Advanced в новой отдельной копии учебного шаблона, которую вы полностью удаляете в конце. Здесь нет пошагового примера: в описании сформулированы потребности команды, а архитектурные решения вы принимаете самостоятельно. [Критерии завершения Advanced](README.md#by-the-end-of-this-level-you-can) представляют собой полный квалификационный профиль уровня. Скрипт проверки анализирует закоммиченные файлы и два отчета в `.practice/`, а остальные пункты вы отмечаете лично в разделе [Вне зоны видимости скрипта проверки](#what-the-check-cant-see).

## You need

- Node.js LTS, git, `jq` и GitHub CLI с выполненным входом `gh auth login`.
- Новая приватная копия учебного шаблона, созданная исключительно для этого практикума и удаляемая после его прохождения:

  ```bash
  gh repo create claude-capstone --template wesammustafa/claude-code-practice --private --clone
  cd claude-capstone
  ```

  Флаг `--template` разворачивает проект из шаблона, а `--clone` клонирует его локально ([gh repo create](https://cli.github.com/manual/gh_repo_create)). Скрипт проверки запускается в этой копии без параметра `--dir`.
- В созданной копии запустите `claude` один раз, подтвердите доверие директории и выйдите. В директории без подтверждения доверия запуск `claude --worktree` завершается ошибкой ([Start Claude in a worktree](https://code.claude.com/docs/en/worktrees#start-claude-in-a-worktree)).
- Учетные данные для CI: отдельный API-ключ Claude Console (рекомендуется, так как процедура очистки удаляет его) либо токен подписки из `claude setup-token` для тарифов Pro, Max, Team или Enterprise ([Manual setup](https://code.claude.com/docs/en/github-actions#manual-setup)).
- На тарифе Pro включенные динамические воркфлоу в строке **Dynamic workflows** меню `/config` ([Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows)).
- macOS, Linux или WSL 2: скрипт автоматического ревью написан на bash.
- Расход токенов: высокий (heavy). Две одновременные сессии, динамический воркфлоу, автономный скрипт и запуск в CI. Каждый запуск в CI расходует минуты GitHub Actions и токены (оплата по счету Console при использовании API-ключа либо списание с лимитов плана при использовании токена) ([Manage costs](https://code.claude.com/docs/en/github-actions#manage-costs)). Экономный вариант: запускайте все сессии с параметром `--model sonnet` ([Model aliases](https://code.claude.com/docs/en/model-config#model-aliases)), добавьте `--model sonnet` в скрипт и в `claude_args` ([Pass CLI arguments](https://code.claude.com/docs/en/github-actions#pass-cli-arguments)), а в динамическом воркфлоу части 4 ограничьте анализ двумя файлами из `src/`.

## The task

Команда хочет настроить автоматическое ревью pull requests через Claude в CI, работающее одинаково на машинах всех разработчиков и не содержащее неконтролируемых действий. Полное техническое задание находится в файле `capstone/advanced-brief.md` вашей копии репозитория.

Реализуйте каждую часть в соответствии с материалами уроков:

1. **Две сессии одновременно** ([Урок 1](01-parallel-sessions.md)). Добавьте `.claude/worktrees/` в закоммиченный `.gitignore` и отправьте изменения в `main`: новые worktrees ответвляются от ветки по умолчанию удаленного репозитория ([Choose the base branch](https://code.claude.com/docs/en/worktrees#choose-the-base-branch)). Затем в двух отдельных терминалах одновременно запустите `claude --worktree <name>` с разными именами ([Reuse a worktree name](https://code.claude.com/docs/en/worktrees#reuse-a-worktree-name)): одна сессия разрабатывает часть 2, другая часть 3, и каждая делает коммит в своей ветке. Когда в обеих ветках зафиксированы коммиты и оба worktree существуют, сохраните их список из основного дерева:

   ```bash
   mkdir -p .practice && git worktree list --porcelain > .practice/a-capstone-worktrees.txt
   ```

   Затем слейте обе ветки в `main` командой `git merge` (не `git merge --squash`) и удалите оба рабочих дерева.
2. **Скрипт ревью** ([Урок 4](04-headless.md)): `scripts/review-staged.sh` по критериям Урока 4. Он передает подготовленный diff через конвейер в один вызов `claude -p` с выводом в формате JSON и в режиме `dontAsk` ([dontAsk mode](https://code.claude.com/docs/en/permission-modes#allow-only-pre-approved-tools-with-dontask-mode)), предварительно разрешает только `Read`, `Grep` и `Glob`, и ограничивает раунды числом 10 или менее ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)). После слияния веток подготовьте однострочное изменение в основном дереве, запустите скрипт один раз, прочитайте рецензию и закоммитьте правку:

   ```bash
   mkdir -p .practice && scripts/review-staged.sh .practice/a-capstone-run.json
   ```

   Значение `total_cost_usd` в сохраненном JSON дает оценку стоимости одного ревью. Как и при расчете среза в Уроке 7, это расчетная оценка клиента, а не финансовый счет ([Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking#estimates-not-billing)).
3. **Командный плагин** ([Урок 6](06-share-your-setup.md)): локальный каталог плагинов в папке `team-marketplace/`, содержащий один плагин по относительному пути (в собственной подпапке внутри `team-marketplace/`), как в примере Урока 6, и как минимум один навык или агент в составе плагина. Закоммиченный файл `.claude/settings.json` регистрирует каталог по относительному пути и включает плагин ([Require plugins per repository](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository)). Обе проверки валидности завершаются успешно: `claude plugin validate` для каталога и для директории плагина с флагом `--strict` ([plugin validate](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate)). Шаблон набора из примера Урока 6 служит хорошей отправной точкой. Тестируйте плагин только после завершения слияния веток: относительный путь рассчитывается от основного дерева репозитория даже внутри worktree ([Require plugins per repository](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository)), поэтому сессия части 3 еще не может загрузить его на этапе разработки.
4. **Сохраненный динамический воркфлоу** ([Уроки 2](02-orchestration-patterns.md) и [3](03-dynamic-workflows.md)): в сессии в основном рабочем дереве после слияния веток запросите воркфлоу, проверяющий каждый файл в `src/` с перепроверкой каждого замечания вторым агентом. Во время работы меню `/workflows` показывает расход токенов каждым агентом ([Cost](https://code.claude.com/docs/en/workflows#cost)). Сохраните воркфлоу в проект: сохранение в проект записывает файл в ближайшую существующую директорию `.claude/workflows/` между рабочей папкой сессии и корнем проекта либо в `.claude/workflows/` в корне ([Save the workflow for reuse](https://code.claude.com/docs/en/workflows#save-the-workflow-for-reuse)). Закоммитьте полученный файл.
5. **CI-ревью с ограничениями** ([Уроки 5](05-github-actions.md) и [7](07-bounded-runs.md)): файл `.github/workflows/claude-review.yml` запускается по событию `pull_request` ([Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request)) и использует экшен `anthropics/claude-code-action` (Anthropic) с параметром `prompt`, запрашивающим рецензирование изменений в PR ([Action parameters](https://code.claude.com/docs/en/github-actions#action-parameters)). Соблюдайте все ограничения Урока 5:
   - экшен зафиксирован на релизном теге или полном SHA коммита ([Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idstepsuses));
   - учетные данные считываются из секрета репозитория через соответствующий входной параметр `anthropic_api_key` или `claude_code_oauth_token` ([Manual setup](https://code.claude.com/docs/en/github-actions#manual-setup));
   - токен задачи доступен только на чтение: никаких прав на запись в блоке `permissions`, кроме `id-token: write` ([Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#permissions)), а параметр `github_token` указывает на собственный токен задачи, благодаря чему Claude действует от имени этого токена ([Action parameters](https://code.claude.com/docs/en/github-actions#action-parameters));
   - в `claude_args` передаются `--max-turns` со значением 10 или меньше и список `--allowedTools` без открытого доступа к Bash;
   - параметр `timeout-minutes` ограничен 30 минутами или менее ([Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idtimeout-minutes)).

   При текстовом промпте у Claude нет доступа к терминалу, пока вы явно не разрешите необходимые команды ([Run on a schedule](https://code.claude.com/docs/en/github-actions#run-on-a-schedule)). Чтобы агент увидел изменения, выполните чекаут полной истории коммитов (как в `.github/workflows/progress.yml` шаблона) и разрешите команду `git diff` через правило `Bash(git diff *)`, соответствующее только командам `git diff` ([Wildcard patterns](https://code.claude.com/docs/en/permissions#wildcard-patterns)). В промпте явно запросите выполнение `git diff` в виде отдельной одиночной команды.

   Установите `display_report: true` по аналогии с Уроком 5, чтобы рецензия Claude публиковалась в сводке запуска ([action.yml at v1.0.237](https://github.com/anthropics/claude-code-action/blob/v1.0.237/action.yml), [Adding a job summary](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands#adding-a-job-summary)). Включайте этот параметр только для доверенных входных данных, так как репозиторий является вашей личной приватной копией.

   Добавьте ключ в секреты репозитория: выполните `gh secret set ANTHROPIC_API_KEY` или `gh secret set CLAUDE_CODE_OAUTH_TOKEN` и вставьте значение ([gh secret set](https://cli.github.com/manual/gh_secret_set)). Никогда не коммитьте секреты в код файлов ([Protect your credentials](https://code.claude.com/docs/en/github-actions#protect-your-credentials)).

   Закоммитьте воркфлоу в ветку `main` и отправьте в remote. Затем отправьте небольшую правку в новой ветке и откройте pull request. Ознакомьтесь с рецензией Claude на странице сводки выполнения воркфлоу и слейте pull request вручную.

Завершите работу на ветке `main`: подтяните слитый PR, убедитесь, что все изменения закоммичены и команда `npm test` завершается успешно.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный файл `.gitignore` игнорирует `.claude/worktrees/` и не затрагивает остальное содержимое `.claude/`.
- [ ] Файл `.practice/a-capstone-worktrees.txt` содержит список двух рабочих деревьев в `.claude/worktrees/`, коммиты которых разошлись, и оба влиты в вашу ветку.
- [ ] Закоммиченный скрипт в `.claude/workflows/` содержит буквальный литерал `export const meta` с полями `name` и `description` в качестве первой инструкции и запускает субагентов через `agent()`.
- [ ] Скрипт `scripts/review-staged.sh` закоммичен и имеет права на исполнение.
- [ ] Вызов `claude -p` запрашивает вывод в JSON, работает в режиме `dontAsk`, предварительно разрешает только `Read`, `Grep` и `Glob`, ограничивает количество раундов 10 или менее и содержит промпт сразу после `-p`.
- [ ] Он передает подготовленный diff через stdin, сохраняет JSON по переданному пути, выводит результат, завершается с ненулевым кодом при сбое и не вызывает Claude при отсутствии подготовленных изменений.
- [ ] Файл `.practice/a-capstone-run.json` содержит успешно завершенный запуск в рамках установленного лимита.
- [ ] Закоммиченный воркфлоу запускает экшен `anthropics/claude-code-action` по событию `pull_request`, зафиксирован на теге релиза или полном SHA коммита, в режиме автоматизации, с получением учетных данных из секрета и без ключей в файле, с токеном только для чтения (без прав записи в `permissions`, кроме `id-token: write`, с параметром `github_token`, привязанным к токену задачи), лимитом `--max-turns` 10 или менее, списком `--allowedTools` без неограниченного Bash и `timeout-minutes` не более 30 минут.
- [ ] Никакие API-ключи или токены Claude не закоммичены ни в `HEAD`, ни в коммите, из которого скрипт проверки считывает воркфлоу.
- [ ] Закоммиченный командный каталог плагинов содержит плагин по относительному пути, и имя записи совпадает с полем `name` в файле `plugin.json` плагина.
- [ ] Закоммиченный `.claude/settings.json` регистрирует маркетплейс по относительному пути и включает плагин.
- [ ] Команда `npm test` проходит успешно, а вывод `git status --porcelain` пуст.

Запустите команду `npm run check -- a-capstone` в данной копии шаблона. При сбоях вывод укажет, что необходимо исправить. Скрипт проверки запускает локальный скрипт с заглушкой вместо Claude и считывает воркфлоу из истории git, поэтому проверка сохраняет статус прохождения как до, так и после процедуры очистки.

## What the check can't see

Скрипт проверки не может автоматически проконтролировать следующие пункты квалификационного профиля уровня Advanced. Отметьте их самостоятельно:

- [ ] Я запускал обе сессии worktree одновременно, и ни одна из них не видела незакоммиченных изменений другой.
- [ ] Я могу проследить задачу пофайлового ревью с перепроверкой вторым агентом по вопросам Урока 2 и обосновать, почему она требует именно динамического воркфлоу, а не простых субагентов в одном диалоге.
- [ ] Я понимаю, в каких случаях автономному запуску требуется песочница из [Урока 7](07-bounded-runs.md), и требуется ли она моему скрипту ревью и воркфлоу CI.
- [ ] Я лично наблюдал запуск CI для моего pull request и ознакомился с рецензией Claude на сводной странице перед слиянием PR.
- [ ] Новая сессия в основном рабочем дереве после слияния веток автоматически загрузила командный плагин без команд ручной установки.
- [ ] До включения воркфлоу в CI я использовал значение `total_cost_usd` локального запуска (по аналогии со срезом в Уроке 7) для оценки расходов на одно ревью в CI, и оценил расход токенов в меню `/workflows`.

Когда все пункты на этой странице отмечены, вы подтвердили профиль уровня Advanced и завершили все три уровня практического руководства. Остался последний шаг: процедура очистки ниже.

## Tear down

После успешного прохождения проверки выполните процедуру очистки: это обязательное требование.

> [!WARNING]
> До выполнения очистки активные учетные данные остаются в секретах репозитория. Удаление секрета оставляет действительными сами ключи и токены ([Uninstall](https://code.claude.com/docs/en/github-actions#uninstall)). Токен подписки из `claude setup-token` действует длительное время ([Authentication](https://code.claude.com/docs/en/authentication#generate-a-long-lived-token)), и простое удаление секрета не отзывает сам токен.

1. Удалите воркфлоу командой `git rm .github/workflows/claude-review.yml`, закоммитьте и отправьте изменения. После удаления файла экшен Claude Code больше не запустится ([Uninstall](https://code.claude.com/docs/en/github-actions#uninstall)).
2. Удалите секрет: `gh secret delete ANTHROPIC_API_KEY` или `gh secret delete CLAUDE_CODE_OAUTH_TOKEN` ([gh secret delete](https://cli.github.com/manual/gh_secret_delete)). Команда `gh secret list` больше не будет показывать его ([gh secret list](https://cli.github.com/manual/gh_secret_list)).
3. При использовании API-ключа удалите временный ключ на странице [API keys](https://platform.claude.com/settings/keys) в Claude Console.

Запустите команду `npm run check -- a-capstone` еще раз: проверка по-прежнему завершается успешно.

---

<sub>Sources: [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees) · [Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows) · [Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Model configuration](https://code.claude.com/docs/en/model-config) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference) · [Manage Claude Code plugins for your organization](https://code.claude.com/docs/en/plugins/org) · [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions) · [Authentication](https://code.claude.com/docs/en/authentication) · claude-code-action at v1.0.237: [action.yml](https://github.com/anthropics/claude-code-action/blob/v1.0.237/action.yml) · GitHub: [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows), [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax), [Workflow commands](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands) · GitHub CLI manual: [gh repo create](https://cli.github.com/manual/gh_repo_create), [gh secret set](https://cli.github.com/manual/gh_secret_set), [gh secret delete](https://cli.github.com/manual/gh_secret_delete), [gh secret list](https://cli.github.com/manual/gh_secret_list) · the [Advanced lessons](README.md), which cite the official pages for each step.</sub>

<sub>← [Установка ограничений для автономных запусков](07-bounded-runs.md) · [Advanced index](README.md) · [За рамками этого руководства](beyond-this-guide.md) → · [Stuck on this capstone?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-capstone)</sub>
