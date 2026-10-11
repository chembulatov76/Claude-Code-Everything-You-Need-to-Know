# Use Claude Code in your editor

<sub>**Beginner** · Elective · about 15 minutes · Needs: [Install, sign in and look around](../01-install-and-look-around.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого факультатива вы сможете запускать Claude Code внутри VS Code или JetBrains IDE, ссылаться на выделенные строки кода и проверять предлагаемые изменения во встроенном diff-вьюере редактора. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can run Claude Code inside VS Code or a JetBrains IDE, point it at the lines you're looking at, and review its edits in the editor's diff view.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений, открытая в VS Code или JetBrains IDE (например, IntelliJ IDEA или PyCharm).
- Для JetBrains: утилита командной строки Claude Code из урока 1, так как плагин запускает её во встроенном терминале IDE.
- Расход токенов: минимальный (light).

## The idea

Терминал не единственное место, где работает Claude Code. Две официальные интеграции для редакторов позволяют использовать ассистента прямо рядом с кодом: <details style="display:inline"><summary><code>[en]</code></summary>The terminal isn't the only place Claude Code runs. Two editor integrations put it next to your code</details>

- **Расширение для VS Code** содержит собственную панель чата и включает встроенную копию Claude Code. В режиме Manual оно показывает каждое предлагаемое изменение бок о бок с оригиналом (side by side) и запрашивает подтверждение перед применением ([VS Code](https://code.claude.com/docs/en/vs-code)).
- **Плагин для JetBrains** запускает команду `claude` в терминале IDE и открывает предлагаемые правки во встроенном средстве просмотра diff ([JetBrains IDEs](https://code.claude.com/docs/en/jetbrains)).

В обоих случаях это тот же самый Claude Code: та же учётная запись, те же режимы разрешений и единый файл правил `CLAUDE.md`.

## Worked example

В VS Code:

1. Откройте панель расширений сочетанием клавиш `Cmd+Shift+X` (macOS) или `Ctrl+Shift+X` (Windows и Linux), найдите "Claude Code" и нажмите кнопку **Install**. Расширение не работает в ограниченном режиме (Restricted Mode), поэтому если VS Code показывает баннер Restricted Mode или спрашивает, доверяете ли вы папке, подтвердите доверие: это ваш собственный локальный код ([Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust)).
2. Откройте файл `src/config.js`. Нажмите на значок Spark (искорка) в правом верхнем углу редактора, чтобы открыть Claude Code, и войдите в аккаунт, если потребуется. Значка нет? Нажмите `✻ Claude Code` в строке состояния (status bar) в правом нижнем углу.
3. Выделите тело функции `parseConfig` и нажмите `Option+K` (macOS) или `Alt+K` (Windows и Linux). В поле ввода появится ссылка на путь к файлу и номера строк, например `@src/config.js#7-17`. Задайте вопрос:

   ```text
   What happens if linkcheck.json holds an array instead of an object?
   ```

4. Переключите режим в нижней части поля ввода на Manual, затем попросите внести небольшое изменение, например комментарий над функцией `parseConfig`. Изменение откроется рядом с оригиналом в окне сравнения, а Claude запросит разрешение в панели чата. Выберите **No** ([Configure permissions](https://code.claude.com/docs/en/permissions#add-a-comment-when-you-answer-a-permission-prompt)). Кнопки **Reject this change** внутри diff отменяют только конкретную правку в предложении, но не завершают процедуру проверки.

В JetBrains IDE:

1. Установите [плагин Claude Code](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-) из JetBrains Marketplace и перезапустите IDE.
2. Нажмите кнопку Claude Code в интерфейсе IDE или нажмите сочетание клавиш `Cmd+Esc` (macOS) или `Ctrl+Esc` (Windows и Linux), чтобы запустить Claude Code в терминале IDE. Если ничего не происходит, откройте терминал IDE вручную и выполните `claude`: запущенный оттуда, агент автоматически подключается к IDE. Если IDE выводит сообщение "Cannot launch Claude Code", значит, утилита `claude` не найдена в переменной `PATH`: укажите полный путь к `claude` в меню **Settings** > **Tools** > **Claude Code [Beta]** > **Claude command** ([plugin settings](https://code.claude.com/docs/en/jetbrains#general-settings)).
3. Откройте `src/config.js`, выделите тело функции `parseConfig` и нажмите `Cmd+Option+K` (macOS) или `Alt+Ctrl+K` (Windows и Linux). В поле ввода добавится ссылка вида `@src/config.js#L7-17`. Задайте тот же вопрос.
4. Нажимайте `Shift+Tab`, пока в строке статуса под промптом не появится `⏸ manual mode on` ([Choose a permission mode](https://code.claude.com/docs/en/permission-modes#switch-permission-modes)). В нативной Windows, если `Shift+Tab` не срабатывает, используйте `Alt+M` ([Interactive mode](https://code.claude.com/docs/en/interactive-mode)). Затем попросите выполнить то же небольшое изменение: оно откроется во встроенном diff-вьюере IDE, а Claude запросит разрешение в терминале. Выберите ответ **No**.

## Your turn

1. Установите расширение или плагин для того редактора, который вы используете.
2. Откройте `src/links.js`, выделите функцию `findLinks` целиком и нажмите шорткат для создания ссылки на код: `Option+K` (macOS) или `Alt+K` (Windows и Linux) в VS Code, либо `Cmd+Option+K` (macOS) / `Alt+Ctrl+K` (Windows и Linux) в JetBrains. Затем спросите, какие типы ссылок она находит.
3. Переключитесь в режим Manual, как в разобранном примере, попросите внести небольшую правку и просмотрите её в окне diff. После этого выберите **No** в запросе разрешений Claude: в панели чата VS Code или в терминале JetBrains.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте каждый пункт самостоятельно. Задание выполнено, когда выполняются все условия:

- [ ] Интеграция установлена. VS Code: панель расширений (`Cmd+Shift+X` или `Ctrl+Shift+X`) показывает Claude Code как установленное. JetBrains: плагин отображается в списке **Settings** > **Plugins** > **Installed**.
- [ ] Команда `git diff` ничего не выводит, поскольку вы отклонили предложенную правку.
- [ ] Self-check (not tested): вы задали вопрос со ссылкой на предварительно выделенные строки кода.

Если `git diff` отображает изменения, значит, они были применены: сессия находилась не в режиме Manual, либо вы случайно подтвердили правку. Отмените изменение командой `git restore <file>`, подставив имя файла из вывода `git diff`. Затем переключитесь в Manual, снова запросите правку и ответьте **No**.

## Go further

- [Desktop quickstart](https://code.claude.com/docs/en/desktop-quickstart): работа с Claude Code в настольном приложении Claude desktop.
- [VS Code commands and shortcuts](https://code.claude.com/docs/en/vs-code#vs-code-commands-and-shortcuts).

---

<sub>Sources: [VS Code](https://code.claude.com/docs/en/vs-code) · [JetBrains IDEs](https://code.claude.com/docs/en/jetbrains) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust)</sub>

<sub>Back to the [Beginner index](../README.md) · Topic: [Память и контекст](../../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-ide)</sub>
