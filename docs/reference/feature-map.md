<a id="feature-map"></a>
# Карта возможностей

<sub>Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Полная карта актуальных возможностей и рабочих окружений Claude Code, с которыми сталкивается разработчик, с точными статусами стабильности из официальной документации и списком доступных тарифов и провайдеров. Каждый пункт ссылается на страницу официальной документации, а также на уроки и факультативы этого руководства. Недавние изменения отражены в официальном журнале [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) и еженедельнике [What's new](https://code.claude.com/docs/en/whats-new).

Как читать таблицу:

- **Статус (Label)** цитирует формулировку официальной документации (beta, research preview или experimental). Прочерк означает отсутствие специальной пометки (общедоступная функциональность).
- **Кому доступно (Who gets it)** основано на [Feature availability](https://code.claude.com/docs/en/feature-availability) или на профильной странице функции. Формулировка «Любой провайдер» означает подписку Claude, Claude Console, Claude Platform on AWS, Amazon Bedrock, Google Cloud Agent Platform и Microsoft Foundry: CLI и все локальные механизмы работают на всех них одинаково.
- В руководстве не приводятся цены и лимиты запросов; актуальные данные доступны на [claude.com/pricing](https://claude.com/pricing).

<a id="where-you-run-it"></a>
## Среды выполнения

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Терминал](https://code.claude.com/docs/en/quickstart); рассматривается в [Установка, вход и знакомство с интерфейсом](../beginner/01-install-and-look-around.md) | – | Любой провайдер |
| [VS Code](https://code.claude.com/docs/en/vs-code); рассматривается в [Расширения для IDE](../beginner/electives/ide-extensions.md) | – | Любой провайдер |
| [JetBrains IDEs](https://code.claude.com/docs/en/jetbrains); рассматривается в [Расширения для IDE](../beginner/electives/ide-extensions.md) | На странице нет отдельной фразы, но в пути настроек указано "Claude Code [Beta]" | Любой провайдер |
| [Приложение Desktop](https://code.claude.com/docs/en/desktop) | На Linux: "Linux support for the Claude desktop app is in beta." ([Linux](https://code.claude.com/docs/en/desktop-linux)) | Подписка Claude; на Bedrock, Agent Platform и Foundry через Claude Desktop для этих провайдеров |
| [Облачные сессии в браузере](https://code.claude.com/docs/en/claude-code-on-the-web) | – | Pro, Max и Team; Enterprise с премиальной лицензией или Chat + Claude Code |
| [Мобильное приложение](https://code.claude.com/docs/en/mobile) | – | Подписка Claude |
| [Remote Control](https://code.claude.com/docs/en/remote-control) | – | Pro и Max; Team и Enterprise после включения администратором |
| [Claude Code в Slack](https://code.claude.com/docs/en/slack) | – | Pro, Max, Team или Enterprise с доступом к Claude Code; на Team и Enterprise выводится из эксплуатации в пользу Claude Tag |
| [Claude Tag](https://claude.com/docs/claude-tag/overview) | Отмечен как "Public Beta" на странице сервиса | Team и Enterprise на собственной платформе Anthropic |
| [Claude в Chrome](https://code.claude.com/docs/en/chrome); рассматривается в [Claude в Chrome](../intermediate/electives/claude-in-chrome.md) | – | Pro, Max, Team или Enterprise |
| [Computer use](https://code.claude.com/docs/en/computer-use) | "Computer use is a research preview on macOS that requires a Pro or Max plan." | Pro и Max |
| [Dispatch](https://code.claude.com/docs/en/desktop#sessions-from-dispatch) | – | Pro и Max |
| [Голосовой ввод](https://code.claude.com/docs/en/voice-dictation) | – | Подписка Claude |

<a id="in-a-session"></a>
## Возможности внутри сессии

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Режим планирования (Plan mode)](https://code.claude.com/docs/en/permission-modes); рассматривается в [Режимы разрешений и режим планирования](../beginner/02-permission-modes-and-plan-mode.md) | – | Любой провайдер |
| [Автоматический режим (Auto mode)](https://code.claude.com/docs/en/permission-modes); рассматривается в [Режимы разрешений и режим планирования](../beginner/02-permission-modes-and-plan-mode.md) | – | Подписка Claude, Console и Claude Platform on AWS; на Bedrock, Agent Platform и Foundry только с новыми моделями. Администраторы Team и Enterprise могут отключить |
| [Контрольные точки и откат (Checkpointing / rewind)](https://code.claude.com/docs/en/checkpointing); рассматривается в [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md) | – | Любой провайдер |
| [Память CLAUDE.md](https://code.claude.com/docs/en/memory); рассматривается в [Память проекта с CLAUDE.md](../beginner/05-project-memory.md) и [Организация памяти проекта и проверка загрузки](../intermediate/02-organize-memory.md) | – | Любой провайдер |
| [Песочница (Sandbox)](https://code.claude.com/docs/en/sandboxing); рассматривается в [Разрешения, области видимости настроек и песочница](../intermediate/04-permissions-and-sandbox.md) и [Ограничения для автономных запусков](../advanced/07-bounded-runs.md) | – | Любой провайдер |
| [Стили вывода (Output styles)](https://code.claude.com/docs/en/output-styles); рассматриваются во [Встроенные обучающие инструменты](../beginner/electives/built-in-teachers.md) | – | Любой провайдер |
| [Строка состояния](https://code.claude.com/docs/en/statusline); рассматривается в [Строка состояния](../intermediate/electives/status-line.md) | – | Любой провайдер |
| [`/goal`](https://code.claude.com/docs/en/goal); рассматривается в [`/goal`, `/loop` и рутины](../advanced/electives/goal-loop-routines.md) | – | Любой провайдер |
| [`/powerup`](https://code.claude.com/docs/en/commands); рассматривается во [Встроенные обучающие инструменты](../beginner/electives/built-in-teachers.md) | – | В документации ограничений нет |
| [Быстрый режим (Fast mode)](https://code.claude.com/docs/en/fast-mode); рассматривается в [Быстрый режим](../intermediate/electives/fast-mode.md) | "Fast mode is in research preview." | Подписка Claude за кредиты использования (владелец включает для Team и Enterprise) и организации в Console с доступом; недоступен на Claude Platform on AWS, Bedrock, Agent Platform и Foundry |
| [Advisor](https://code.claude.com/docs/en/advisor) | "The advisor tool is experimental and requires the Anthropic API." | Подписка Claude и Console |
| [Веб-поиск](https://code.claude.com/docs/en/tools-reference#websearch-tool-behavior) | – | Подписка Claude, Console и Claude Platform on AWS; отсутствует на Bedrock; ограничен на Agent Platform и Foundry |

<a id="extending-claude-code"></a>
## Расширение возможностей Claude Code

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Навыки (Skills)](https://code.claude.com/docs/en/skills); рассматриваются в [Превращение повторяющегося процесса в навык](../intermediate/01-first-skill.md) и [Паттерны разработки навыков](../intermediate/electives/skill-patterns.md) | – | Любой провайдер |
| [Хуки (Hooks)](https://code.claude.com/docs/en/hooks-guide); рассматриваются в [Применение правил с помощью хуков](../intermediate/05-hooks.md) | – | Любой провайдер |
| [Агентные хуки](https://code.claude.com/docs/en/hooks-guide#agent-based-hooks) | "Agent hooks are experimental." | Аналогично хукам |
| [Субагенты](https://code.claude.com/docs/en/sub-agents); рассматриваются в [Делегирование кастомному субагенту](../intermediate/06-subagents.md) и [Выбор паттерна оркестрации](../advanced/02-orchestration-patterns.md) | – | Любой провайдер |
| [MCP](https://code.claude.com/docs/en/mcp); рассматривается в [Подключение инструментов через MCP](../intermediate/07-mcp.md) | – | Любой провайдер; коннекторы claude.ai только при авторизации через подписку Claude |
| [Плагины и маркетплейсы](https://code.claude.com/docs/en/plugins/overview); рассматриваются в [Установка плагинов и управление ими](../intermediate/08-plugins.md), [Совместное использование настроек в команде](../advanced/06-share-your-setup.md) и [Тестирование и публикация плагина](../advanced/electives/publish-a-plugin.md) | – | Любой провайдер |
| [Каналы (Channels)](https://code.claude.com/docs/en/channels) | "Channels are in research preview." | Подписка Claude (Team и Enterprise после включения администратором) и Console; недоступно на Claude Platform on AWS, Bedrock, Agent Platform и Foundry |
| [Dev-контейнеры](https://code.claude.com/docs/en/devcontainer); рассматриваются в [Dev-контейнеры](../advanced/electives/dev-containers.md) | – | Любой провайдер |

<a id="parallel-and-unattended-work"></a>
## Параллельная и автономная работа

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Рабочие деревья (Worktrees)](https://code.claude.com/docs/en/worktrees); рассматриваются в [Параллельные сессии с worktree](../advanced/01-parallel-sessions.md) | – | Любой провайдер |
| [Динамические воркфлоу](https://code.claude.com/docs/en/workflows); рассматриваются в [Запуск и сохранение динамического воркфлоу](../advanced/03-dynamic-workflows.md) | – | Любой провайдер; на тарифе Pro включаются в `/config` |
| [Режим Agent view](https://code.claude.com/docs/en/agent-view); рассматривается в [Режим Agent view](../advanced/electives/agent-view.md) | "Agent view is in research preview." | Любой провайдер ([Settings and provider](https://code.claude.com/docs/en/agent-view#settings-and-provider)) |
| [Команды агентов (Agent teams)](https://code.claude.com/docs/en/agent-teams); рассматриваются в [Команды агентов](../advanced/electives/agent-teams.md) | "Agent teams are experimental and disabled by default." | Любой провайдер |
| [Межсессионные сообщения](https://code.claude.com/docs/en/cross-session-messaging) | – | Подписка Claude; у остальных провайдеров только между сессиями на одной локальной машине |
| [Проекты (Projects)](https://code.claude.com/docs/en/claude-projects) | "Projects are in public beta on Pro and Max plans and rolling out gradually …" | Pro и Max; пока недоступно на Team и Enterprise |
| [`/loop` и запуск запросов по расписанию](https://code.claude.com/docs/en/scheduled-tasks); рассматривается в [`/goal`, `/loop` и рутины](../advanced/electives/goal-loop-routines.md) | – | Любой провайдер |
| [Рутины (Routines)](https://code.claude.com/docs/en/routines); рассматриваются в [`/goal`, `/loop` и рутины](../advanced/electives/goal-loop-routines.md) | "Routines are in research preview." | Pro, Max, Team и Enterprise |
| [Вызов Claude Code из скриптов](https://code.claude.com/docs/en/headless); рассматривается в [Автоматизация Claude Code с помощью `claude -p`](../advanced/04-headless.md) и [Ограничения для автономных запусков](../advanced/07-bounded-runs.md) | – | Любой провайдер |
| [GitHub Actions](https://code.claude.com/docs/en/github-actions); рассматривается в [Интеграция с GitHub Actions](../advanced/05-github-actions.md) | – | Подписка Claude, Console, Bedrock, Agent Platform и Foundry; недоступно на Claude Platform on AWS |
| [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) | "Claude Code for GitLab CI/CD is currently in beta." | Любой провайдер, кроме Microsoft Foundry |

<a id="review-and-security"></a>
## Ревью и безопасность

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Code Review](https://code.claude.com/docs/en/code-review) | "Code Review is in research preview, available for Team and Enterprise subscriptions." | Team и Enterprise |
| [Ultrareview](https://code.claude.com/docs/en/ultrareview) | "Ultrareview is a research preview feature." | Подписка Claude |
| [Плагин Security guidance](https://code.claude.com/docs/en/security-guidance) | – | Все тарифы; требует Python 3.7+ (для агентного ревью коммитов и при работе со сторонними провайдерами - Python 3.10+) |
| [Claude Security](https://code.claude.com/docs/en/claude-security) | – | Платные планы, доступ к Anthropic API или сторонние облачные провайдеры |

<a id="other"></a>
## Прочее

| Возможность | Статус | Кому доступно |
|---|---|---|
| [Артефакты (Artifacts)](https://code.claude.com/docs/en/artifacts) | – | Pro, Max, Team и Enterprise на Anthropic API |
| [Шаблоны `/design` и `/slides`](https://code.claude.com/docs/en/artifacts#start-from-a-slides-design-or-docs-template) | "Templates are in beta." | Включены по умолчанию на Pro, Max и Team; на Enterprise включаются администратором |

---

<sub>Sources: [Feature availability](https://code.claude.com/docs/en/feature-availability) and each feature's linked page, checked on 2026-10-05.</sub>

<sub>Up: [Reference](README.md)</sub>
