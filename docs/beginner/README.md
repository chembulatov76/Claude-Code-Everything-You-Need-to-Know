# Beginner

<sub>**Beginner** · about 2½ hours · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Одна сессия, одно изменение. Вы установите Claude Code, определите, что ему разрешено делать без подтверждения, проведёте одно изменение от запроса до коммита, научитесь удерживать сессию в рамках задачи и добавите проекту долговременную память. <details style="display:inline"><summary><code>[en]</code></summary>One session, one change. You install Claude Code, decide what it may do without asking, take one change from request to commit, keep a long session on track, and give it a project memory.</details>

<a id="youre-ready-for-this-level-if-you-can"></a>
## You're ready for this level if you can

- [ ] Уверенно работать в терминале: переходить между папками, запускать команды и читать их вывод.
- [ ] Пользоваться git: клонировать репозиторий, делать коммиты и ориентироваться в `git status` и `git diff`.
- [ ] Иметь платную подписку Claude (Pro, Max, Team или Enterprise; бесплатный план не включает Claude Code), аккаунт Claude Console либо доступ через поддерживаемого облачного провайдера ([Quickstart](https://code.claude.com/docs/en/quickstart), [Advanced setup](https://code.claude.com/docs/en/setup)). В Уроке 1 показано, как войти в систему.

Впервые в терминале? Официальное [Руководство по терминалу для новичков](https://code.claude.com/docs/en/terminal-guide) шаг за шагом показывает, как открыть терминал и установить Claude Code.

<a id="practice"></a>
## Practice

Перед началом Урока 1 создайте собственную копию [шаблона для практики](https://github.com/wesammustafa/claude-code-practice): нажмите **Use this template**, затем **Create a new repository** и клонируйте полученный репозиторий. Все упражнения уровня Beginner и выпускное задание выполняются в нём.

Каждый основной урок и итоговое задание завершаются скриптом проверки, который вы запускаете в своей копии командой `npm run check -- <id>`, где идентификатор: от `b-1` до `b-5` для уроков и `b-capstone` для выпускного задания. Факультативы завершаются чек-листом для самостоятельной проверки. Для работы скриптов проверки необходим Node.js LTS (в среде Codespace вашей копии он уже предустановлен). Если вы хотите практиковаться в одном из своих рабочих репозиториев, сохраните также копию шаблона: проверки запускаются из неё с указанием пути через флаг `--dir`.

На чистой Windows, если PowerShell блокирует запуск `npm` с сообщением `running scripts is disabled on this system` ([Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install#running-scripts-is-disabled-on-this-system)), следуйте указаниям для Windows из [Урока 1](01-install-and-look-around.md#you-need).

<a id="lessons"></a>
## Lessons

1. [Установка, вход и первый запуск](01-install-and-look-around.md) <details style="display:inline"><summary><code>[en]</code></summary>Install, sign in and look around</details>
2. [Режимы разрешений и режим планирования](02-permission-modes-and-plan-mode.md) <details style="display:inline"><summary><code>[en]</code></summary>Permission modes and plan mode</details>
3. [Первое изменение: от запроса до коммита](03-first-change.md) <details style="display:inline"><summary><code>[en]</code></summary>Your first change, from request to commit</details>
4. [Управление сессией и удержание фокуса](04-keep-a-session-on-track.md) <details style="display:inline"><summary><code>[en]</code></summary>Keep a session on track</details>
5. [Память проекта с CLAUDE.md](05-project-memory.md) <details style="display:inline"><summary><code>[en]</code></summary>Project memory with CLAUDE.md</details>

Каждый урок занимает около 20 минут, а выпускное задание: от 30 до 60 минут.

<a id="capstone"></a>
## Capstone

[Исправление ошибки в коде: от плана до коммита](capstone.md): зафиксируйте версию Claude Code, затем найдите и устраните зарегистрированную ошибку в учебном шаблоне: спланируйте исправление, измените два исходных файла, добавьте тест (падающий до фикса и проходящий после него) и сделайте коммит. Проверочный скрипт анализирует изменения в файлах и коммитах git, а остальные навыки из списка [Критериев завершения Beginner](#by-the-end-of-this-level-you-can) вы отмечаете самостоятельно.

<a id="by-the-end-of-this-level-you-can"></a>
## By the end of this level you can

Этот перечень представляет собой квалификационный профиль уровня Beginner ([Глоссарий](../reference/glossary.md)), который выпускное задание объединяет в единый рабочий процесс.

- [ ] Установить Claude Code, выполнить вход, запустить инструмент в репозитории и определить версию, модель и аккаунт текущей сессии.
- [ ] Выбирать и переключать режимы разрешений, а также утверждать план в режиме plan mode до внесения любых правок в файлы.
- [ ] Проводить изменение от запроса до коммита: исследование, план, реализация, запуск тестов, проверка diff, корректировка, коммит.
- [ ] Удерживать сессию в рамках задачи, используя команды context, clear, compact, rewind и resume.
- [ ] Создавать файл CLAUDE.md проекта и подтверждать, что новая сессия соблюдает описанные в нём правила.

<a id="electives"></a>
## Electives

Дополнительные практические темы (факультативы):

- [Интеграция с редакторами и IDE](electives/ide-extensions.md)
- [Скриншоты и изображения в промптах](electives/screenshots-and-images.md)
- [Встроенные инструменты обучения](electives/built-in-teachers.md): команда `/powerup`, стили вывода Learning и Explanatory, а также субагент `claude-code-guide`

<a id="official-companions"></a>
## Official companions

- [Quickstart](https://code.claude.com/docs/en/quickstart): пошаговое руководство от Anthropic от установки до первого изменения.
- [Claude Code 101](https://academy.claude.com/courses/claude-code-101): вводный курс от Claude Academy.
- `/powerup`: интерактивные обучающие уроки прямо внутри Claude Code ([Commands](https://code.claude.com/docs/en/commands)).

<a id="next-level"></a>
## Next level

[Intermediate](../intermediate/README.md): конфигурация рабочего места индивидуального разработчика.

---

<sub>Sources: [Quickstart](https://code.claude.com/docs/en/quickstart) · [Advanced setup](https://code.claude.com/docs/en/setup) · [Terminal guide for new users](https://code.claude.com/docs/en/terminal-guide) · [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install) · [Commands](https://code.claude.com/docs/en/commands) · [Output styles](https://code.claude.com/docs/en/output-styles) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents)</sub>

<sub>Up: [Claude Code: Everything You Need to Know](../../README.md#pick-your-level)</sub>
