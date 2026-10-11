<a id="slash-commands--cheatsheet"></a><a id="commands-by-level"></a>
# Команды по уровням

<sub>Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Встроенные команды, системные навыки, воркфлоу и формы вызова `claude` из терминала, сгруппированные по уровням руководства. Это не полный перечень: официальный справочник [commands reference](https://code.claude.com/docs/en/commands#all-commands) описывает все команды и опции, [CLI reference](https://code.claude.com/docs/en/cli-reference#cli-flags) содержит флаги командной строки, а ввод символа `/` в терминале выводит список команд, доступных в текущей сессии.

<a id="common-built-in-slash-commands"></a><a id="beginner"></a>
## Начальный уровень (Beginner)

- `/status`: информация о версии, модели, аккаунте и сетевом подключении. [Установка, вход и знакомство с интерфейсом](../beginner/01-install-and-look-around.md)
- `/exit`: завершение сессии. [Установка, вход и знакомство с интерфейсом](../beginner/01-install-and-look-around.md)
- `/plan`: переход в режим планирования из строки ввода (опционально с текстом задачи). [Режимы разрешений и режим планирования](../beginner/02-permission-modes-and-plan-mode.md)
- `/diff`: просмотр изменений в рабочей копии, включая правки Claude. [Первое изменение: от запроса до коммита](../beginner/03-first-change.md)
- `/context`: степень заполнения контекстного окна в виде цветной сетки. [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md)
- `/clear`: начало нового диалога с чистым контекстом. [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md)
- `/compact`: сжатие истории диалога для освобождения контекста. [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md)
- `/rewind`: возврат к предыдущей точке диалога, кода или обоих состояний ([Checkpointing](https://code.claude.com/docs/en/checkpointing)). [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md)
- `/resume`: возобновление одной из прошлых сессий ([Manage sessions](https://code.claude.com/docs/en/sessions)). [Контроль контекста и точность сессии](../beginner/04-keep-a-session-on-track.md)
- `/init`: генерация начального файла `CLAUDE.md` для проекта. [Память проекта с CLAUDE.md](../beginner/05-project-memory.md)
- `/memory`: редактирование файлов `CLAUDE.md` и управление автоматической памятью. [Память проекта с CLAUDE.md](../beginner/05-project-memory.md)

Факультативы: `/powerup` и `/output-style` ([Встроенные обучающие инструменты](../beginner/electives/built-in-teachers.md)).

<a id="intermediate"></a>
## Средний уровень (Intermediate)

- `/skills`: список навыков, доступных в текущей сессии. [Превращение повторяющегося процесса в навык](../intermediate/01-first-skill.md)
- `/doctor`: диагностика окружения; команда `/doctor prompt-audit` проверяет файлы инструкций. [Организация памяти проекта и проверка загрузки](../intermediate/02-organize-memory.md)
- `/model`: переключение модели с сохранением значения по умолчанию; клавиши стрелок регулируют effort на поддерживаемых моделях. [Выбор модели и глубины рассуждений](../intermediate/03-model-and-effort.md)
- `/effort`: выбор уровня усилий рассуждений. [Выбор модели и глубины рассуждений](../intermediate/03-model-and-effort.md)
- `/permissions`: управление правилами allow, ask и deny. [Разрешения, области видимости настроек и песочница](../intermediate/04-permissions-and-sandbox.md)
- `/sandbox`: включение и выключение песочницы. [Разрешения, области видимости настроек и песочница](../intermediate/04-permissions-and-sandbox.md)
- `/hooks`: просмотр настроенных перехватчиков событий (хуков). [Применение правил с помощью хуков](../intermediate/05-hooks.md)
- `/agents`: справочная панель по созданию субагентов и управлению ими. [Делегирование кастомному субагенту](../intermediate/06-subagents.md)
- `/mcp`: управление подключениями и авторизацией MCP-серверов. [Подключение инструментов через MCP](../intermediate/07-mcp.md)
- `/plugin`: обзор, установка и управление плагинами. [Установка плагинов и управление ими](../intermediate/08-plugins.md)

Факультативы: `/statusline` ([Строка состояния](../intermediate/electives/status-line.md)); `/chrome` ([Claude в Chrome](../intermediate/electives/claude-in-chrome.md)); `/fast` ([Быстрый режим](../intermediate/electives/fast-mode.md)).

<a id="advanced"></a>
## Продвинутый уровень (Advanced)

- `claude --worktree <name>` (`-w`): запуск сессии в изолированном git worktree по пути `.claude/worktrees/<name>` ([CLI reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). [Параллельные сессии с worktree](../advanced/01-parallel-sessions.md)
- `/tasks`: просмотр и управление фоновыми задачами в сессии, включая завершенные субагенты. [Выбор паттерна оркестрации](../advanced/02-orchestration-patterns.md)
- `/workflows`: отслеживание, пауза, возобновление и сохранение динамических воркфлоу. [Запуск и сохранение динамического воркфлоу](../advanced/03-dynamic-workflows.md)
- `/deep-research <question>`: встроенный воркфлоу, распараллеливающий поиск по веб-источникам, проверяющий факты и формирующий отчет со ссылками. [Запуск и сохранение динамического воркфлоу](../advanced/03-dynamic-workflows.md)
- `claude -p "<prompt>"`: выполнение одиночного запроса без интерактивного режима, вывод ответа в stdout и выход; для скриптов используются флаги `--output-format json`, `--allowedTools` и `--max-turns` ([CLI reference](https://code.claude.com/docs/en/cli-reference#cli-commands)). [Автоматизация Claude Code с помощью `claude -p`](../advanced/04-headless.md)
- `/install-github-app`: установка приложения Claude GitHub App для репозитория с настройкой воркфлоу и секретов. [Интеграция с GitHub Actions](../advanced/05-github-actions.md)
- `claude plugin validate <dir>`: валидация маркетплейса или папки плагина перед публикацией; флаг `--strict` переводит предупреждения в ошибки ([Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate)). [Совместное использование настроек в команде](../advanced/06-share-your-setup.md)
- `/usage`: статистика стоимости сессии, лимиты тарифа и активность. [Ограничения для автономных запусков](../advanced/07-bounded-runs.md)

Факультативы: `claude agents` и `/background` ([Режим Agent view](../advanced/electives/agent-view.md)); `/goal`, `/loop` и `/schedule` ([`/goal`, `/loop` и рутины](../advanced/electives/goal-loop-routines.md)); `claude plugin eval` ([Тестирование и публикация плагина](../advanced/electives/publish-a-plugin.md)).

<a id="custom-slash-commands-skills"></a><a id="your-own-commands"></a>
## Пользовательские команды

Собственная команда создается в виде навыка (skill): файла `SKILL.md` в папке `.claude/skills/<name>/` или Markdown-файла в устаревшем формате в `.claude/commands/` ([Extend Claude with skills](https://code.claude.com/docs/en/skills)). Их создание подробно рассмотрено в уроке [Превращение повторяющегося процесса в навык](../intermediate/01-first-skill.md).

<a id="shortcuts-and-cheat-sheets"></a>
## Горячие клавиши и шпаргалки

- [Interactive mode](https://code.claude.com/docs/en/interactive-mode): горячие клавиши, режимы ввода и интерактивные функции.
- [Claude Code cheatsheet](https://support.claude.com/en/articles/14553413-claude-code-cheatsheet): ключевые команды и комбинации клавиш для быстрого старта.

---

<sub>Sources: [Commands](https://code.claude.com/docs/en/commands) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Checkpointing](https://code.claude.com/docs/en/checkpointing) · [Manage sessions](https://code.claude.com/docs/en/sessions) · [Extend Claude with skills](https://code.claude.com/docs/en/skills) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference) · [Claude Code cheatsheet](https://support.claude.com/en/articles/14553413-claude-code-cheatsheet)</sub>

<sub>Up: [Reference](README.md)</sub>
