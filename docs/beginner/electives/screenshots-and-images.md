# Give Claude a screenshot

<sub>**Beginner** · Elective · about 15 minutes · Needs: [Your first change, from request to commit](../03-first-change.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого факультатива вы сможете прикреплять скриншот или изображение к диалогу с Claude Code и получать изменения в коде на основе того, что на нём показано. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can add a screenshot or an image to a conversation and get a change from what it shows.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений и инструмент создания скриншотов вашей операционной системы.
- Расход токенов: минимальный (light).

> [!NOTE]
> В GitHub Codespaces Claude Code выполняется внутри контейнера codespace, а не на вашем локальном компьютере ([Development containers](https://code.claude.com/docs/en/devcontainer)), поэтому передавайте скриншоты в виде файлов внутри контейнера. Сохраните скриншот на локальном компьютере, переименуйте в соответствии с инструкцией (`shot.png`, затем `test-summary.png`) и перетащите файл из файлового менеджера в папку `.practice` в проводнике (Explorer) codespace ([GitHub Codespaces](https://code.visualstudio.com/docs/remote/codespaces#_known-limitations-and-adaptations)). В тех местах, где в разобранном примере предлагается вставить картинку из буфера обмена, укажите путь к файлу в промпте, например: `Look at this image: .practice/shot.png`. В блоке Your turn пропустите команду `mv`.

## The idea

Некоторые вещи гораздо проще показать, чем описывать словами: сообщение об ошибке на экране, особенности вёрстки, схему или диаграмму. Claude Code поддерживает мультимодальный ввод изображений, и прикрепить картинку к диалогу можно тремя способами ([Common workflows](https://code.claude.com/docs/en/common-workflows#work-with-images)): <details style="display:inline"><summary><code>[en]</code></summary>Some things are faster to show than to describe: an error on screen, a layout, a diagram. Claude Code reads images, and you can add one to the conversation in three ways</details>

- перетащить файл изображения в окно Claude Code (drag and drop);
- скопировать изображение в буфер обмена и вставить сочетанием клавиш `Ctrl+V` (в терминале iTerm2 также работает `Cmd+V`; в Windows и WSL используется `Alt+V`) ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#general-controls));
- указать относительный или абсолютный путь к изображению в промпте, например: "Look at this image: /path/to/shot.png".

## Worked example

1. В вашей учебной копии запустите проверку ссылок на тестовых файлах:

   ```bash
   npm run linkcheck -- samples
   ```

   Утилита выведет неработающую ссылку в файле `guide.md`, а затем строку `1 broken link(s)`.
2. Сделайте скриншот этого вывода и скопируйте его в буфер обмена. На macOS удерживайте клавишу `Ctrl` вместе со стандартными клавишами (например, `Ctrl+Shift+Cmd+4`): без `Ctrl` система сохраняет скриншот файлом на рабочий стол ([Take a screenshot on Mac](https://support.apple.com/en-us/102646)).
3. Запустите `claude` и вставьте изображение из буфера обмена. В поле ввода появится интерактивный бейдж `[Image #1]` ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#general-controls)). Если бейдж не появился, значит, изображение не добавилось: перетащите файл скриншота в окно терминала с Claude Code либо укажите путь к файлу, как описано в разделе Your turn. Затем отправьте запрос:

   ```text
   This is the link checker's output. Change the last line so it also says how many Markdown files were checked, like "1 broken link(s) in 2 files".
   ```

4. Подтвердите выполнение правки, если Claude запросит разрешение. Затем во втором терминале в той же папке снова запустите `npm run linkcheck -- samples`. Последняя строка вывода теперь отображает количество проверенных файлов.
5. Во втором терминале выполните команду `git diff`, чтобы увидеть внесённые изменения. Затем временно отложите изменения из примера в git stash, чтобы приступить к разделу Your turn в чистом репозитории:

   ```bash
   git stash push --include-untracked -m b-images-example
   ```

   После этого команда `git status` сообщит: `nothing to commit, working tree clean`.

## Your turn

1. Во втором терминале запустите `npm test` и сделайте скриншот финальной части вывода: нескольких последних названий тестов и итоговой сводки под ними. Сохраните скриншот в файл, затем переместите его в папку `.practice/` вашей учебной копии. Git игнорирует эту папку, поэтому файл изображения не появится в статусе изменений:

   ```bash
   mv "<your screenshot file>" .practice/test-summary.png
   ```

   **WSL:** Windows сохраняет файлы на своём диске, к которому WSL обращается через точку монтирования `/mnt/c` ([Working across file systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)): путь `C:\Users\<user name>\shot.png` преобразуется в `/mnt/c/Users/<user name>/shot.png`.
2. В окне Claude Code (запустите `claude`, если агент не запущен) обратитесь к изображению по пути к файлу: `Look at this image: .practice/test-summary.png. What does each line of this summary mean?`
3. Попросите внести небольшое изменение на основе информации из изображения (например, сделать понятнее название одного из тестов, показанных на скриншоте) и подтвердите правку при запросе разрешений. Проверьте результат с помощью `git diff` во втором терминале.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте каждый пункт самостоятельно. Задание выполнено, когда выполняются все условия:

- [ ] Команда `git diff` отображает изменение, о котором вы попросили на основе анализа изображения.
- [ ] Self-check (not tested): вы добавили одно изображение вставкой из буфера обмена или перетаскиванием, а второе - по пути к файлу (в среде Codespaces - оба через путь к файлу).

По завершении отложите сделанные изменения, чтобы следующий урок начался в чистом рабочем дереве: `git stash push --include-untracked -m b-images`. После этого `git status` должен показывать: `nothing to commit, working tree clean`.

## Watch out

- Изображение отправляется в Claude вместе с вашим текстовым промптом. Перед отправкой обязательно кадрируйте скриншот или замазывайте конфиденциальную информацию: токены, пароли, адреса электронной почты и персональные данные клиентов.

## Go further

- [Work with images](https://code.claude.com/docs/en/common-workflows#work-with-images): дополнительные сценарии использования изображений (например, работа с дизайн-макетами интерфейса).

---

<sub>Sources: [Common workflows](https://code.claude.com/docs/en/common-workflows) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Development containers](https://code.claude.com/docs/en/devcontainer) · [GitHub Codespaces](https://code.visualstudio.com/docs/remote/codespaces) · [Take a screenshot on Mac](https://support.apple.com/en-us/102646) · [Working across file systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)</sub>

<sub>Back to the [Beginner index](../README.md) · Topic: [Memory and context](../../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-images)</sub>
