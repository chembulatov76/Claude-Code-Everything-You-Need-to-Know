# Agent teams

<sub>**Advanced** · Elective · about 20 minutes, plus run time · Needs: [Choose an orchestration pattern](../02-orchestration-patterns.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

> [!NOTE]
> Команды агентов являются экспериментальной возможностью и отключены по умолчанию ([Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams)).

## Goal

К концу этого факультатива вы сможете комплектовать команду агентов из собственных определений субагентов, подтверждать факт формирования команды и корректно останавливать ее участников по именам. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can staff an agent team from your own subagent definitions, show that a team formed, and shut its teammates down by name.</details>

## You need

- Материал урока [Выбор паттерна оркестрации](../02-orchestration-patterns.md) и файлы определений субагентов, написанные в [Делегирование кастомному субагенту](../../intermediate/06-subagents.md).
- Один из ваших репозиториев с настроенной папкой `.practice/` по описанию в [Уроке 1 уровня Beginner](../../beginner/01-install-and-look-around.md#you-need).
- macOS, Linux или WSL 2, а также утилита `jq`: хуки, логирующие состав команды, используют ее в командах оболочки.
- Интерактивная сессия: в автономном запуске `-p` Claude не порождает участников команды ([Enable agent teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)). Режим отображения по умолчанию (in-process), в котором участники работают прямо внутри вашего терминала, функционирует в любом окружении ([Choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode)).
- Расход токенов: высокий (heavy). Каждый участник команды представляет собой отдельный экземпляр Claude с собственным контекстным окном, а расход токенов масштабируется с числом активных участников ([Token usage](https://code.claude.com/docs/en/agent-teams#token-usage)). Экономный вариант: измените один небольшой файл в пошаговом примере, запустите двух участников в разделе Ваша очередь и останавливайте участников сразу после получения их отчетов.

## The idea

Команда агентов (agent team) объединяет несколько сессий Claude Code в совместную работу. Ваша сессия выступает ведущей (lead): она порождает участников команды (teammates), каждого в отдельном окне контекста. Участники могут напрямую обмениваться сообщениями друг с другом, а вы можете обращаться к любому из них (в отличие от субагентов, возвращающих результат исключительно вызывающей стороне) ([Compare with subagents](https://code.claude.com/docs/en/agent-teams#compare-with-subagents)).

Участник команды загружает ваш CLAUDE.md, серверы MCP и навыки, а также вводный промпт ведущего, но не наследует историю диалога ведущего, поэтому задача полностью формулируется в промпте ([Context and communication](https://code.claude.com/docs/en/agent-teams#context-and-communication)). Укажите в промпте имя определения субагента, и участник запустится с его инструментами `tools`, инструкциями и моделью `model` (если в промпте не задана другая модель); встроенный участник также получает инструмент `SendMessage`, поэтому даже участник только для чтения может связываться с остальными ([Use subagent definitions for teammates](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)).

Команды агентов расходуют существенно больше токенов по сравнению с одиночной сессией ([Token usage](https://code.claude.com/docs/en/agent-teams#token-usage)), а участники разделяют общее рабочее дерево ([Choose an approach](https://code.claude.com/docs/en/agents#choose-an-approach)). Применяйте их там, где необходимы независимые точки зрения: разностороннее ревью с разных позиций или открытый спор альтернативных гипотез.

## Worked example

1. В вашем репозитории загрузите двух ревьюеров только для чтения из [примера команд агентов](../../../examples/advanced/electives/agent-teams/README.md). Проверьте файлы перед использованием:

   ```bash
   mkdir -p .claude/agents
   for a in security-reviewer test-reviewer; do
     curl -fsSL "https://raw.githubusercontent.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/main/examples/advanced/electives/agent-teams/dot-claude/agents/$a.md" -o ".claude/agents/$a.md"
   done
   grep -H '^tools:' .claude/agents/security-reviewer.md .claude/agents/test-reviewer.md
   ```

   Каждая строка заканчивается на `tools: Read, Grep, Glob`: ревьюеры могут читать файлы, но не могут редактировать их или запускать команды, включая `git`. Поисковые инструменты Grep и Glob по умолчанию скрыты в macOS, Linux и WSL; субагент, перечисляющий их без инструмента Bash, делает их доступными ([Glob tool behavior](https://code.claude.com/docs/en/tools-reference#glob-tool-behavior)). Оба агента используют `model: sonnet`. Если сессия уже была открыта и папка `.claude/agents/` была создана только что, перезапустите сессию: Claude Code сканирует директории агентов только на старте ([Write subagent files](https://code.claude.com/docs/en/sub-agents#write-subagent-files)).
2. Возьмите реальное изменение, над которым вы работаете, либо внесите временную тестовую правку: измените один отслеживаемый файл (например, функцию, принимающую пользовательский ввод, без добавления теста). Оставьте изменение незакоммиченным. Ревьюеры не имеют доступа к `git diff`, поэтому сохраните diff в файл:

   ```bash
   git diff > .practice/a-teams.diff
   ```

3. Сохраните конфигурацию как `.practice/a-teams.json`:

   ```json
   {
     "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
     "teammateMode": "in-process",
     "hooks": {
       "TeammateIdle": [
         { "hooks": [ { "type": "command", "command": "jq -r '.teammate_name' >> \"$CLAUDE_PROJECT_DIR/.practice/a-teams-teammates.txt\"" } ] }
       ]
     }
   }
   ```

   Она включает команды агентов только для сессий, запущенных с этим файлом ([Enable agent teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams), [Claude spawns teammates instead of subagents](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)), а параметр `teammateMode` задает режим отображения in-process ([`teammateMode`](https://code.claude.com/docs/en/settings-reference#teammatemode)). Хук [`TeammateIdle`](https://code.claude.com/docs/en/hooks#teammateidle) записывает имя каждого участника в `.practice/a-teams-teammates.txt` перед переходом в режим ожидания.
4. Запустите ведущую сессию с этим файлом:

   ```bash
   CLAUDE_CODE_ENABLE_TODO_TOOLS=1 claude --settings .practice/a-teams.json
   ```

   Параметр `--settings` применяет настройки файла поверх вашей конфигурации только для текущей сессии ([CLI reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). Переменная окружения активирует инструменты Task на всех моделях; без нее Claude Code предоставляет их только на старых моделях из списка доступности ([Task tool availability](https://code.claude.com/docs/en/tools-reference#task-tool-availability)), и участники координируют действия через сообщения вместо общего списка задач. Выполните команду `/context`: в списке субагентов появятся `security-reviewer` и `test-reviewer` ([See what loaded into context](https://code.claude.com/docs/en/debug-your-config#see-what-loaded-into-context)).
5. Запросите запуск команды. Назначьте каждому участнику свою специализацию, как в примере параллельного ревью из документации ([run a parallel code review](https://code.claude.com/docs/en/agent-teams#run-a-parallel-code-review)), а также имя и тип агента ([use subagent definitions for teammates](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)):

   ```text
   Create an agent team to review the change saved in .practice/a-teams.diff and the files it touches.
   Spawn two teammates: one named security, using the security-reviewer agent type, and one named
   tests, using the test-reviewer agent type. When both have reported, write one combined report
   to .practice/a-teams-review.md.
   ```

   Claude Code запускает участников команды без дополнительного подтверждения ([How Claude starts agent teams](https://code.claude.com/docs/en/agent-teams#how-claude-starts-agent-teams)). В панели агентов под строкой ввода появится строка для каждого участника. Комбинация `Ctrl+T` переключает отображение списка задач ([Talk to teammates directly](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)). Промпт не указывает модель, поэтому участники запускаются на модели из своего определения, если не задана переменная `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` ([Specify teammates and models](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models)). Когда оба участника завершат работу, ведущий сформирует объединенный отчет; подтвердите запись при запросе.
6. Остановите участников команды по именам:

   ```text
   Ask the security and tests teammates to shut down.
   ```

   Каждый участник подтверждает запрос и завершает работу либо отклоняет его с объяснением причин ([Shut down teammates](https://code.claude.com/docs/en/agent-teams#shut-down-teammates)); если участник отклонил запрос, ознакомьтесь с причиной и повторите команду. Участник, чья строка исчезла из панели, скрыт, но не остановлен, поэтому явно называйте его в запросе ([Start your first agent team](https://code.claude.com/docs/en/agent-teams#start-your-first-agent-team)). Затем завершите сессию. Claude Code удаляет конфигурацию команды при выходе из сессии и сохраняет список задач для возобновляемых сессий ([Architecture](https://code.claude.com/docs/en/agent-teams#architecture)).
7. Проверьте лог:

   ```bash
   sort -u .practice/a-teams-teammates.txt
   ```

   Команда выведет `security` и `tests`, подтверждая создание команды.

## Your turn

Проведите расследование проблемы через конкуренцию гипотез, как в примере [Investigate with competing hypotheses](https://code.claude.com/docs/en/agent-teams#investigate-with-competing-hypotheses). Выберите реальный баг или неочевидное поведение в вашем репозитории, причина которого не ясна с первого взгляда. Создайте определение исследователя только для чтения (например, скопировав `test-reviewer.md` в `.claude/agents/hypothesis-tester.md` с изменением инструкций). Запустите новую сессию аналогичным образом и поручите ей создать трех участников команды на базе этого определения, где каждый участник отстаивает одну гипотезу и оппонирует теориям других через сообщения. Экономный вариант: два участника.

Критерии приемки:

- Определение содержит собственные поля `name` и `description`, `model: sonnet` и однострочный параметр `tools:` с указанием только `Read`, `Grep` и `Glob` по аналогии с примером из документации ([Quickstart: create your first subagent](https://code.claude.com/docs/en/sub-agents#quickstart-create-your-first-subagent)). Определение без параметра `tools` наследует все доступные субагентам инструменты ([Frontmatter reference](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields)).
- Каждый участник запускается на базе вашего определения под выбранным в промпте именем.
- Ведущий сохраняет выдержавшую проверку гипотезу и опровергнутые теории в `.practice/a-teams-debate.md`.
- Отслеживаемые файлы не изменились сверх вашей тестовой правки из пошагового примера.
- Вы явно запросили завершение работы каждого участника команды по имени.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Каждое использованное участниками команды определение работает в режиме только для чтения. Следующая команда не выводит ошибок при корректном параметре `tools:` (укажите имя файла вашего определения):

  ```bash
  for f in .claude/agents/security-reviewer.md .claude/agents/test-reviewer.md .claude/agents/hypothesis-tester.md; do
    grep -qxE 'tools: *(Read|Grep|Glob)( *, *(Read|Grep|Glob))* *' "$f" || echo "$f: tools: names more than Read, Grep and Glob, or is missing"
  done
  ```

- [ ] Была сформирована полноценная команда, а не просто субагенты: `sort -u .practice/a-teams-teammates.txt` содержит все выбранные вами имена участников.
- [ ] Оба отчета сохранены: команда `ls .practice/a-teams-review.md .practice/a-teams-debate.md` находит оба файла.
- [ ] Команды не изменили отслеживаемых файлов: команда `git diff | diff - .practice/a-teams.diff` не выводит различий.
- [ ] Самопроверка (не тестируется): отчет о дискуссии объясняет, какие теории были опровергнуты и почему, и вы остановили каждого участника по имени.

Если лог участников отсутствует, команда не была создана: запустите ведущую сессию по шагу 4 и явно запросите формирование agent team. Если `git diff` отличается от сохраненного файла, ведущий или участник изменили файлы: проверьте строку `tools:` в определениях. По завершении сохраните изменение либо откатите временную правку через `git restore <file>`.

## Watch out

- Участники команды стартуют в вашем текущем режиме разрешений (за исключением `dontAsk`), а при запуске с флагом `--dangerously-skip-permissions` каждый участник также получает эти права. Их запросы разрешений отображаются в вашей сессии ([Permissions](https://code.claude.com/docs/en/agent-teams#permissions)). При включенных командах агентов обращение к субагенту по имени может породить участника команды: данный факультатив активирует команды только для сессий с флагом `--settings`, не затрагивая настройки пользователя или проекта ([Enable agent teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)).
- Каждый активный участник продолжает расходовать токены до своего завершения или окончания сессии. Держите команды небольшими, фиксируйте модель Sonnet в определениях, формулируйте четкие промпты и останавливайте участников по завершении задач ([Agent team token costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)).
- Одновременное редактирование одного файла двумя участниками приводит к перезаписи правок: выделяйте каждому участнику отдельные файлы ([Avoid file conflicts](https://code.claude.com/docs/en/agent-teams#avoid-file-conflicts)). Команды `/resume` и `/rewind` не восстанавливают встроенных участников: после возобновления сессии попросите ведущего породить новых участников ([Limitations](https://code.claude.com/docs/en/agent-teams#limitations)).

## Go further

- [Контроль качества через хуки](https://code.claude.com/docs/en/agent-teams#enforce-quality-gates-with-hooks): хуки `TeammateIdle`, `TaskCreated` и `TaskCompleted` с кодом завершения 2 удерживают участника за работой или блокируют создание/завершение задачи до выполнения ваших условий.
- [Планирование перед реализацией](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing): режим планирования только для чтения для команд, модифицирующих код.
- [Выбор режима отображения](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode): запуск каждого участника в отдельной панели tmux или iTerm2 вместо единого интерфейса in-process.

---

<sub>Sources: [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams) · [Run agents in parallel](https://code.claude.com/docs/en/agents) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [Hooks reference](https://code.claude.com/docs/en/hooks) · [Tools reference](https://code.claude.com/docs/en/tools-reference) · [All settings](https://code.claude.com/docs/en/settings-reference) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) · [Manage costs effectively](https://code.claude.com/docs/en/costs)</sub>

<sub>Back to the [Advanced index](../README.md) · Topic: [Субагенты и параллельная работа](../../topics/subagents-and-parallel-work.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-agent-teams)</sub>
