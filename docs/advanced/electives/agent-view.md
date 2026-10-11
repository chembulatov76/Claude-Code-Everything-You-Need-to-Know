# Agent view

<sub>**Advanced** · Elective · about 20 minutes, plus run time · Needs: [Parallel sessions with worktrees](../01-parallel-sessions.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

> [!NOTE]
> «Agent view находится в статусе research preview. Интерфейс и сочетания клавиш могут меняться по мере развития функции.» ([Manage multiple agents with agent view](https://code.claude.com/docs/en/agent-view))

## Goal

К концу этого факультатива вы сможете поручать несколько независимых задач фоновым сессиям с одного экрана, вмешиваться только когда сессии требуется ваше участие, а также сливать или отклонять результаты каждой из них.

## You need

- Ваша копия учебного шаблона и Node.js LTS для раздела Worked example; один из ваших собственных репозиториев для Your turn. Вы практикуетесь в одноразовых клонах, чьим remote является локальная папка на вашем компьютере, так что ни один push не покидает вашу машину. Шаг 1 создает клон для Worked example.
- macOS, Linux или WSL 2 и два терминала: один для agent view, второй для git. На macOS храните клоны вне Desktop, Documents и Downloads, где фоновая сессия может возвращать `Operation not permitted`, пока вы не предоставите явный доступ ([Background sessions can't read Desktop, Documents, or Downloads on macOS](https://code.claude.com/docs/en/agent-view#background-sessions-can%E2%80%99t-read-desktop-documents-or-downloads-on-macos)).
- Расход лимитов: умеренный. Каждая сессия расходует квоту независимо ([Quick start](https://code.claude.com/docs/en/agent-view#quick-start)), а однострочная сводка для каждой строки создается коротким запросом класса Haiku при завершении шага и раз в несколько минут во время долгого шага ([Row summaries](https://code.claude.com/docs/en/agent-view#row-summaries)). Экономный вариант: в шаге 3 запустите только первую задачу, а в качестве второй введите `! npm test`. Shell-задача выполнит команду вместо Claude без вызова языковой модели, а ее строка очистится через несколько минут после завершения команды ([Run a shell command](https://code.claude.com/docs/en/agent-view#run-a-shell-command)). Шаги 4-8 в этом случае применяются к единственной сессии Claude: ожидайте одно worktree, одну ветку и одну сессию для удаления.

## The idea

[Agent view](https://code.claude.com/docs/en/agent-view), открываемый командой `claude agents`, представляет собой единый экран для управления фоновыми сессиями: полноценными диалогами Claude Code, работающими без привязки к терминалу. Каждый введенный запрос запускает отдельную сессию, которую фоновый сервис продолжает выполнять даже после закрытия экрана. Каждая строка отображает состояние сессии: в работе, ожидает ввода или завершена ([Read session state](https://code.claude.com/docs/en/agent-view#read-session-state)).

В [уроке 1](../01-parallel-sessions.md) вы создавали рабочие деревья вручную. Здесь это делает Claude: перед тем как запущенная сессия приступает к редактированию файла, она перемещается в собственное worktree в `.claude/worktrees/`. Claude Code инструктирует ее коммитить в ветку этого worktree и отправлять ветку в удаленный репозиторий, если у репозитория есть remote (если только задача, ваш `CLAUDE.md` или память не указывают, что git вы ведете вручную). Сессии строго запрещено отправлять изменения в `main` или `master`, делать force-push или слияния. Любая задача завершается отчетом о сделанном и указанием места расположения работы ([How file edits are isolated](https://code.claude.com/docs/en/agent-view#how-file-edits-are-isolated)).

> [!WARNING]
> Фоновая сессия выполняет действия без вашего прямого наблюдения. При запуске из оболочки без ключа `--permission-mode` agent view запускает каждую сессию точно так же, как обычная сессия `claude` в этой папке, чаще всего в режиме auto ([Permission mode](https://code.claude.com/docs/en/agent-view#permission-mode), [Which mode a session starts in](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in)), где Claude коммитит без подтверждений и пушит ветку при наличии remote. Запускайте agent view с явно выбранным режимом разрешений, никогда не используйте `--dangerously-skip-permissions` и практикуйтесь в клоне с локальным remote на вашей машине.

## Worked example

1. Из папки с копией учебного шаблона создайте одноразовый клон с локальным remote и проверьте прохождение тестов. Замените `<template copy>` именем папки вашей копии:

   ```bash
   git clone --bare <template copy> av-origin.git
   git clone av-origin.git av-practice
   cd av-practice
   mkdir -p .practice
   printf '%s\n' .practice/ .claude/worktrees/ >> "$(git rev-parse --git-path info/exclude)"
   npm test
   ```

   `npm test` завершается успешно, а `origin` указывает на локальную папку `av-origin.git`, поэтому пуш не покинет ваш компьютер. Строки в exclude скрывают две папки из `git status` без коммита: `.practice/`, где сохраняются результаты для Your turn (как в [уроке 1 для начинающих](../../beginner/01-install-and-look-around.md#you-need)), и рабочие деревья, создаваемые Claude ([gitignore](https://git-scm.com/docs/gitignore), [Start Claude in a worktree](https://code.claude.com/docs/en/worktrees#start-claude-in-a-worktree)).
2. В первом терминале откройте agent view с двумя настройками по умолчанию: ручной режим (Manual), в котором без подтверждения выполняются только операции чтения ([Available modes](https://code.claude.com/docs/en/permission-modes#available-modes)), и модель Sonnet ([Model aliases](https://code.claude.com/docs/en/model-config#model-aliases)):

   ```bash
   claude agents --permission-mode manual --model sonnet
   ```

   Примите диалог доверия для новой папки. В шапке отображается модель по умолчанию для новых сессий ([Set the model](https://code.claude.com/docs/en/agent-view#set-the-model)), и каждая сессия из этого экрана запускается с этими параметрами ([Dispatch defaults](https://code.claude.com/docs/en/agent-view#dispatch-defaults)). Если у вас нет других фоновых сессий, список покажет пустые заголовки разделов с описаниями. Если `claude agents` выводит список субагентов и завершается, убедитесь, что в конфигурациях параметр `disableAgentView` не равен `true` и переменная `CLAUDE_CODE_DISABLE_AGENT_VIEW` не задана ([Turn off agent view](https://code.claude.com/docs/en/agent-view#turn-off-agent-view)).
3. Запустите две задачи, затрагивающие разные файлы. Введите каждую и нажмите `Enter`; каждый запрос запускает отдельную сессию:

   ```text
   Add a --json option to src/cli.js that prints the broken links as a JSON array instead of text lines. Add a test for it in a new file, test/cli.test.js, and run npm test.
   ```

   ```text
   Write docs/config.md describing each linkcheck.json option in src/config.js, with its default and an example. Don't change any code.
   ```

   Под заголовком `Working` появятся две строки, названные по их запросам.
4. Когда сессии требуется ваше разрешение (на редактирование или команду), ее строка перемещается в категорию `Needs input`. Выберите ее стрелками `↑` или `↓` и нажмите `Пробел`: панель предварительного просмотра покажет, чего именно ждет агент ([Peek and reply](https://code.claude.com/docs/en/agent-view#peek-and-reply)). Ответ, введенный в этой панели, не подтверждает диалог прав, поэтому нажмите `→`, чтобы подключиться. В версии v2.1.285 сессия, ожидающая подтверждения, могла ошибочно отображаться как завершенная (исправлено в v2.1.286), а `claude agents` иногда не отображал запрос разрешений (исправлено в v2.1.287) ([CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)): если строка показывает завершение без отчета или в peek нет запроса, подключитесь клавишей `→`.
5. При подключении сессия занимает весь терминал с кратким пересказом событий за время вашего отсутствия. Ознакомьтесь с намерениями Claude и подтвердите действие, если оно соответствует задаче, но не выбирайте `Yes, and switch to auto mode`: это перевело бы автономную сессию в полностью автоматический режим ([Switch permission modes](https://code.claude.com/docs/en/permission-modes#switch-permission-modes)). Вы можете оставаться подключенным и одобрять следующие шаги (например, `git commit` или `git push`), а затем нажать `←` в пустой строке ввода: вы вернетесь к таблице, а сессия продолжит выполнение в фоне ([Attach to a session](https://code.claude.com/docs/en/agent-view#attach-to-a-session)). Поступайте так же каждый раз, когда сессии требуется подтверждение.
6. Во втором терминале в папке `av-practice` проверьте статус репозитория, пока сессии работают:

   ```bash
   git status --short
   git worktree list
   ```

   `git status --short` ничего не выводит: сессии вносят изменения в других местах. Как только сессия запросит редактирование, `git worktree list` покажет для нее рабочее дерево в `.claude/worktrees/` с указанием ветки в квадратных скобках.
7. Когда строки сессий переместятся в категорию `Completed`, загляните в них (`Пробел`): отчет содержит описание сделанного и ссылку на ветку (формулировки могут отличаться). Результат без описания может все еще ждать подтверждения: подключитесь клавишей `→` и ответьте на запрос. Во втором терминале слейте нужные изменения и запустите тесты:

   ```bash
   git merge --no-edit <branch>
   npm test
   ```

   Выполните слияние для каждой сохраняемой ветки, используя имя ветки из отчета сессии или `git worktree list`. `npm test` проходит успешно; если вы слили ветку с опцией `--json`, тестов выполнится больше, чем на шаге 1.
8. Выполните очистку во втором терминале. Запросите список сессий для этой папки с их короткими идентификаторами `id` и удалите каждую:

   ```bash
   claude agents --json --all --cwd "$PWD"
   claude rm <id>
   ```

   Команда `claude rm` также удаляет рабочее дерево, созданное Claude, если это безопасно. Она сохранит worktree, если в нем остались незакоммиченные правки или коммиты, не обнаруженные в других ветках, и сообщит причину. Коммиты в remote или слитые в текущую дефолтную ветку считаются сохраненными ([What deleting a session removes](https://code.claude.com/docs/en/agent-view#what-deleting-a-session-removes)). После этого `git worktree list` показывает только вашу основную рабочую копию. Нажмите `Esc` в agent view для возврата в командную строку.

## Your turn

**Задание:** в одноразовом клоне вашего собственного репозитория (созданном как в шаге 1 с новыми именами папок и вашей командой тестирования) поручите агентам две независимые задачи, затрагивающие разные файлы. Каждое worktree представляет собой чистую рабочую копию ([Set up the worktree environment](https://code.claude.com/docs/en/worktrees#set-up-the-worktree-environment)), поэтому задача с запуском тестов должна сначала установить зависимости в своем worktree, как в уроке 1. Пока обе сессии работают, сохраните два файла в `.practice/`, указанные в критериях ниже.

Затем во втором терминале запустите интерактивную сессию в клоне командой `claude --permission-mode manual`: сессия, переводимая в фон, сохраняет свой режим разрешений, тогда как обычный запуск `claude` стартует в auto ([Permission mode](https://code.claude.com/docs/en/agent-view#permission-mode)). Задайте вопрос по коду, не требующий правок, и переведите сессию в agent view командой `/background`, что освободит терминал и вернет вас в shell ([Send the session to the background](https://code.claude.com/docs/en/agent-view#send-the-session-to-the-background)). Переведенная сессия продолжает работу прямо в вашей рабочей копии без отдельного worktree. Контролируйте все три сессии из agent view, пока результаты каждой не будут объединены или отклонены.

**Критерии приемки:**

- Agent view открыт с режимом разрешений и моделью, выбранными для этой работы.
- Две запущенные фоновые сессии изменяют файлы независимо друг от друга.
- Во время работы обеих сессий, после того как каждая запросила редактирование, и до запуска сессии, переводимой в фон, сохранено состояние рабочей копии:

  ```bash
  git status --short > .practice/a-view-status.txt
  git worktree list > .practice/a-view-worktrees.txt
  ```

- Интерактивная сессия переведена в agent view командой `/background` (псевдоним `/bg`) после ответа на вопрос ([Commands](https://code.claude.com/docs/en/commands)).
- На каждый запрос в категории `Needs input` дан ответ: на вопросы через панель peek, на запросы разрешений через прямое подключение.
- Выбранные изменения объединены в вашу ветку, тесты проходят, а все созданные сессии удалены с помощью `claude rm <id>`.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Ваша рабочая копия оставалась чистой во время работы сессий, и у каждой было отдельное worktree: `.practice/a-view-status.txt` пуст, а `.practice/a-view-worktrees.txt` содержит пути внутри `.claude/worktrees/` для каждой из них. В клоне команда `[ ! -s .practice/a-view-status.txt ] && grep /.claude/worktrees/ .practice/a-view-worktrees.txt` выводит по одной строке на сессию.
- [ ] `git log --oneline` на вашей ветке содержит сохраненную работу, и тесты проходят.
- [ ] `claude agents --json --all --cwd "$PWD"` не содержит сессий со значением `kind: background`, а `git worktree list` не показывает записей в `.claude/worktrees/`.
- [ ] Самопроверка (не проверяется скриптом): вы ответили на все запросы `Needs input` из agent view без переключения в shell и можете объяснить, какие запросы решились через панель peek, а какие потребовали подключения.

Если первый пункт не сошелся, сохраните файлы повторно при следующем запуске, пока обе сессии активны: сессия до первой попытки редактирования еще не имеет worktree, переведенная сессия редактирует рабочую копию на месте, а настройка [`worktree.bgIsolation`](https://code.claude.com/docs/en/settings-reference#worktree-bgisolation) со значением `"none"` отключает изоляцию рабочих деревьев. Если в ветке нет нужной работы или тесты падают после слияния, найдите соответствующую сессию. Сессия оставляет git пользователю, если задача, `CLAUDE.md` или память указывают на ручной режим; незакоммиченные изменения в worktree нужно закоммитить перед слиянием: подключитесь к сессии и попросите сделать коммит. Если ветка сломала тесты, исправьте проблему сами или подключитесь к сессии до удаления и укажите на ошибку, затем слейте исправление. Если `claude rm` не удалил сессию, прочитайте причину (незакоммиченные правки или неотправленные коммиты) и следуйте руководству [What deleting a session removes](https://code.claude.com/docs/en/agent-view#what-deleting-a-session-removes). Оставшиеся без сессии worktree удаляются командой `git worktree remove <path>` ([`.claude/worktrees/` is filling up](https://code.claude.com/docs/en/agent-view#claude%2Fworktrees%2F-is-filling-up)), предварительно сняв блокировку через `git worktree unlock <path>`, если git сообщает о блокировке ([Clean up subagent and background-session worktrees](https://code.claude.com/docs/en/worktrees#clean-up-subagent-and-background-session-worktrees)).

## Watch out

- Удаление сессии в agent view двойным нажатием `Ctrl+X` удаляет созданное для нее worktree вместе с незакоммиченными правками; `claude rm` сохраняет рабочее дерево с изменениями ([What deleting a session removes](https://code.claude.com/docs/en/agent-view#what-deleting-a-session-removes)). Сначала закоммитьте или слейте ценные данные.
- Фоновые сессии расходуют квоту так же, как интерактивные, поэтому параллельный запуск нескольких сессий ускоряет расход лимитов во столько же раз ([Limitations](https://code.claude.com/docs/en/agent-view#limitations)).
- Команда `/agents` внутри сессии отличается от внешней команды `claude agents`, несмотря на схожее название ([Check on running work](https://code.claude.com/docs/en/agents#check-on-running-work)).

## Go further

- [Чтение состояния сессий из скрипта](https://code.claude.com/docs/en/agent-view#read-session-state-from-a-script): опрос `claude agents --json --all` из строки состояния или внешнего инструмента.
- [Копирование сессии командой /fork](https://code.claude.com/docs/en/agent-view#copy-the-session-with-%2Ffork): отправка копии диалога в фон с продолжением текущей работы на месте.
- [Обмен сообщениями между сессиями Claude Code](https://code.claude.com/docs/en/cross-session-messaging): передача промежуточных результатов между параллельно запущенными сессиями.

---

<sub>Sources: [Manage multiple agents with agent view](https://code.claude.com/docs/en/agent-view) · [Run agents in parallel](https://code.claude.com/docs/en/agents) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Commands](https://code.claude.com/docs/en/commands) · [Model configuration](https://code.claude.com/docs/en/model-config) · [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [gitignore](https://git-scm.com/docs/gitignore)</sub>

<sub>Back to the [Advanced index](../README.md) · Topic: [Субагенты и параллельная работа](../../topics/subagents-and-parallel-work.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-agent-view)</sub>
