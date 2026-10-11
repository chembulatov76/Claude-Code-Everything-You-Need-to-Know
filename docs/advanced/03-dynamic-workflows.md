# Run and save a dynamic workflow

<sub>**Advanced** · Lesson 3 of 7 · about 20 minutes, plus run time · Needs: [Choose an orchestration pattern](02-orchestration-patterns.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете поручать Claude написание динамического воркфлоу для задач, охватывающих множество файлов, удерживать объем вычислений в разумных границах и сохранять воркфлоу как команду, принимающую входные данные через `args`. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can have Claude write a dynamic workflow for a job that fans out across many files, keep the run small, and save it as a command that takes its input from `args`.</details>

## You need

- Один из ваших репозиториев, содержащий как минимум две директории, в каждой из которых есть актуальный комментарий `TODO` или `FIXME`: команда `git grep -n -E 'TODO|FIXME'` покажет их наличие. Если в вашем проекте их нет, используйте локальный клон любого стороннего проекта; отправлять изменения в remote не потребуется. Запускайте проверку из вашей копии учебного шаблона с Node.js LTS. Настройте папку `.practice/` в соответствии с описанием в [Уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если еще не сделали этого.
- Расход токенов: высокий (heavy). Каждый запуск стартует нескольких субагентов, а воркфлоу `/deep-research` также ищет информацию в интернете ([Run a bundled workflow](https://code.claude.com/docs/en/workflows#run-a-bundled-workflow)). Экономный вариант: пропустите шаг 3, ограничьте каждый запуск одной небольшой папкой и запускайте Claude Code с флагом `--model sonnet`, как показано на шаге 1.

> [!NOTE]
> На тарифе Pro динамические воркфлоу отключены, пока вы не включите их в строке **Dynamic workflows** меню `/config`, а ориентир размера по умолчанию равен `small` ([Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows), [Set a size guideline](https://code.claude.com/docs/en/workflows#set-a-size-guideline)).

## The idea

Динамический воркфлоу (dynamic workflow) представляет собой скрипт на JavaScript, который Claude пишет специально под вашу задачу. Среда выполнения исполняет его в фоновом режиме, сохраняя диалоговую сессию отзывчивой ([Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows)). Скрипт запускает субагентов и сохраняет промежуточные результаты в собственных переменных, поэтому в ваш диалог попадает только итоговый отчет: планом управляет скрипт, а не контекст Claude ([Holds the plan](02-orchestration-patterns.md#the-idea), [When to use a workflow](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)).

Поскольку план выражен в виде кода, вы можете прочесть его до запуска, независимые агенты могут перепроверять выводы друг друга перед отправкой в отчет, а успешный запуск можно сохранить как команду `/<name>`, принимающую новые параметры через `args` при каждом вызове ([When to use a workflow](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow), [Pass input to a saved workflow](https://code.claude.com/docs/en/workflows#pass-input-to-a-saved-workflow)).

Ориентир размера (size guideline) подсказывает Claude, на какое число агентов следует ориентироваться при генерации скрипта. Это рекомендация, а не жесткий лимит ([Set a size guideline](https://code.claude.com/docs/en/workflows#set-a-size-guideline)).

## Worked example

1. В корне вашего репозитория запустите Claude Code в ручном режиме (Manual mode):

   ```bash
   claude --permission-mode manual
   ```

   Для экономии токенов запустите сессию на модели Sonnet:

   ```bash
   claude --permission-mode manual --model sonnet
   ```

   Строка состояния покажет индикатор `⏸ manual mode on` ([Switch permission modes](https://code.claude.com/docs/en/permission-modes#switch-permission-modes)). В режиме Manual Claude Code запрашивает подтверждение перед каждым запуском воркфлоу (если только вы ранее не выбрали запоминание для этого воркфлоу в данном проекте), благодаря чему вы видите план до старта ([Approve the plan before it runs](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs)).

   Если в коде воркфлоу не указано иное, агенты запускаются на текущей модели сессии ([Cost](https://code.claude.com/docs/en/workflows#cost)). Флаг `--model` задает модель только для текущей сессии, в то время как команда `/model` также сохранила бы ее как значение по умолчанию ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags), [Commands](https://code.claude.com/docs/en/commands#all-commands)).
2. Задайте ориентир размера воркфлоу:

   ```text
   /config workflowSizeGuideline=small
   ```

   Значение `small` инструктирует Claude генерировать минимальное количество агентов; документация рекомендует его, когда необходимо ограничить расходы на выполнение воркфлоу. Claude Code сохраняет эту настройку в файле `~/.claude.json`, а не в репозитории, и она начинает действовать со следующего запроса ([All settings](https://code.claude.com/docs/en/settings-reference#workflowsizeguideline), [Set a size guideline](https://code.claude.com/docs/en/workflows#set-a-size-guideline)).
3. Пропустите этот шаг при экономии токенов. Запустите встроенный исследовательский воркфлоу ([Run a bundled workflow](https://code.claude.com/docs/en/workflows#run-a-bundled-workflow)). Ему требуется инструмент WebSearch, недоступный на Amazon Bedrock ([WebSearch tool behavior](https://code.claude.com/docs/en/tools-reference#websearch-tool-behavior)):

   ```text
   /deep-research What changed in the Node.js permission model between v20 and v22?
   ```

   Claude Code выведет список запланированных этапов и запросит разрешение на запуск. Так как вы запустили воркфлоу по имени, он также предложит вариант **Yes, and don't ask again for `<name>` in `<path>`** ([Approve the plan before it runs](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs)). Выберите **Yes, run it**. Воркфлоу выполняется в фоне. Его агенты следуют вашим правилам доступа, поэтому в режиме Manual поиск в сети или загрузка страниц могут запросить подтверждение, и процесс будет ожидать вашего ответа ([Behavior and limits](https://code.claude.com/docs/en/workflows#behavior-and-limits)). По завершении отчет со ссылками на источники появится в вашем диалоге. Дождитесь его завершения.
4. Попросите сгенерировать собственный воркфлоу. Замените `<folder>` на небольшую директорию вашего проекта:

   ```text
   use a workflow to audit every file under <folder> for errors that are caught and then ignored, and adversarially verify each finding before reporting it
   ```

   Фраза "use a workflow" в промпте указывает Claude написать новый воркфлоу под эту задачу ([Ask for a workflow in your prompt](https://code.claude.com/docs/en/workflows#ask-for-a-workflow-in-your-prompt)). Claude Code покажет этапы и варианты: **Yes, run it**, **View raw script** и **No**. Для скрипта, написанного специально под текущую задачу, вариант "не спрашивать снова" отсутствует. Выберите **View raw script**, чтобы изучить код скрипта, или нажмите `Ctrl+G` для открытия во внешнем редакторе ([Approve the plan before it runs](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs)). Скрипт имеет структуру, похожую на пример из документации:

   ```javascript
   export const meta = {
     name: 'audit-routes',
     description: 'Audit every route handler for missing auth checks',
   }

   const found = await agent('List every .ts file under src/routes/.', {
     schema: { type: 'object', required: ['files'], properties: { files: { type: 'array', items: { type: 'string' } } } },
   })

   const audits = await pipeline(found.files, file =>
     agent(`Audit ${file} for missing authentication checks.`, { label: file }),
   )

   return audits.filter(Boolean)
   ```

   Функция `agent()` запускает одного субагента, `pipeline()` последовательно запускает по агенту на каждый элемент массива, а `parallel()` запускает группу агентов параллельно ([What the saved script looks like](https://code.claude.com/docs/en/workflows#what-the-saved-script-looks-like)). Затем выберите **Yes, run it**.
5. Во время выполнения запустите команду `/workflows` и выберите запуск аудита (если в сессии был только один запуск, он откроется сразу). Экран прогресса отображает фазы и количество агентов в них. Клавиши `↑` и `↓` переключают строки, `Enter` открывает этап, а затем конкретного агента (его промпт, недавние вызовы инструментов и результат), `Esc` возвращает назад, а клавиша `p` приостанавливает выполнение или возобновляет его ([Watch the run](https://code.claude.com/docs/en/workflows#watch-the-run)). Однострочный статус прогресса также отображается в панели задач под полем ввода ([Run a bundled workflow](https://code.claude.com/docs/en/workflows#run-a-bundled-workflow)).
6. Когда запуск завершится и отчет появится в сессии, выполните `/workflows`, выберите запуск аудита (не `/deep-research`) и нажмите клавишу `s`. В диалоге сохранения клавиша `Tab` переключает между `.claude/workflows/` в вашем проекте (доступно всей команде после коммита) и `~/.claude/workflows/` (доступно только вам во всех проектах). Убедитесь, что выбран вариант `.claude/workflows/` в проекте, и нажмите `Enter` ([Save the workflow for reuse](https://code.claude.com/docs/en/workflows#save-the-workflow-for-reuse)). Claude Code запишет скрипт как `.js` файл в ближайшую существующую папку `.claude/workflows/` между текущей директорией и корнем репозитория, либо в `.claude/workflows/` в корне проекта. Начиная со следующей сессии воркфлоу доступен как слэш-команда `/<name>`, отображаемая в автодополнении по `/` рядом с `/deep-research` ([Bundled workflows](https://code.claude.com/docs/en/workflows#bundled-workflows)).

## Your turn

**Задание:** создайте воркфлоу, который находит каждый комментарий `TODO` и `FIXME` в указанной папке, проверяет, относится ли он к текущему состоянию кода, и выводит в отчет только актуальные комментарии. Держите объем небольшим: агенты должны проверять комментарии небольшими пачками, а не запускать по агенту на каждый комментарий. Выберите две папки, в каждой из которых есть как минимум один актуальный `TODO` или `FIXME`: отчет, не нашедший ни одного комментария, не содержит координат `path:line` и не пройдет проверку.

**Критерии приемки:**

- Скрипт сохранен в `.claude/workflows/`, закоммичен и запускается по имени команды. Его первой инструкцией является литерал `export const meta` с полями `name` и `description`: использование переменных, вызовов функций или spread-оператора в этом месте скрывает команду `/<name>` из автодополнения `/` ([Edit a saved script](https://code.claude.com/docs/en/workflows#edit-a-saved-script)).
- Скрипт считывает целевую папку из переменной `args`, благодаря чему вызов `Run /<name> on <folder>` работает для любой директории без изменения кода скрипта.
- Каждый пункт отчета начинается со строки с `TODO` или `FIXME` в формате `path:line` относительно корня репозитория, после чего в одну строку поясняется, почему комментарий все еще актуален, без цитирования других координат `path:line`. Устаревшие комментарии исключаются из отчета.
- Запуск на небольшой папке не выводит предупреждение `Large workflow` в панели задач. При заданном ориентире размера это предупреждение появляется, когда запланировано больше агентов, чем предусмотрено лимитом ([Cost](https://code.claude.com/docs/en/workflows#cost)).

Один из способов реализации: создайте первую версию запросом "use a workflow" для одной директории и сохраните ее по шагу 6. Затем выполните команду `/workflow-authoring`, которая загружает документацию по созданию скриптов, и попросите Claude изменить сохраненный скрипт так, чтобы он принимал путь к папке из `args` ([Edit a saved script](https://code.claude.com/docs/en/workflows#edit-a-saved-script)). Выполните `/reload-skills`, а затем вызовите `Run /<name> on <folder>`.

Когда скрипт соответствует всем критериям, закоммитьте его. Затем запустите его по имени для двух разных директорий без редактирования кода между запусками. После каждого запуска попросите Claude сохранить отчет:

```text
Save that report to .practice/a-3-run1.md: the folder you ran it on as the first line, then one line per item that still applies, starting with its path:line, and nothing else.
```

Для второй папки сохраните отчет в `.practice/a-3-run2.md`.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный скрипт в `.claude/workflows/` содержит буквальную инструкцию `export const meta` с полями `name` и `description` первой строкой и считывает параметры из `args`: команда `git ls-tree -r --name-only HEAD .claude/workflows` отображает его.
- [ ] Файлы `.practice/a-3-run1.md` и `.practice/a-3-run2.md` называют две разные папки в своих первых строках, и каждый содержит хотя бы одну ссылку `path:line` внутри своей папки.
- [ ] Каждая указанная строка `path:line` существует и содержит комментарий `TODO` или `FIXME`.
- [ ] Самопроверка (не тестируется): вы запустили сохраненный воркфлоу по имени для обоих отчетов без правок скрипта между запусками, и ни один запуск не показал предупреждения `Large workflow` в панели задач.

Запустите команду `npm run check -- a-3 --dir <your repo>` из вашей копии учебного шаблона.

Если первый пункт завершается ошибкой, убедитесь, что скрипт закоммичен (команда `git check-ignore -v .claude/workflows/<file>.js` выведет правило, скрывающее файл; добавьте его через `git add -f` и сделайте коммит) и что он читает `args`. Если отчет указывает `path:line` вне заданной папки или не находит строк внутри нее, убедитесь, что скрипт читает папку из `args` и фильтрует файлы, запускайте `claude` в корне репозитория (чтобы пути отсчитывались от корня) и повторите запуск для папки с актуальными `TODO` или `FIXME`. Если указанная строка не содержит `TODO` или `FIXME`, возможно, отчет сослался на другие строки в обосновании или файл изменился: попросите Claude цитировать только строки самих комментариев, либо повторите запуск и сохраните отчет приведенным выше промптом.

## Watch out

- В режиме auto Claude Code запрашивает подтверждение только перед первым запуском воркфлоу: выбор **Yes** сохраняет согласие в настройках пользователя, и последующие запуски стартуют без запросов. Агенты воркфлоу подчиняются вашим правилам разрешений, и если сессия работает в режиме accept-edits, auto или bypass, они исполняются в этом же режиме ([Approve the plan before it runs](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs), [Permission modes](https://code.claude.com/docs/en/sub-agents#permission-modes)). Всегда просматривайте скрипт перед подтверждением и оставайтесь в режиме Manual для незнакомых воркфлоу.
- Выполнение воркфлоу может израсходовать существенно больше токенов, чем аналогичная задача в диалоге, и расходует лимиты тарифного плана. Сначала протестируйте воркфлоу на небольшой директории и инструктируйте Claude использовать меньшие модели для шагов, не требующих максимальной мощности. Меню `/workflows` показывает расход токенов каждым агентом в реальном времени, а клавиша `x` на выбранном запуске останавливает его. При сильном разрастании процесса строка прогресса выводит предупреждение `Large workflow`. Если выбран ориентир размера, его значение заменяет встроенный порог количества агентов; предупреждение также может сработать при большом расчетном объеме токенов. Это предупреждение не приостанавливает и не блокирует выполнение ([Cost](https://code.claude.com/docs/en/workflows#cost)).
- Клавиша `p` приостанавливает и возобновляет работу, однако полностью остановленный запуск невозможно продолжить нажатием `p`: попросите Claude перезапустить его с тем же скриптом в этой же сессии или после возобновления через `claude --resume`. Агенты, работавшие в момент остановки, начнут выполнение заново ([Resume after a pause](https://code.claude.com/docs/en/workflows#resume-after-a-pause)). Агент, вывод которого прекратился на слишком долгое время, перезапускается автоматически, и меню `/workflows` добавляет суффикс `(retry 1)` к его имени ([When an agent stalls and restarts](https://code.claude.com/docs/en/workflows#when-an-agent-stalls-and-restarts)). В версии v2.1.285 агент также перезапускается с начала своего промпта при зависании сетевого соединения на несколько минут в середине ответа (исправлено в v2.1.286) или при выходе Mac из спящего режима (исправлено в v2.1.290) ([CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)), поэтому держите систему активной во время выполнения.

## Go further

- [Сохраненный воркфлоу этого руководства](../../.claude/workflows/stale-docs-audit.js): реальный рабочий скрипт, читающий `args`, разделяющий агентов по этапам и поручающий отдельным агентам верификацию каждого утверждения перед добавлением в отчет.
- [Примеры промптов для воркфлоу](https://code.claude.com/docs/en/workflows#example-workflow-prompts): типовые шаблоны задач, такие как массовая миграция файлов или исправление кода до успешного прохождения проверки.
- [Достижение лимитов использования](https://code.claude.com/docs/en/workflows#when-a-run-hits-your-usage-limit): поведение воркфлоу при ожидании сброса лимита вместо аварийного завершения.

---

<sub>Sources: [Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Commands](https://code.claude.com/docs/en/commands) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Tools reference](https://code.claude.com/docs/en/tools-reference) · [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)</sub>

<sub>← [Выбор паттерна оркестрации](02-orchestration-patterns.md) · [Advanced index](README.md) · [Скриптинг Claude Code с `claude -p`](04-headless.md) → · Topic: [Субагенты и параллельная работа](../topics/subagents-and-parallel-work.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-3)</sub>
