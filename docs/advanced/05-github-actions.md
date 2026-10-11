# GitHub Actions

<sub>**Advanced** · Lesson 5 of 7 · about 20 minutes, plus run time · Needs: [Script Claude Code with `claude -p`](04-headless.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете запускать Claude Code автономно в воркфлоу GitHub Actions, безопасно считывающем учетные данные из секретов репозитория и жестко ограниченном лимитами, заданными прямо в файле конфигурации. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can run Claude Code unattended in a GitHub Actions workflow that reads its credential from a repository secret and stops within limits written in the file.</details>

## You need

- Флаги `claude -p` из [Урока 4](04-headless.md), такие как `--max-turns` и `--allowedTools`.
- Установленный [GitHub CLI](https://cli.github.com) с выполненным входом `gh auth login`, необходимый для быстрой настройки ([Quick setup](https://code.claude.com/docs/en/github-actions#quick-setup)).
- Новая приватная копия учебного шаблона, созданная исключительно для этого урока. Вы являетесь ее владельцем с полными правами ([Permission levels for a personal account repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/repository-access-and-collaboration/permission-levels-for-a-personal-account-repository#owner-access-for-a-repository-owned-by-a-personal-account)), включая права администратора для настройки ([Setup](https://code.claude.com/docs/en/github-actions#setup)). Создайте ее командой:

  ```bash
  gh repo create claude-actions-practice --template wesammustafa/claude-code-practice --private --clone
  cd claude-actions-practice
  ```

  Флаг `--template` разворачивает новый репозиторий из шаблона, а `--clone` клонирует его локально ([gh repo create](https://cli.github.com/manual/gh_repo_create)). Проверка запускается в этой копии с Node.js LTS без флага `--dir`. В конце урока вы полностью удалите воркфлоу, секрет и доступ GitHub App.
- Учетные данные. Рекомендуется создать отдельный API-ключ Claude Console для этого урока, так как процедура очистки удаляет его. На тарифах Pro, Max, Team или Enterprise ([Manual setup](https://code.claude.com/docs/en/github-actions#manual-setup)) быстрая настройка может выпустить токен подписки ([Quick setup](https://code.claude.com/docs/en/github-actions#quick-setup)). Если Claude Code на вашем компьютере уже настроен через API-ключ, быстрая настройка повторно использует его, поэтому удаление секрета затронет и ключ вашей локальной среды; шаг 3 пошагового примера показывает, как заменить его на временный ключ.
- Расход токенов: минимальный (light). Каждый запуск короток, но также расходует минуты GitHub Actions (см. раздел Предостережения).

## The idea

[GitHub Action для Claude Code](https://code.claude.com/docs/en/github-actions), `anthropics/claude-code-action` (Anthropic), запускает Claude Code в качестве отдельного шага воркфлоу. Без входного параметра `prompt` он ожидает упоминания `@claude` в issues или pull requests и отвечает комментарием; при наличии `prompt` запуск происходит сразу без ожидания упоминания ([Interactive and automation modes](https://code.claude.com/docs/en/github-actions#interactive-and-automation-modes)), а входной параметр `display_report` публикует отчет Claude на странице сводки выполнения ([action.yml at v1.0.237](https://github.com/anthropics/claude-code-action/blob/v1.0.237/action.yml)).

Воркфлоу указывает имя секрета репозитория, содержащего ключ или токен доступа ([Manual setup](https://code.claude.com/docs/en/github-actions#manual-setup)).

Поскольку за клавиатурой никого нет, ограничения задаются прямо в файле. Параметр `claude_args` передает флаги командной строки Claude Code, такие как `--max-turns` и `--allowedTools` ([Pass CLI arguments](https://code.claude.com/docs/en/github-actions#pass-cli-arguments)), а при передаче промпта в виде обычного текста у Claude нет доступа к терминалу, пока вы явно не разрешите нужные команды ([Run on a schedule](https://code.claude.com/docs/en/github-actions#run-on-a-schedule)). Параметры `permissions` и `timeout-minutes` на уровне job ограничивают права токена GitHub и максимальное время выполнения ([Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idpermissions)).

## Worked example

Подключите копию репозитория через быструю настройку, затем задайте вопрос Claude в issue.

1. В копии шаблона запустите `claude`, выполните `/install-github-app` и следуйте подсказкам для этого репозитория: Claude Code установит приложение Claude GitHub App ([Quick setup](https://code.claude.com/docs/en/github-actions#quick-setup)). Когда GitHub предложит выбрать доступные репозитории, выберите **Only select repositories** и укажите только созданную копию ([Installing a GitHub App](https://docs.github.com/en/apps/using-github-apps/installing-a-github-app-from-a-third-party#installing-a-github-app)). Если приложение уже установлено на вашем аккаунте, добавьте репозиторий в разделе **Repository access** ([Modifying repository access](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps#modifying-repository-access)).
2. Claude Code спросит, продолжить ли настройку GitHub Actions. Продолжите и выберите только интерактивный воркфлоу `@claude` (`claude.yml`), без воркфлоу ревью. В качестве учетных данных укажите временный ключ урока или создайте токен подписки. Claude Code сохранит секрет `ANTHROPIC_API_KEY` или `CLAUDE_CODE_OAUTH_TOKEN`, отправит ветку с выбранным воркфлоу и откроет браузер со страницей создания pull request ([Quick setup](https://code.claude.com/docs/en/github-actions#quick-setup)).
3. Создайте и слейте этот pull request, затем обновите локальный клон и проверьте список секретов:

   ```bash
   git pull
   gh secret list
   ```

   Список покажет секрет `ANTHROPIC_API_KEY` или `CLAUDE_CODE_OAUTH_TOKEN` ([gh secret list](https://cli.github.com/manual/gh_secret_list)). Слитый файл `.github/workflows/claude.yml` уже настроен на использование этого секрета, и триггер `@claude` теперь активен в репозитории ([Quick setup](https://code.claude.com/docs/en/github-actions#quick-setup)). Если настройка переиспользовала ваш основной ключ, замените его на временный ключ урока: команда `gh secret set ANTHROPIC_API_KEY` запросит новое значение ([gh secret set](https://cli.github.com/manual/gh_secret_set)).
4. Создайте issue с упоминанием `@claude`. В интерактивном режиме Claude реагирует на вызов `@claude` в теле или заголовке созданного issue ([Interactive and automation modes](https://code.claude.com/docs/en/github-actions#interactive-and-automation-modes)):

   ```bash
   gh issue create --title "What does links.js do?" --body "@claude In one paragraph, what does src/links.js do? Don't change any files."
   ```

   Команда `gh issue create` выведет URL созданного issue ([gh issue create](https://cli.github.com/manual/gh_issue_create)). Команда `gh run list --limit 3` покажет запущенный процесс ([gh run list](https://cli.github.com/manual/gh_run_list)). По завершении команда `gh issue view <issue-url> --comments` покажет ответ Claude в комментариях ([gh issue view](https://cli.github.com/manual/gh_issue_view)).

## Your turn

**Задание:** добавьте файл `.github/workflows/linkcheck-report.yml` с воркфлоу ручного запуска, в котором Claude выполняет проверку ссылок шаблона в папке `samples/` и формирует пояснения по каждой битой ссылке в сводке запуска (job summary), не изменяя файлы в репозитории. В качестве основы используйте пример [Run on a schedule](https://code.claude.com/docs/en/github-actions#run-on-a-schedule).

**Критерии приемки:**

- Запускается только вручную: триггер `on: workflow_dispatch` без других событий.
- Использует экшен `anthropics/claude-code-action`, зафиксированный на релизном теге или полном SHA коммита, но не `@v1` (причины в разделе Предостережения). Релиз v1.0.237 устанавливает версию Claude Code v2.1.285, с которой тестировалась эта страница ([run.ts at v1.0.237](https://github.com/anthropics/claude-code-action/blob/v1.0.237/src/entrypoints/run.ts)).
- Содержит параметр `prompt`, работая в режиме автоматизации.
- Получает учетные данные из сохраненного секрета через соответствующий входной параметр `anthropic_api_key` или `claude_code_oauth_token` ([Manual setup](https://code.claude.com/docs/en/github-actions#manual-setup)). Секреты не прописываются в коде файла.
- Обладает токеном только для чтения: без прав на запись в блоке `permissions`, за исключением `id-token: write`. В блоке заданы `contents: read` и опционально `id-token: write` для аутентификации GitHub App по умолчанию ([Respond to @claude mentions](https://code.claude.com/docs/en/github-actions#respond-to-@claude-mentions)). Параметр `github_token` указывает на собственный `GITHUB_TOKEN` задачи, благодаря чему Claude действует от имени этого токена ([Action parameters](https://code.claude.com/docs/en/github-actions#action-parameters)).
- В `claude_args` передается `--max-turns` со значением 10 или менее, а также правило `--allowedTools` строго для команды проверки ссылок, без открытого доступа ко всей оболочке Bash ([Configure permissions](https://code.claude.com/docs/en/permissions#wildcard-patterns)). Задавайте лимит раундов двумя словами: `--max-turns <n>`, как в документации ([Pass CLI arguments](https://code.claude.com/docs/en/github-actions#pass-cli-arguments)), так как экшен считывает значение из следующего слова ([parse-sdk-options.ts at v1.0.237](https://github.com/anthropics/claude-code-action/blob/v1.0.237/base-action/src/parse-sdk-options.ts)).
- В задаче установлен `timeout-minutes` со значением 30 или менее ([Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idtimeout-minutes)).
- Шаги воркфлоу выполняют чекаут репозитория и настраивают Node.js до вызова экшена, аналогично `.github/workflows/progress.yml` в шаблоне.
- Параметр `display_report` установлен в `true`, благодаря чему отчет Claude появляется в сводке запуска ([action.yml at v1.0.237](https://github.com/anthropics/claude-code-action/blob/v1.0.237/action.yml), [Adding a job summary](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands#adding-a-job-summary)). Включайте этот параметр только для доверенных входных данных: промпт и файлы контролируются вами. Параметр `show_full_output` отключен, и отладочный режим `ACTIONS_STEP_DEBUG` не активирован, так как полный вывод может раскрыть секреты в логах ([security notes](https://github.com/anthropics/claude-code-action/blob/v1.0.237/docs/security.md)).

Затем выполните запуск, сохраните результат и проведите процедуру очистки:

1. Закоммитьте воркфлоу и отправьте в ветку `main`. Ручной запуск доступен только тогда, когда файл воркфлоу находится в ветке по умолчанию ([`workflow_dispatch`](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#workflow_dispatch)).
2. Запустите воркфлоу, определите его ID, дождитесь окончания и откройте сводку в браузере ([gh workflow run](https://cli.github.com/manual/gh_workflow_run), [gh run watch](https://cli.github.com/manual/gh_run_watch), [gh run view](https://cli.github.com/manual/gh_run_view)):

   ```bash
   gh workflow run linkcheck-report.yml
   gh run list --workflow linkcheck-report.yml --limit 1
   gh run watch <run-id> --exit-status
   gh run view <run-id> --web
   ```

   Сводка выполнения отобразит отчет Claude с пояснениями по каждой битой ссылке.
3. Сохраните запись о выполнении для скрипта проверки со всеми требуемыми полями ([gh run list](https://cli.github.com/manual/gh_run_list)):

   ```bash
   mkdir -p .practice
   gh run list --workflow linkcheck-report.yml --json databaseId,status,conclusion,event,headSha,url,workflowName > .practice/a-5-runs.json
   ```

4. Запустите проверку `npm run check -- a-5`. Когда она пройдет успешно, обязательно выполните шаги очистки. Не удаляйте секреты до прохождения проверки: устранение возможных ошибок требует рабочего секрета.

> [!WARNING]
> Удаление секрета оставляет действительными сами учетные данные ([Uninstall](https://code.claude.com/docs/en/github-actions#uninstall)). Секрет `CLAUDE_CODE_OAUTH_TOKEN` содержит OAuth-токен типа `claude setup-token` ([Action parameters](https://code.claude.com/docs/en/github-actions#action-parameters)). Токен действует длительное время ([Authentication](https://code.claude.com/docs/en/authentication#generate-a-long-lived-token)), и простое удаление секрета из репозитория не отзывает сам токен.

5. Удалите оба воркфлоу, затем закоммитьте и отправьте изменения:

   ```bash
   git rm .github/workflows/claude.yml .github/workflows/linkcheck-report.yml
   git commit -m "Remove the Claude workflows"
   git push
   ```

6. Удалите секрет: `gh secret delete ANTHROPIC_API_KEY` или `gh secret delete CLAUDE_CODE_OAUTH_TOKEN` ([gh secret delete](https://cli.github.com/manual/gh_secret_delete)).
7. При использовании API-ключа удалите временный ключ на странице [API keys](https://platform.claude.com/settings/keys) в Claude Console.
8. На GitHub перейдите в **Settings** > **Applications** > **Installed GitHub Apps** и нажмите **Configure** возле Claude. Удалите доступ для тестового репозитория в разделе **Repository access** либо нажмите **Uninstall**, если приложение больше не используется ([Modifying repository access](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps#modifying-repository-access), [Blocking access](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps#blocking-access)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный воркфлоу запускает `anthropics/claude-code-action`, зафиксированный на теге релиза или SHA коммита, в режиме автоматизации, с получением учетных данных из секрета и без открытых ключей в файле.
- [ ] Ограничения прописаны в файле: `--max-turns` со значением 10 или менее, список `--allowedTools` без неограниченного Bash, токен задачи только для чтения (без прав записи в `permissions`, кроме `id-token: write`) и `timeout-minutes` не более 30 минут.
- [ ] Файл `.practice/a-5-runs.json` подтверждает завершенный успешный ручной запуск на коммите, содержащем воркфлоу.
- [ ] Самопроверка (не тестируется): Claude ответил на ваше issue комментарием, а сводка выполнения воркфлоу объясняет найденные битые ссылки.

Запустите `npm run check -- a-5` в данной копии шаблона.

Если первый пункт не проходит на строке `uses:`, зафиксируйте конкретный тег релиза (например, `@v1.0.237`) или полный SHA коммита. Если не проходит второй пункт, подсказка укажет на отсутствующий или некорректный лимит. Если сохраненный запуск завершился сбоем, команда `gh run view <run-id> --log-failed` покажет причину: несоответствие имени секрета или превышение лимита раундов ([CLI reference](https://code.claude.com/docs/en/cli-reference#cli-flags)); исправьте ошибку, перезапустите воркфлоу и перезапишите JSON-файл.

Скрипт проверки считывает воркфлоу из истории git, поэтому проверка сохраняет статус прохождения даже после очистки файлов. После успешного прохождения выполните процедуру очистки по шагам с 5 по 8.

## Watch out

- Любой пользователь с правами на запись в репозиторий может инициировать запуск через `@claude`, а текст issue или комментария поступает на вход модели и может содержать скрытые инструкции (промпт-инъекции) ([Who can trigger runs](https://code.claude.com/docs/en/github-actions#who-can-trigger-runs), [the action's security notes](https://github.com/anthropics/claude-code-action/blob/v1.0.237/docs/security.md)). Держите тестовый репозиторий приватным: только пользователи с правами на чтение видят логи выполнения воркфлоу ([Using workflow run logs](https://docs.github.com/en/actions/how-tos/monitor-workflows/use-workflow-run-logs)).
- Каждый запуск расходует минуты GitHub Actions и токены Claude. В приватном репозитории минуты списываются с лимита аккаунта ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)). Токены оплачиваются по счету Console либо списываются с лимитов плана. Флаги `--max-turns` и параметр `timeout-minutes` сдерживают максимальный объем вычислений, сокращая расходы ([Manage costs](https://code.claude.com/docs/en/github-actions#manage-costs)).
- Тег `v1` в примерах документации перемещается с каждым новым релизом ([tags](https://github.com/anthropics/claude-code-action/tags)), а каждый релиз фиксирует конкретную версию Claude Code, поэтому воркфлоу на `@v1` может неожиданно запустить более новую версию. GitHub отмечает, что фиксация на полном SHA коммита является единственным способом сделать запуск экшена полностью неизменяемым ([Secure use reference](https://docs.github.com/en/actions/reference/security/secure-use#using-third-party-actions)). Фиксация на релизном теге вида `v1.0.237` является допустимым компромиссом для этого урока.

## Go further

- [Запуск по расписанию](https://code.claude.com/docs/en/github-actions#run-on-a-schedule): аналогичный режим автоматизации на триггере cron.
- [Настройка для организации](https://code.claude.com/docs/en/github-actions#set-up-for-an-organization): одна установка GitHub App и общий секрет организации для множества репозиториев, либо интеграция workload identity federation вместо постоянных ключей.
- [Руководство по готовым решениям экшена](https://github.com/anthropics/claude-code-action/blob/v1.0.237/docs/solutions.md) (Anthropic): готовые шаблоны воркфлоу для ревью pull request, регулярного обслуживания и триажа issues.

---

<sub>Sources: [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions) · [Authentication](https://code.claude.com/docs/en/authentication) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Configure permissions](https://code.claude.com/docs/en/permissions) · claude-code-action at v1.0.237: [action.yml](https://github.com/anthropics/claude-code-action/blob/v1.0.237/action.yml), [run.ts](https://github.com/anthropics/claude-code-action/blob/v1.0.237/src/entrypoints/run.ts), [parse-sdk-options.ts](https://github.com/anthropics/claude-code-action/blob/v1.0.237/base-action/src/parse-sdk-options.ts), [run-claude-sdk.ts](https://github.com/anthropics/claude-code-action/blob/v1.0.237/base-action/src/run-claude-sdk.ts), [security](https://github.com/anthropics/claude-code-action/blob/v1.0.237/docs/security.md), and its [tags](https://github.com/anthropics/claude-code-action/tags) · GitHub: [Installing a GitHub App from a third party](https://docs.github.com/en/apps/using-github-apps/installing-a-github-app-from-a-third-party), [Permission levels for a personal account repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/repository-access-and-collaboration/permission-levels-for-a-personal-account-repository), [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows), [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax), [Workflow commands](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands), [Secure use reference](https://docs.github.com/en/actions/reference/security/secure-use), [Using workflow run logs](https://docs.github.com/en/actions/how-tos/monitor-workflows/use-workflow-run-logs), [Reviewing and modifying installed GitHub Apps](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps), [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions) · GitHub CLI manual: [gh repo create](https://cli.github.com/manual/gh_repo_create), [gh issue create](https://cli.github.com/manual/gh_issue_create), [gh issue view](https://cli.github.com/manual/gh_issue_view), [gh secret list](https://cli.github.com/manual/gh_secret_list), [gh secret set](https://cli.github.com/manual/gh_secret_set), [gh workflow run](https://cli.github.com/manual/gh_workflow_run), [gh run list](https://cli.github.com/manual/gh_run_list), [gh run watch](https://cli.github.com/manual/gh_run_watch), [gh run view](https://cli.github.com/manual/gh_run_view), [gh secret delete](https://cli.github.com/manual/gh_secret_delete)</sub>

<sub>← [Скриптинг Claude Code с `claude -p`](04-headless.md) · [Advanced index](README.md) · [Совместное использование настроек в команде](06-share-your-setup.md) → · Topic: [Автоматизация](../topics/automation.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-5)</sub>
