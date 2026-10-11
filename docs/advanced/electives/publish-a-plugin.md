# Testing and publishing a plugin

<sub>**Advanced** · Elective · about 20 minutes, plus run time · Needs: [Share your setup with a team](../06-share-your-setup.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого факультатива вы сможете валидировать файлы плагина командой `claude plugin validate`, оценивать изменения его поведения с помощью `claude plugin eval` и публиковать его в маркетплейсе, откуда другие пользователи смогут его устанавливать и обновлять.

## You need

- Структура маркетплейса из [урока 6](../06-share-your-setup.md): маркетплейс со ссылкой на плагин по относительному пути.
- Место для практики: новая папка вне других репозиториев, которая станет отдельным приватным репозиторием GitHub. Для факультативов проверки по шаблону нет.
- bash, `jq` и git версии 2.31 или новее на macOS, Linux или WSL 2. На более старых версиях git `claude plugin eval` останавливается до запуска кейсов ([Requirements](https://code.claude.com/docs/en/plugin-evals#requirements)).
- GitHub CLI с авторизацией через `gh auth login`. Claude Code клонирует приватный маркетплейс с существующими локальными учетными данными git (SSH или HTTPS) без дополнительных запросов; для HTTPS команда `gh auth setup-git` после входа сохраняет рабочий токен ([Grant access to a private marketplace](https://code.claude.com/docs/en/plugins/host-marketplace#grant-access-to-a-private-marketplace)).
- Расход лимитов: умеренный. Каждый запуск eval выполняет реальные вызовы модели по вашему тарифу ([Test plugins with evals](https://code.claude.com/docs/en/plugin-evals)). Раздел Worked example выполняет один запуск. Полный прогон в Your turn запускает каждый кейс по три раза с плагином и без него; экономный вариант выполняет каждый кейс один раз.

## The idea

Проверяйте плагин двумя способами перед тем, как делиться им. Команда `claude plugin validate` анализирует файлы: манифесты и заголовки frontmatter; с флагом `--strict` предупреждения также приводят к ошибке ([Test and debug](https://code.claude.com/docs/en/plugins/create#test-and-debug)). Команда `claude plugin eval` оценивает поведение: каждый кейс (промпт плюс грейдеры, начисляющие баллы) выполняется в изолированной сессии только с вашим плагином (по умолчанию 3 раза, ветка WITH) и столько же раз без плагина (ветка W/OUT). Разница между ними (`Δ = WITH - W/OUT`) отражает реальный вклад плагина ([How an eval run works](https://code.claude.com/docs/en/plugin-evals#how-an-eval-run-works)).

Грейдеры типов `regex`, `tool_used`, `tool_order` и `file_exists` бесплатны; типы `llm` и `baseline` вызывают модель-судью. Добавляйте в каждый кейс один грейдер на результат и один на последовательность шагов (например, «навык был вызван») ([Choose and weight graders](https://code.claude.com/docs/en/plugin-evals#grade-the-result)).

Публикация сводится к размещению файла `.claude-plugin/marketplace.json` в git-репозитории: после пуша плагин опубликован без каких-либо согласующих форм ([Publish and distribute a plugin](https://code.claude.com/docs/en/plugins/publish#publish-through-your-own-marketplace)). Если у плагина указана версия `version`, пользователи получают обновления только при изменении этого значения ([Release a new version](https://code.claude.com/docs/en/plugins/host-marketplace#release-a-new-version)).

> [!WARNING]
> Перед публикацией репозитория маркетплейса в открытый доступ проверьте все файлы в нем: публичный репозиторий доступен каждому ([About repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories#about-repository-visibility)), и любой пользователь, способный его клонировать, может установить плагин ([Control who can install](https://code.claude.com/docs/en/plugins/publish#control-who-can-install)). Хуки или stdio MCP-серверы, добавленные в будущих версиях, будут выполняться на компьютерах обновившихся пользователей с их правами доступа ([Plugin security and trust](https://code.claude.com/docs/en/plugins/security#understand-what-a-plugin-can-do)).

## Worked example

1. Создайте репозиторий с одним плагином `commit-helper` в папке `plugins/` по схеме из руководства [Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace#create-a-marketplace). Его единственный навык формирует сообщения коммитов. Замените `<you>` вашим логином на GitHub, а `Your Name` вашим именем:

   ```bash
   mkdir -p claude-plugins/.claude-plugin claude-plugins/plugins/commit-helper/{.claude-plugin,skills/commit-message}
   cd claude-plugins && git init
   mkdir -p .practice && echo '.practice/' >> "$(git rev-parse --git-path info/exclude)"
   echo '**/evals/results/' > .gitignore
   echo '# commit-helper: drafts commit messages in a short, consistent format.' > plugins/commit-helper/README.md
   ```

   Папка `.practice/` хранит проверочные файлы без коммита (как в [уроке 1 для начинающих](../../beginner/01-install-and-look-around.md#you-need)). Правило `.gitignore` исключает отчеты тестирования в `evals/results/` ([Write and refine cases](https://code.claude.com/docs/en/plugin-evals#write-and-refine-cases)), а чек-лист релиза требует наличия README ([Prepare your plugin for release](https://code.claude.com/docs/en/plugins/publish#prepare-your-plugin-for-release)). Сохраните манифест плагина с требуемыми полями как `plugins/commit-helper/.claude-plugin/plugin.json`:

   ```json
   {
     "name": "commit-helper", "version": "1.0.0", "author": { "name": "Your Name" },
     "description": "Drafts commit messages in a short, consistent format.",
     "homepage": "https://github.com/<you>/claude-plugins", "repository": "https://github.com/<you>/claude-plugins"
   }
   ```

   Навык сохраните как `plugins/commit-helper/skills/commit-message/SKILL.md`:

   ```markdown
   ---
   name: commit-message
   description: Drafts a commit message for a described change. Use when the user asks for a commit message.
   ---

   Write a commit message for the change the user describes: a subject line in the imperative mood, at most 50 characters, with no trailing period; a blank line; then one sentence on why.
   ```

   Манифест маркетплейса сохраните как `.claude-plugin/marketplace.json`. Имя плагина `name` должно совпадать с именем в его собственном манифесте ([Keep the entry name and the manifest name the same](https://code.claude.com/docs/en/plugins/create-marketplace#keep-the-entry-name-and-the-manifest-name-the-same)):

   ```json
   {
     "name": "<you>-plugins", "description": "Plugins from <you>", "owner": { "name": "Your Name" },
     "plugins": [{ "name": "commit-helper", "source": "./plugins/commit-helper", "description": "Drafts commit messages." }]
   }
   ```

2. Выполните валидацию плагина, а затем маркетплейса: `claude plugin validate --strict ./plugins/commit-helper`, затем `claude plugin validate --strict .`. Каждая проверка завершается выводом `✔ Validation passed`. Валидация маркетплейса проверяет файл маркетплейса и `plugin.json` каждого плагина по относительному пути ([Problems that validation reports](https://code.claude.com/docs/en/plugins/create-marketplace#problems-that-validation-reports)), но не открывает файлы навыков, поэтому проверяйте саму папку плагина отдельно ([Validate a directory](https://code.claude.com/docs/en/plugins/cli-reference#validate-a-directory)).
3. Создайте тестовый кейс. Команда `claude plugin eval init --bare` создает заготовку кейса: `prompt.md` и шаблон грейдера `graders/criteria.md` с оценкой через языковую модель. В таком виде он ничего не тестирует ([Write a case manually](https://code.claude.com/docs/en/plugin-evals#write-a-case-manually)). Замените шаблон двумя бесплатными детерминированными грейдерами:

   ```bash
   cd plugins/commit-helper
   claude plugin eval init --bare names-the-rename
   cd evals/names-the-rename && rm graders/criteria.md
   printf '%s\n' '---' 'max_turns: 10' 'allowed_tools: [Skill]' '---' '' 'Write me a commit message for this change: I renamed getUser to fetchUser and updated the three call sites.' > prompt.md
   printf '%s\n' '---' 'type: tool_used' 'tool: Skill' "input_match: 'commit-message'" '---' > graders/skill-fired.md
   printf '%s\n' '---' 'type: regex' "pattern: 'fetchUser'" '---' > graders/mentions-rename.md
   cd ../..
   ```

   Сессия получает доступ только к тем инструментам, которые явно разрешены в кейсе, поэтому поле `allowed_tools: [Skill]` разрешает Claude запуск навыка ([prompt.md frontmatter](https://code.claude.com/docs/en/plugin-evals#prompt-md-fields)). Грейдер `skill-fired` проверяет вызов инструмента `Skill` с аргументом `commit-message` (фактический запуск навыка); грейдер `mentions-rename` проверяет текст ответа ([Grader types](https://code.claude.com/docs/en/plugin-evals#grader-types)). Проверяйте факт из промпта (например, `fetchUser`), так как общий текст ответа варьируется. Теперь в `evals/names-the-rename/graders/` находятся файлы `mentions-rename.md` и `skill-fired.md`.
4. Запустите кейс один раз только с плагином без отправки отчета в сеть: `claude plugin eval . --runs 1 --ablation none --no-publish` ([Command options](https://code.claude.com/docs/en/plugin-evals#command-options)). На вопрос `Trust this plugin directory?` ответьте `y`; внутри git-репозитория это отмечает доверенным весь репозиторий, включая интерактивные сессии ([Trust the plugin directory](https://code.claude.com/docs/en/plugin-evals#trust-the-plugin-directory)). В итоговой таблице выводятся колонки `SCORE` и `PASS%`, а строка `Report:` указывает путь к `report.html` в `evals/results/` ([Create your first eval suite](https://code.claude.com/docs/en/plugin-evals#create-your-first-eval-suite)). Оценки могут отличаться.
5. Опубликуйте плагин. В корне репозитория (`cd ../..`) выполните `git add -A && git status --short`, просмотрите список файлов и сделайте коммит с созданием репозитория на GitHub:

   ```bash
   git commit -m "Publish commit-helper 1.0.0"
   gh repo create claude-plugins --private --source . --push
   ```

   Параметр `--source .` создает репозиторий GitHub из текущей папки, а `--push` отправляет коммит ([gh repo create](https://cli.github.com/manual/gh_repo_create)). Маркетплейс опубликован для всех, кто имеет доступ к репозиторию (пока только для вас). Перевод репозитория в публичный сделает его общедоступным.
6. Установите плагин как обычный пользователь из GitHub:

   ```bash
   claude plugin marketplace add <you>/claude-plugins
   claude plugin install commit-helper@<you>-plugins
   claude plugin details commit-helper
   ```

   Команда добавления сообщает `✔ Successfully added marketplace: <you>-plugins (declared in user settings)`, установка выводит `✔ Successfully installed plugin: commit-helper@<you>-plugins (scope: user)`, а в описании деталей в секции `Component inventory` отображается `Skills (1)  commit-message` ([Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace#create-a-marketplace)). Имя маркетплейса берется из `marketplace.json`, а не из имени репозитория ([Host your marketplace](https://code.claude.com/docs/en/plugins/host-marketplace#host-your-marketplace)). Плагин останется установленным до завершения раздела Your turn.

## Your turn

**Задание:** выпустите версию `commit-helper` 1.0.1, подтвержденную автотестами, и обновите установленную копию.

**Критерии приемки:**

- Второй кейс `ignores-unrelated` имеет тот же frontmatter, что и `names-the-rename`, давая навыку возможность сработать, но передает запрос, к которому навык неприменим (например, `Explain what a git rebase does.`). Грейдер `tool_used` для `Skill` с `min: 0`, `max: 0` и `arm: both` проверяет, что навык не был вызван. Опция `arm: both` обязательна, так как двухветочный запуск иначе исключает грейдеры `tool_used` по `Skill` из итоговой оценки ([Score against the no-plugin baseline](https://code.claude.com/docs/en/plugin-evals#compare-against-a-no-plugin-baseline)). Регулярное выражение проверяет ответ (например, `pattern: 'rebase'` с `flags: i`).
- В навык внесено изменение, распознаваемое грейдером (например, добавление префикса вроде `refactor:` в тему), что проверяется новым грейдером `regex` в `names-the-rename`. Поле `version` в `plugin.json` равно `1.0.1`, а запись в маркетплейсе не переопределяет версию ([Release a new version](https://code.claude.com/docs/en/plugins/host-marketplace#release-a-new-version)).
- Обе строгие валидации проходят успешно, а полный прогон `claude plugin eval . --threshold <score> --no-publish` из папки плагина завершается с результатом каждого кейса не ниже заданного порога от 0 до 1 (например, `0.8`) ([Command options](https://code.claude.com/docs/en/plugin-evals#command-options)). Экономный вариант: добавьте флаги `--runs 1 --ablation none`.
- Папка `evals/` закоммичена, папка `evals/results/` исключена из коммита, а релиз отправлен в GitHub.
- Команда `claude plugin update commit-helper@<you>-plugins` обновляет плагин до 1.0.1 и завершается строкой `Restart to apply changes.` ([Plugin loading reference](https://code.claude.com/docs/en/plugins/loading#check-which-stage-a-plugin-reached)). Сообщение о том, что плагин уже актуален, говорит о том, что версия еще не отправлена на GitHub.

Затем из корня репозитория сохраните список плагинов командой `claude plugin list --json > .practice/a-publish-list.json` ([JSON output](https://code.claude.com/docs/en/plugins/cli-reference#json-output)), и только после этого выполните `claude plugin marketplace remove <you>-plugins --scope user`. Удаление маркетплейса из последней области видимости автоматически деинсталлирует его плагины ([plugin marketplace remove](https://code.claude.com/docs/en/plugins/cli-reference#plugin-marketplace-remove)), поэтому сохраненный список необходим для проверки Check ниже. Удаление самого репозитория опционально: для `gh repo delete <you>/claude-plugins --yes` требуется скоуп `delete_repo`, добавляемый через `gh auth refresh -s delete_repo` ([gh repo delete](https://cli.github.com/manual/gh_repo_delete)).

## Check

Задание выполнено, когда из корня репозитория выполняются все условия:

- [ ] Команды `claude plugin validate --strict ./plugins/commit-helper` и `claude plugin validate --strict .` завершаются строкой `✔ Validation passed`.
- [ ] `git ls-files plugins/commit-helper/evals` содержит файлы двух кейсов и не содержит файлов из `results/`.
- [ ] Последний результат eval охватывает оба кейса, и все кейсы пройдены ([JSON result](https://code.claude.com/docs/en/plugin-evals#json-result)): `jq '.partial != true and .aggregates.casesTotal == 2 and .aggregates.casesPassed == .aggregates.casesTotal' "$(ls -d plugins/commit-helper/evals/results/*/ | tail -1)aggregate-result.json"` возвращает `true`.
- [ ] `jq -r '.[] | select(.id == "commit-helper@<you>-plugins") | .version' .practice/a-publish-list.json` возвращает `1.0.1`.
- [ ] Самопроверка (не проверяется скриптом): вы можете объяснить, что измеряет величина `Δ`, и почему двухветочный прогон не оценивает критерий «навык был вызван».

Если валидация завершается ошибкой только при наличии `--strict`, строки над вердиктом указывают на предупреждение (например, отсутствие `description` или `version`, см. [Output and exit codes](https://code.claude.com/docs/en/plugins/cli-reference#output-and-exit-codes)). Если в списке есть `results/`, удалите их из индекса: `git rm -r --cached plugins/commit-helper/evals/results`, проверьте `.gitignore` и закоммитьте. Если результат возвращает `false`, запустите оба кейса заново, если в последнем прогоне был только один, или откройте `report.html`: без ключа `--threshold` любой балл ниже максимума считается провалом ([The run exits 1 but the results look fine](https://code.claude.com/docs/en/plugin-evals#the-run-exits-1-but-the-results-look-fine)). Если проверка версии ничего не выводит или возвращает `1.0.0`, убедитесь в замене `<you>`, отправьте новую версию на GitHub, добавьте маркетплейс, переустановите плагин и сохраните список до удаления маркетплейса.

## Watch out

- Команда `claude plugin eval` выполняет навыки, хуки и агенты плагина локально от вашего имени, а успешное прохождение тестов не гарантирует безопасность самого кода. Передавайте флаги `--allow-tools`, `--scaffold` или `--allow-real-servers` только для тех плагинов и тестов, исходный код которых вы лично изучили ([What a run can access](https://code.claude.com/docs/en/plugin-evals#security)).
- Грейдеры на базе языковой модели добавляют вызовы API на каждый запуск, поэтому используйте их только там, где детерминированные проверки бессильны. Параметр `--max-cost-usd` ограничивает оценку по базовым тарифам API, а не квоту подписки, и при исчерпании лимитов посреди набора кейсов последующие тесты обычно получают 0, хотя весь набор не помечается как `partial` ([Command options](https://code.claude.com/docs/en/plugin-evals#command-options), [Troubleshooting](https://code.claude.com/docs/en/plugin-evals#runs-fail-with-a-usage-limit-or-rate-limit-error-partway-through)).
- Относитесь к имени плагина `name` как к постоянному идентификатору: установка, включение и настройка привязаны к формату `name@marketplace`, поэтому смена имени делает плагин совершенно новым для всех существующих установок. Для изменения отображаемого имени используйте поле `displayName` ([Prepare your plugin for release](https://code.claude.com/docs/en/plugins/publish#prepare-your-plugin-for-release)). Некоторые имена маркетплейсов зарезервированы (например, начинающиеся с `claudeai-`); при отказе выберите другое имя ([Reserved names](https://code.claude.com/docs/en/plugins/marketplace-reference#reserved-names)).

## Go further

- [Запуск eval в CI](https://code.claude.com/docs/en/plugin-evals#run-evals-in-ci): ключ `--trust-plugin`, фиксированные модели, формат `--json` и ограничение бюджета для пайплайнов вроде рассмотренного в уроке по [GitHub Actions](../05-github-actions.md).
- [Публикация в каталоге Anthropic](https://code.claude.com/docs/en/plugins/publish#submit-to-anthropics-directory): размещение плагина в общем каталоге claude.ai.
- [Хостинг и поддержка маркетплейса](https://code.claude.com/docs/en/plugins/host-marketplace): автообновление, релизные каналы, переименование и удаление плагинов.

---

<sub>Sources: [Test plugins with evals](https://code.claude.com/docs/en/plugin-evals) · [Publish and distribute a plugin](https://code.claude.com/docs/en/plugins/publish) · [Create a Claude Code plugin](https://code.claude.com/docs/en/plugins/create) · [Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace) · [Host and maintain a marketplace](https://code.claude.com/docs/en/plugins/host-marketplace) · [Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference) · [Plugin loading reference](https://code.claude.com/docs/en/plugins/loading) · [Marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference) · [Plugin security and trust](https://code.claude.com/docs/en/plugins/security) · GitHub: [About repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories) · GitHub CLI manual: [gh repo create](https://cli.github.com/manual/gh_repo_create), [gh repo delete](https://cli.github.com/manual/gh_repo_delete)</sub>

<sub>Back to the [Advanced index](../README.md) · Topic: [Плагины](../../topics/plugins.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-publish-plugin)</sub>
