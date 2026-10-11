<a id="claude-code-everything-you-need-to-know-"></a>
# Claude Code: Всё, что вам нужно знать (Everything You Need to Know) <img src="assets/brand/claude-jumping.svg" width="44" height="40" alt="Animated Claude" align="right" />

**Освойте Claude Code на трёх уровнях: от первой интерактивной сессии до автономных мультиагентных запусков.** <details style="display:inline"><summary><code>[en]</code></summary>Learn Claude Code in three levels, from your first session to unattended multi-agent runs.</details>

Обучение построено на практике: каждый основной урок и выпускное задание (capstone) завершаются проверкой, которую вы запускаете в своей учебной копии репозитория, а Урок 1 для начинающих проходит пошагово в терминальном приложении бок о бок с Claude Code.\
**[Попробовать прямо сейчас](#try-it-now)** · [Выбрать свой уровень](#pick-your-level) · [Все страницы руководства](#everything-in-the-guide)

[![Verified against Claude Code v2.1.285 (stable)](https://img.shields.io/badge/verified-v2.1.285_stable-2e6e57)](CHANGELOG.md#v202610---2026-10-05) [![Edition release date](https://img.shields.io/github/release-date/wesammustafa/Claude-Code-Everything-You-Need-to-Know?label=edition&color=2e6e57)](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/releases/latest) [![License: MIT](https://img.shields.io/badge/license-MIT-5b6470)](LICENSE) [![Mentioned in Awesome Claude Code](https://awesome.re/mentioned-badge.svg)](https://github.com/hesreallyhim/awesome-claude-code)

<p align="center">
  <!-- Captured with Claude Code v2.1.285 (stable) on 2026-10-10. -->
  <img src="assets/readme/learn-step-turns-green.png" width="640" alt="The app noticed the saved file on its own: all three commands are ticked, the step's box turned green, and it quotes the version line it found in .practice/version.txt." />
  <br />
  <sub>Обучающее приложение, <code>npm run learn</code>, Beginner урок 1, шаг 3: вы вводите команды во втором терминале, а шаг сам подсвечивается зелёным при выполнении.</sub>
</p>

<details>
<summary>Тот же экран в виде текста (The same screen as text)</summary>

```text
B1  Install, sign in and look around                 GUIDED   step 3 of 12

in your other terminal   ✔ DONE

Save the version
Go to your practice copy and save that output for the check:

  $ cd /Users/Shared/claude-code-practice           ✔
  $ mkdir -p .practice                              ✔
  $ claude --version > .practice/version.txt        ✔

Done when .practice/version.txt holds a line such as X.Y.Z (Claude Code).

Step 3 done!
.practice/version.txt holds 2.1.285 (Claude Code)
3 steps this session

enter  next: Start, sign in, trust    b back    m map    q quit
```

</details>

<a id="try-it-now"></a>
## Попробовать прямо сейчас (Try it now)

1. Создайте свою учебную копию: откройте [claude-code-practice](https://github.com/wesammustafa/claude-code-practice), выберите **Use this template** > **Create a new repository** и клонируйте репозиторий. Или откройте копию в Codespace, где Node.js и Claude Code уже предустановлены.
2. В созданной копии запустите обучающее приложение. Для работы требуются Node.js LTS и git, дополнительных установок через npm не требуется:

   ```bash
   npm run learn
   ```

3. Откройте второй терминал в этой же директории. Приложение подскажет, что именно вводить (начиная с установки Claude Code), и будет проверять ваши действия по ходу выполнения:

   ```text
   this window                     your other terminal
   npm run learn                   where you type commands
   one step at a time,     --->    and run Claude Code
   checks your work                (this app checks it)
   ```

Приложение проводит вас по Уроку 1 уровня Beginner шаг за шагом; для остальных уроков уровня Beginner оно отслеживает выполнение проверок, пока вы следуете инструкциям страницы. В обучающей утилите нет AI, и она не вносит изменений в ваш код: она считывает файлы, вызывает `git status` и проверяющий скрипт урока, записывая только свой собственный файл прогресса.

Предпочитаете просто читать? Каждый урок самодостаточен и может выполняться прямо со страницы. В конце каждого ключевого урока есть проверка, запускаемая из вашей тренировочной копии (например, `npm run check -- b-1`): каждый пункт выводит статус `PASS` или `FAIL` с подсказкой по исправлению.

<a id="help-that-fades-as-you-learn"></a>
## Подсказки, уменьшающиеся по мере обучения (Help that fades as you learn)

Объём подсказок сокращается по мере роста ваших навыков: режим GUIDED подробно расписывает каждую команду, режим HINTED формулирует цель и даёт подсказку по нажатию `h`, а режим CHALLENGE оставляет только саму цель.

<table>
  <tr>
    <td width="50%" valign="top"><img src="assets/readme/learn-help-guided.png" width="380" alt="At step 1 the app names the window to type in and shows the exact install command." /></td>
    <td width="50%" valign="top"><img src="assets/readme/learn-help-challenge.png" width="380" alt="By step 11 the app gives only the goal and the lesson's rule, and reports from git status that nothing in the copy has changed yet." /></td>
  </tr>
  <tr>
    <td><b>Шаг 1 из 12, GUIDED:</b> точная команда и окно, в котором её нужно ввести.</td>
    <td><b>Шаг 11 из 12, CHALLENGE:</b> только цель задания, пока приложение наблюдает за изменениями.</td>
  </tr>
</table>

<!-- Both captured with Claude Code v2.1.285 (stable) on 2026-10-10. -->
<sub>Beginner урок 1, шаги 1 и 11.</sub>

<a id="pick-your-level"></a><a id="-choose-your-path"></a>
## Выберите свой уровень (Pick your level)

<a id="beginner"></a>
### Beginner (Начальный)
**Подходит вам, если** вы новичок в Claude Code и уверенно чувствуете себя в терминале и с git.\
Вы безопасно проведёте одно изменение от запроса до коммита и добавите память проекту. Около 2.5 часов: 5 уроков, выпускное задание и 3 факультатива.\
**[Начать Beginner](docs/beginner/README.md)**

<a id="intermediate"></a>
### Intermediate (Средний)
**Вы готовы, если** умеете планировать изменения в режиме plan mode, анализировать diff и коммитить результат, а также удерживать фокус в длинных сессиях.\
Вы настроите Claude Code под свой проект с помощью skills (навыков), hooks (хуков), subagents (субагентов), MCP и плагинов. Около 3 часов: 8 уроков, выпускное задание и 4 факультатива.\
**[Начать Intermediate](docs/intermediate/README.md)**

<a id="advanced"></a>
### Advanced (Продвинутый)
**Вы готовы, если** умеете писать проектные навыки, контролировать правила с помощью хуков, делегировать задачи субагентам и подключать MCP-серверы.\
Вы освоите параллельные запуски Claude Code, автономную работу в CI и командные сценарии в рамках заданных лимитов. Около 3 часов 20 минут (плюс время запусков): 7 уроков, выпускное задание и 5 факультативов.\
**[Начать Advanced](docs/advanced/README.md)**

<a id="install"></a>
## Установка (Install)

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

macOS, Linux и WSL. Для Windows и альтернативных способов установки обратитесь к [официальной странице настройки](https://code.claude.com/docs/en/setup). Для Claude Code требуется платный тариф Claude (Pro, Max, Team или Enterprise), аккаунт Claude Console либо доступ через поддерживаемого облачного провайдера ([Quickstart](https://code.claude.com/docs/en/quickstart)).

<a id="how-the-guide-stays-current"></a>
## Как руководство поддерживается в актуальном состоянии (How the guide stays current)

- Каждая страница урока снабжена штампом (Stamp): версия Claude Code `stable`, по которой проверялись утверждения, и дата проверки ([The stamp](CONTRIBUTING.md#the-stamp)).
- Сами скрипты проверки тоже тестируются: в шаблоне практики CI подтверждает, что проверки падают на ветке `main` (где задания ещё не выполнены) и успешно проходят на эталонных решениях ([Assert checks](https://github.com/wesammustafa/claude-code-practice/blob/main/.github/workflows/assert-checks.yml)).
- Автоматические проверки работают по расписанию между редакциями: внешние ссылки и новые стабильные (`stable`) релизы проверяются еженедельно, маркеры срока годности фактов: ежедневно, а совпадение цитат в обучающем приложении: еженедельно. По каждой проблеме автоматически создаётся трекинг-issue ([Maintainer routine](CONTRIBUTING.md#maintainer-routine)).
- Примеры неактивны (inert), пока вы явно не скопируете их в рабочий проект, и следуют [контракту безопасности](CONTRIBUTING.md#adding-a-skill): работают строго локально (за исключением зафиксированного сервера в уроке по MCP), соблюдают принцип наименьших привилегий и содержат смоук-тесты, запускаемые без участия Claude.

<a id="everything-in-the-guide"></a><a id="-whats-inside"></a><a id="-reading-tips"></a>
## Все разделы руководства (Everything in the guide)

<details>
<summary>Все страницы по уровням (Every page, by level)</summary>

**Beginner:** [Оглавление уровня](docs/beginner/README.md) · [Установка, вход и первый запуск](docs/beginner/01-install-and-look-around.md) · [Режимы разрешений и режим планирования](docs/beginner/02-permission-modes-and-plan-mode.md) · [Первое изменение: от запроса до коммита](docs/beginner/03-first-change.md) · [Управление фокусом и контекстом сессии](docs/beginner/04-keep-a-session-on-track.md) · [Память проекта с CLAUDE.md](docs/beginner/05-project-memory.md) · [Выпускное задание](docs/beginner/capstone.md)

**Beginner Electives (Факультативы):** [Интеграция с редакторами и IDE](docs/beginner/electives/ide-extensions.md) · [Скриншоты и изображения](docs/beginner/electives/screenshots-and-images.md) · [Встроенные обучающие инструменты](docs/beginner/electives/built-in-teachers.md)

**Intermediate:** [Оглавление уровня](docs/intermediate/README.md) · [Превращение повторяющегося сценария в навык](docs/intermediate/01-first-skill.md) · [Организация проектной памяти](docs/intermediate/02-organize-memory.md) · [Выбор модели и уровня усилий](docs/intermediate/03-model-and-effort.md) · [Разрешения, области настроек и песочница](docs/intermediate/04-permissions-and-sandbox.md) · [Ограничение правил с помощью хуков](docs/intermediate/05-hooks.md) · [Делегирование задач субагентам](docs/intermediate/06-subagents.md) · [Подключение внешних инструментов через MCP](docs/intermediate/07-mcp.md) · [Установка и управление плагинами](docs/intermediate/08-plugins.md) · [Выпускное задание](docs/intermediate/capstone.md)

**Intermediate Electives (Факультативы):** [Паттерны навыков](docs/intermediate/electives/skill-patterns.md) · [Claude в браузере Chrome](docs/intermediate/electives/claude-in-chrome.md) · [Строка статуса](docs/intermediate/electives/status-line.md) · [Быстрый режим (Fast mode)](docs/intermediate/electives/fast-mode.md)

**Advanced:** [Оглавление уровня](docs/advanced/README.md) · [Параллельные сессии с git worktrees](docs/advanced/01-parallel-sessions.md) · [Выбор шаблона оркестрации](docs/advanced/02-orchestration-patterns.md) · [Динамические рабочие процессы](docs/advanced/03-dynamic-workflows.md) · [Скриптинг Claude Code через `claude -p`](docs/advanced/04-headless.md) · [Интеграция с GitHub Actions](docs/advanced/05-github-actions.md) · [Командное использование настроек](docs/advanced/06-share-your-setup.md) · [Ограничения автономных запусков](docs/advanced/07-bounded-runs.md) · [Выпускное задание](docs/advanced/capstone.md) · [За рамками руководства](docs/advanced/beyond-this-guide.md)

**Advanced Electives (Факультативы):** [Команды агентов (Agent teams)](docs/advanced/electives/agent-teams.md) · [Просмотр агентов (Agent view)](docs/advanced/electives/agent-view.md) · [Команды `/goal`, `/loop` и рутины](docs/advanced/electives/goal-loop-routines.md) · [Тестирование и публикация плагина](docs/advanced/electives/publish-a-plugin.md) · [Контейнеры разработки (Dev containers)](docs/advanced/electives/dev-containers.md)

**Topics (Темы):** [Все темы](docs/topics/README.md) · [Разрешения и безопасность](docs/topics/permissions-and-safety.md) · [Память и контекст](docs/topics/memory-and-context.md) · [Модели, усилия и стоимость](docs/topics/models-effort-and-cost.md) · [Навыки (Skills)](docs/topics/skills.md) · [Хуки (Hooks)](docs/topics/hooks.md) · [Субагенты и параллельная работа](docs/topics/subagents-and-parallel-work.md) · [MCP](docs/topics/mcp.md) · [Плагины](docs/topics/plugins.md) · [Автоматизация](docs/topics/automation.md)

**Reference (Справочник):** [Оглавление справочника](docs/reference/README.md) · [Модели и параметры усилий](docs/reference/models.md) · [Команды по уровням](docs/reference/commands.md) · [Карта возможностей](docs/reference/feature-map.md) · [Материалы для углубления](docs/reference/further-reading.md) · [Глоссарий](docs/reference/glossary.md) · [История изменений](docs/reference/changelog.md)

**Examples (Примеры):** [Как использовать примеры](examples/README.md)

</details>

---

Нашли неточность или устаревший факт? [Сообщите об этом с ссылкой на первоисточник](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=stale-content.yml). Возникли трудности с уроком? Воспользуйтесь ссылкой обратной связи внизу соответствующего урока. Claude Code является продуктом Anthropic; данное руководство представляет собой независимый проект сообщества, не аффилированный с компанией Anthropic.

Если руководство оказалось полезным, звезда на GitHub поможет другим разработчикам найти его.

[Лицензия MIT](LICENSE) · [Вклад в проект (Contributing)](CONTRIBUTING.md) · [Журнал изменений (Changelog)](CHANGELOG.md) · [Безопасность (Security)](SECURITY.md)

*Талисман: [`assets/brand/claude-jumping.svg`](assets/brand/claude-jumping.svg) взят из [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), Copyright (c) 2025-2026 Shayan Rais, используется по лицензии [MIT](https://github.com/shanraisshan/claude-code-best-practice/blob/main/LICENSE).*

<a id="links-from-the-previous-edition"></a>
### Ссылки из предыдущей редакции (Links from the previous edition)

The previous edition's README sections moved. If a link brought you here, find its section below:

- <a id="sdlc"></a><a id="what-are-llms-and-how-do-they-differ-from-ai-tools-like-claude-code"></a><a id="what-is-claude-code"></a><a id="claude-code-setup"></a><a id="1-install"></a><a id="2-authenticate"></a><a id="3-run-your-first-prompt"></a>Moved to [Install, sign in and look around](docs/beginner/01-install-and-look-around.md).
- <a id="prompt-engineering-deep-dive"></a><a id="1-explore--plan--code--commit"></a><a id="2-test-driven-workflow-write-tests--code--commit"></a>Moved to [Your first change, from request to commit](docs/beginner/03-first-change.md).
- <a id="4-optional-generate-a-claudemd"></a>Moved to [Project memory with CLAUDE.md](docs/beginner/05-project-memory.md).
- <a id="3-visual-iteration-code--screenshot--iterate--commit"></a>Moved to [Give Claude a screenshot](docs/beginner/electives/screenshots-and-images.md).
- <a id="claude-opus-46-the-latest-powerhouse"></a><a id="claude-opus-47-the-latest-flagship"></a><a id="the-claude-5-era-todays-model-lineup"></a>Moved to [Models and effort](docs/reference/models.md).
- <a id="effort-levels"></a><a id="4-effort-levels--how-hard-claude-thinks"></a>Moved to [Pick the model and effort](docs/intermediate/03-model-and-effort.md).
- <a id="claude-commands"></a><a id="built-in-slash-commands"></a><a id="day-1-essentials"></a>Moved to [Commands by level](docs/reference/commands.md).
- <a id="custom-slash-commands"></a><a id="claude-skills"></a><a id="your-first-skill-in-3-minutes"></a><a id="want-more-depth"></a><a id="what-are-skills"></a><a id="built-in-vs-custom-skills"></a><a id="skills-faq"></a><a id="creating-custom-skills"></a>Moved to [Turn a repeated workflow into a skill](docs/intermediate/01-first-skill.md).
- <a id="available-skills-reference"></a><a id="using-skills-in-workflow"></a>Moved to [Turn a repeated workflow into a skill](docs/intermediate/01-first-skill.md), which shows how to run a skill. The catalog of this repo's own skills and its workflow recipes were removed in edition v2026.10: see the [CHANGELOG](CHANGELOG.md#removed-live-config).
- <a id="troubleshooting-skills"></a><a id="skills-best-practices"></a>Moved to [Skill patterns](docs/intermediate/electives/skill-patterns.md).
- <a id="hooks"></a><a id="setting-up-claude-hooks"></a><a id="setting-up-hooks"></a><a id="hook-events"></a><a id="hook-input"></a><a id="hook-output"></a><a id="security-considerations"></a><a id="hook-execution-details-and-debugging"></a><a id="execution--debugging"></a>Moved to [Enforce a rule with a hook](docs/intermediate/05-hooks.md).
- <a id="2-general-purpose-subagents--when-one-claude-isnt-enough"></a><a id="3-specialized-subagents--drop-in-role-prompts"></a>Moved to [Delegate to a custom subagent](docs/intermediate/06-subagents.md). The drop-in role prompts were removed in edition v2026.10: see the [CHANGELOG](CHANGELOG.md#removed-listings).
- <a id="ai-agents"></a><a id="running-agents-in-parallel"></a><a id="subagents--running-agents-in-parallel"></a><a id="orchestrating-specialists-from-the-main-session"></a>Moved to [Choose an orchestration pattern](docs/advanced/02-orchestration-patterns.md).
- <a id="1-git-worktrees--parallel-branches-parallel-sessions"></a>Moved to [Parallel sessions with worktrees](docs/advanced/01-parallel-sessions.md).
- <a id="agent-teams-experimental---2026"></a><a id="agent-teams-experimental"></a><a id="enable-it"></a><a id="the-example-that-justifies-the-cost"></a><a id="staff-a-team-with-the-role-prompts-you-already-have"></a><a id="three-things-that-catch-people-out"></a><a id="monitoring-and-the-naming-trap"></a><a id="best-practices"></a>Moved to [Agent teams](docs/advanced/electives/agent-teams.md).
- <a id="dynamic-workflows"></a><a id="try-it-in-2-minutes--no-script-required"></a><a id="starting-your-own"></a>Moved to [Run and save a dynamic workflow](docs/advanced/03-dynamic-workflows.md).
- <a id="beyond-one-terminal--the-2026-automation-surface"></a>Moved to [Feature map](docs/reference/feature-map.md).
- <a id="model-context-protocol-mcp"></a><a id="the-nm-problem-mcp-solves"></a><a id="three-pillars"></a><a id="the-mcp-registry--self-discovering-agents"></a><a id="the-mcp-ecosystem-today"></a>Moved to [Connect a tool with MCP](docs/intermediate/07-mcp.md).
- <a id="fast-mode"></a><a id="fast-mode-"></a>Moved to [Fast mode](docs/intermediate/electives/fast-mode.md).
- <a id="beyond-your-own-skills--the-ecosystem"></a>Moved to [Install and manage plugins](docs/intermediate/08-plugins.md). The skills ecosystem catalog was removed in edition v2026.10: see the [CHANGELOG](CHANGELOG.md#removed-listings).
- <a id="featured-mcp-servers"></a><a id="more-mcp-servers-worth-knowing"></a>Moved to [Connect a tool with MCP](docs/intermediate/07-mcp.md). The MCP server tables were removed in edition v2026.10: see the [CHANGELOG](CHANGELOG.md#removed-listings).
- <a id="super-claude-framework"></a><a id="the-bmad-method--ai-agent-framework"></a>Removed in edition v2026.10: listings no lesson uses. See the [CHANGELOG](CHANGELOG.md#removed-listings).
- <a id="updates--deprecations-february-2026"></a><a id="updates--deprecations"></a><a id="updates--deprecations-as-of-july-2026"></a>Removed in edition v2026.10: the guide's summaries of Claude Code releases. See the [CHANGELOG](CHANGELOG.md#removed-changelog-mirror).
- <a id="faq"></a>Moved to [FAQ](docs/reference/faq.md), which says where each answer went.
- <a id="references"></a>Moved to [Further learning](docs/reference/further-reading.md).
- <a id="steal-this-setup"></a><a id="5-bonus-steal-this-repos-setup"></a>Moved to [Share your setup with a team](docs/advanced/06-share-your-setup.md).
- <a id="-when-to-use-what"></a>Moved to [When to use what](docs/intermediate/README.md#when-to-use-what).

