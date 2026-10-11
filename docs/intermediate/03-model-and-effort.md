# Pick the model and effort

<sub>**Intermediate** · Lesson 3 of 8 · about 15 minutes · Needs: [Turn a repeated workflow into a skill](01-first-skill.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого урока вы сможете подбирать модель и уровень мыслительных усилий (effort level) под конкретную задачу, устанавливать их для всей сессии или для отдельного навыка, а также определять по неверному результату, требуется ли повысить effort или переключиться на более крупную модель. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can pick a model and an effort level for a task, set them for a session or for one skill, and tell from a wrong result whether to raise the effort or pick a larger model.</details>

## You need

- Репозиторий, где вы выполняли [урок 1](01-first-skill.md), с созданными навыками, и ваша копия учебного шаблона с Node.js LTS (проверка запускается из неё).
- Расход токенов: минимальный (light). Более крупная модель дороже за токен, а высокий уровень effort обычно генерирует больше токенов, поэтому оба параметра повышают расход лимитов.

## The idea

Два независимых параметра определяют качество и характер ответа ([Choosing a Claude model and effort level in Claude Code](https://claude.com/blog/claude-model-and-effort-level-in-claude-code)): <details style="display:inline"><summary><code>[en]</code></summary>Two separate choices shape an answer</details>

- **Модель (model)** задаёт базовый интеллектуальный потенциал для решения задачи.
- **Уровень усилий (effort)** определяет, какой объём работы Claude совершает на каждом шаге: сколько файлов читает, насколько тщательно верифицирует гипотезы и как далеко продвигается перед запросом обратной связи ([Model configuration](https://code.claude.com/docs/en/model-config#adjust-effort-level)).

Рекомендация Anthropic - начинать с уровня усилий по умолчанию для выбранной модели для большинства задач. Если получен неверный результат, разберитесь в причине перед тем, как менять настройки:

| Что произошло или какова задача | Что изменить |
|---|---|
| Claude не хватило контекста: неточный промпт, отсутствие нужного инструмента или навыка | Контекст: промпт, `CLAUDE.md`, навык |
| Claude приложил недостаточно усилий: пропустил файл или не запустил тесты | Повысить уровень усилий (effort) |
| Контекст был полным, попытка сделана, но ответ неверен: тонкий баг, незнакомый домен, архитектурный выбор | Выбрать более мощную модель, например `opus` |
| Рутинная работа: механические правки, чётко сформулированные изменения | Меньшая модель, например `sonnet`, и низкий effort ([Choose an effort level](https://code.claude.com/docs/en/model-config#choose-an-effort-level)) |
| Глубокое планирование с последующей рутиной | `opusplan`: Opus в режиме планирования, затем Sonnet для исполнения |

Область применения определяет способ настройки. Страница [Models and effort](../reference/models.md) содержит полный список алиасов и уровней:

| Область (Scope) | Модель | Effort |
|---|---|---|
| Текущая сессия | `/model`, затем клавиша `s` на строке | `/effort`, затем клавиша `s` на шкале |
| Значение по умолчанию | `/model <alias>` | `/effort <level>` |
| Один запрос | - | ключевое слово `ultrathink` в промпте запрашивает глубокие рассуждения; уровень остаётся прежним |
| Отдельный навык или субагент | поле `model:` во frontmatter | поле `effort:` во frontmatter |

## Worked example

1. Запустите `claude` в вашем репозитории. Выполните `/status`: команда отобразит текущую модель. Выполните `/effort status`: отобразится действующий уровень усилий ([Commands](https://code.claude.com/docs/en/commands)).
2. Переключите модель только для текущей сессии: выполните `/model`, перейдите к строке Sonnet и нажмите `s`. Теперь `/status` показывает Sonnet, а значение по умолчанию для будущих новых сессий не изменилось ([Setting your model](https://code.claude.com/docs/en/model-config#setting-your-model)).
3. Запросите глубокие рассуждения для одного запроса без изменения постоянного уровня effort:

   ```text
   ultrathink: which links could linkcheck report as broken although their file exists?
   ```

   Claude Code распознаёт токен `ultrathink` и активирует расширенные рассуждения только для этого хода диалога. Обычные фразы вроде "think hard" воспринимаются просто как текст ([Use ultrathink](https://code.claude.com/docs/en/model-config#use-ultrathink-for-one-off-deep-reasoning)).
4. Задайте уровень усилий для отдельного навыка. Откройте навык `tdd` из урока 1, `.claude/skills/tdd/SKILL.md`, и добавьте строку `effort: high` в его frontmatter:

   ```markdown
   ---
   name: tdd
   description: Builds a feature test-first.
   effort: high
   ---
   ```

   Теперь данный навык выполняется с уровнем `high`, пока активен, переопределяя уровень сессии. Значение, заданное переменной окружения `CLAUDE_CODE_EFFORT_LEVEL`, и системные ограничения по-прежнему имеют высший приоритет ([Set the effort level](https://code.claude.com/docs/en/model-config#set-the-effort-level)).

## Your turn

Настройте рутинную задачу на минимальные достаточные модель и уровень усилий.

1. Сохраните следующий навык как `.claude/skills/changelog/SKILL.md`:

   ```markdown
   ---
   name: changelog
   description: Adds a one-line entry for the staged changes to CHANGELOG.md. Use when the user asks for a changelog entry.
   model: TODO
   effort: TODO
   disable-model-invocation: true
   ---

   Read the staged changes with `git diff --staged`. Add one list item (`- `) under `## Unreleased` in CHANGELOG.md that says what changed for someone using the tool, creating the file and the heading if they don't exist. Change no other file.
   ```

2. Замените обе пометки `TODO`, руководствуясь первой таблицей урока. В режиме auto mode используйте `sonnet`: auto mode не поддерживает Haiku, поэтому навык с `haiku` сохранит текущую модель сессии ([Eliminate permission prompts with auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)).
3. Добавьте в индекс git небольшое изменение (`git add`), запустите `/changelog` и подтвердите правку. Если Claude Code сообщает, что команда неизвестна, сначала выполните `/reload-skills`.
4. Закоммитьте сделанное изменение, обновлённый `CHANGELOG.md` и сам навык одним коммитом.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный навык выбирает компактную модель и уровень усилий low или medium: `git show HEAD:.claude/skills/changelog/SKILL.md | grep -E '^(model|effort):'` отображает обе строки.
- [ ] Закоммиченный `CHANGELOG.md` содержит пункт (`- `) под заголовком `## Unreleased`: `git show HEAD:CHANGELOG.md` показывает его.
- [ ] Self-check (not tested): для точного переименования, для новой фичи с понятной структурой и для бага, в котором Claude ошибается при полном контексте, вы можете обосновать выбор модели и уровня усилий.

Запустите `npm run check -- i-3 --dir <your repo>` из учебной копии, либо `npm run check -- i-3` в самой копии.

Если проверка сообщает, что модель больше, чем требуется для задачи, укажите `model: sonnet`: правка с однозначным описанием относится к рутинным операциям. Если в файле нет записей, добавьте изменения в staging (`git add`) перед запуском `/changelog`, поскольку навык анализирует только проиндексированный diff.

## Watch out

- Команды `/model <alias>` и `/effort <level>` сохраняют выбор как значение по умолчанию для будущих новых сессий. Чтобы изменить параметр только для текущей сессии, нажмите клавишу `s` в меню выбора или на шкале.
- Переключение модели посреди сессии (а для большинства моделей и смена effort) приводит к тому, что следующий запрос перечитывает всю историю диалога без попадания в кэш; при прогретом кэше Claude Code запросит подтверждение ([How Claude Code uses prompt caching](https://code.claude.com/docs/en/prompt-caching#switching-models)).
- Уровень `max`, заданный через `/effort`, действует только до конца текущей сессии и может приводить к избыточным рассуждениям: протестируйте поведение перед постоянным использованием ([Choose an effort level](https://code.claude.com/docs/en/model-config#choose-an-effort-level)).

## Go further

- [Choosing a Claude model and effort level in Claude Code](https://claude.com/blog/claude-model-and-effort-level-in-claude-code): руководство Anthropic с методикой "try or know", рассмотренной в этом уроке.
- [Models and effort](../reference/models.md): алиасы моделей, уровни усилий и способы их конфигурирования.
- [Model configuration](https://code.claude.com/docs/en/model-config): значения по умолчанию для аккаунтов и облачных провайдеров, а также политики организаций.

---

<sub>Sources: [Model configuration](https://code.claude.com/docs/en/model-config) · [Commands](https://code.claude.com/docs/en/commands) · [How Claude Code uses prompt caching](https://code.claude.com/docs/en/prompt-caching) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Choosing a Claude model and effort level in Claude Code](https://claude.com/blog/claude-model-and-effort-level-in-claude-code)</sub>

<sub>← [Organize project memory and see what loaded](02-organize-memory.md) · [Intermediate index](README.md) · [Permissions, settings scopes and the sandbox](04-permissions-and-sandbox.md) → · Topic: [Models, effort and cost](../topics/models-effort-and-cost.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-3)</sub>
