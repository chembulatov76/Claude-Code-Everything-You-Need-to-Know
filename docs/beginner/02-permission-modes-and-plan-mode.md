# Choose who approves what: permission modes and plan mode

<sub>**Beginner** · Lesson 2 of 5 · about 20 minutes · Needs: [Install, sign in and look around](01-install-and-look-around.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого урока вы сможете выбирать подходящий режим разрешений (permission mode) для задачи, переключать режимы во время сессии и использовать режим планирования (plan mode) для утверждения плана до того, как Claude изменит ваши файлы. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can pick a permission mode for a task, switch modes during a session, and use plan mode to approve a plan before Claude edits your files.</details>

## You need

- Ваша клонированная копия учебного шаблона без незакоммиченных изменений и Node.js LTS для проверок (в Codespace доступны оба инструмента). Чтобы практиковаться в собственном репозитории, закоммитьте или сохраните в stash текущую работу, чтобы рабочее дерево было чистым, и сохраните также копию шаблона (проверка вызывается из неё). Настройте папку `.practice/` в соответствии с описанием в [Уроке 1](01-install-and-look-around.md#you-need), если не сделали этого ранее.
- Расход токенов: минимальный (light).

> [!NOTE]
> В нативной среде Windows запускайте команды в PowerShell и используйте строки с пометкой **Windows** там, где синтаксис различается. Если комбинация `Shift+Tab` не реагирует, нажимайте `Alt+M` ([Interactive mode](https://code.claude.com/docs/en/interactive-mode)).

## The idea

[Режим разрешений (permission mode)](https://code.claude.com/docs/en/permission-modes) определяет, какие действия Claude может предпринимать без вашего предварительного одобрения. Переключение режимов выполняется комбинацией `Shift+Tab`, а текущий активный режим постоянно отображается в строке статуса под полем ввода.

В данный момент важны четыре режима:

| Строка статуса отображает | Режим | Что Claude выполняет без подтверждения |
|---|---|---|
| `⏸ manual mode on` | Manual | Только чтение |
| `⏵⏵ accept edits on` | Accept edits | Чтение, редактирование файлов, типовые файловые команды |
| `⏸ plan mode on` | Plan | Чтение, плюс команды, одобренные фоновой проверкой безопасности при доступности auto mode, затем составление плана. Файлы не редактируются до одобрения плана вами |
| `⏵⏵ auto mode on` | Auto | Все операции под контролем фоновых проверок безопасности |

Режим Manual подходит для чувствительных задач и незнакомой кодовой базы, где вы хотите лично контролировать каждое действие. Режим Auto эффективен для длительных масштабных задач, избавляя вас от ручного подтверждения каждого шага.

Режим планирования (plan mode): это ключевая привычка, которую стоит сформировать первой: вы видите всё планируемое изменение целиком до того, как Claude коснётся хотя бы одного файла.

## Worked example

1. Запустите `claude` в учебной копии репозитория и обратите внимание на строку статуса. Новая сессия обычно запускается в режиме auto mode.
2. Нажимайте комбинацию `Shift+Tab`, пока в строке статуса не появится `⏸ plan mode on`. Из режима auto первое нажатие включает Manual, затем Accept edits, затем Plan.
3. Отправьте запрос на внесение изменения:

   ```text
   Make parseConfig in src/config.js reject an empty file with a clear error message, and add a test for it.
   ```

4. Claude исследует файлы и может выполнять диагностические команды, но не вносит правок в файлы. Если потребуется уточнение, Claude задаст вопрос с вариантами выбора ([multiple-choice question](https://code.claude.com/docs/en/tools-reference#askuserquestion-tool-behavior)): выберите вариант или введите собственный ответ, после чего планирование продолжится. Затем Claude сформирует план и предложит варианты дальнейших действий:
   - **Yes, and use auto mode** (или **Yes, auto-accept edits**, если автоматический режим вам недоступен)
   - **Yes, manually approve edits**
   - Текстовое поле с надписью **Tell Claude what to change** (в [документации](https://code.claude.com/docs/en/permission-modes#review-and-approve-a-plan) этот пункт называется **No, keep planning**)

   При появлении меню по умолчанию подсвечен первый вариант. До вашего выбора сессия остаётся в режиме планирования.
5. Отправьте план на доработку один раз (даже если он выглядит корректно), чтобы закрепить навык. Стрелкой вниз (`Down arrow`) переместите выделение на пункт **Tell Claude what to change**, введите следующее уточнение и нажмите `Enter`:

   ```text
   Keep the change to src/config.js and test/config.test.js. Don't add new files.
   ```

   Claude обновит план и снова покажет меню. Если вы случайно нажали `Enter` до ввода текста, Claude Code выведет сообщение `User rejected Claude's plan:`, оставаясь в режиме планирования: в таком случае просто введите уточнение в обычной строке промпта.
6. Когда согласованный план готов, оставьте его на экране и откройте второй терминал в той же папке. Сохраните статус git и список файлов, которые план намеревается изменить (по одному относительному пути на строку с прямыми слешами, как их отображает `git status`):

   ```bash
   git status --porcelain > .practice/b-2-before.txt
   printf '%s\n' src/config.js test/config.test.js > .practice/b-2-plan.txt
   ```

   **Windows:** строка `'src/config.js','test/config.test.js' | Set-Content .practice/b-2-plan.txt` заменяет вызов `printf`.
7. Вернитесь в окно Claude Code, стрелкой вниз выберите пункт **Yes, manually approve edits** и нажмите `Enter`. (Нажатие `Enter` на первом подсвеченном пункте разрешило бы Claude редактировать файлы без подтверждений). Сессия выйдет из plan mode и переключится в Manual mode, поэтому Claude будет запрашивать подтверждение перед каждой правкой и перед большинством команд терминала (таких как запуск тестов).

   Внимательно читайте каждый запрос и отвечайте простым нажатием **Yes**: другие варианты Yes отключают повторные запросы для аналогичных действий или переводят сессию в Accept edits либо auto mode ([Configure permissions](https://code.claude.com/docs/en/permissions#permission-system)). Если команда выходит за рамки утверждённого плана, выбирайте **No** и уточняйте, что следует сделать вместо этого. После завершения работы Claude сохраните итоговое состояние git во втором терминале:

   ```bash
   git status --porcelain > .practice/b-2-after.txt
   ```

8. Отложите изменения из примера в stash, чтобы блок практического задания (Your turn) начался на чистом дереве. Git игнорирует папку `.practice/`, поэтому сохранённые снимки останутся на месте:

   ```bash
   git stash push --include-untracked -m b-2-example
   ```

## Your turn

Повторите эти действия в учебной копии или собственном репозитории, выполнив небольшое самостоятельное изменение:

1. Переключитесь в режим plan mode с помощью `Shift+Tab`, пока в строке статуса не появится `⏸ plan mode on`.
2. Опишите требуемое изменение и ответьте на уточняющие вопросы Claude, если они возникнут.
3. Отправьте план на доработку один раз: введите одно исправление в строке **Tell Claude what to change** и нажмите `Enter`.
4. Пока итоговый план отображается на экране, выполните команды шага 6 во втором терминале, указав список файлов из вашего плана.
5. Подтвердите выполнение через **Yes, manually approve edits**, подтверждайте каждую правку и команду как в шаге 7, затем сохраните статус: `git status --porcelain > .practice/b-2-after.txt`.

Остановитесь до выполнения коммита. Следующий урок начнётся с чистого рабочего дерева и доведёт задачу до коммита.

## Check

Задание считается выполненным, когда соблюдены все условия:

- [ ] Файл `.practice/b-2-before.txt`, сохранённый при отображении плана, пуст. Команда `cat .practice/b-2-before.txt` ничего не выводит.
- [ ] Каждый путь из `.practice/b-2-after.txt` присутствует в файле `.practice/b-2-plan.txt`.
- [ ] Self-check (not tested): сформулируйте в одном предложении, в каких ситуациях вы выберете режим Manual вместо Auto.

Во втором терминале запустите команду `npm run check -- b-2` в учебной копии. В собственном репозитории выполните из учебной копии: `npm run check -- b-2 --dir <your repo>`.

Если файл `.practice/b-2-before.txt` содержит файлы, это означает, что либо дерево было загрязнено перед началом Your turn (наличие файлов из примера указывает на пропуск шага 8), либо Claude изменил файлы до подтверждения плана: проверьте, горел ли статус `⏸ plan mode on` при отправке запроса, и найдите команду, изменившую файлы. В любом случае отложите изменения через `git stash push --include-untracked` (или закоммитьте свою работу) и повторите Your turn с шага 1. Если изменённый файл отсутствует в `.practice/b-2-plan.txt`, убедитесь, что пути записаны в формате `git status`, и сравните одобренный план с реальными правками.

## Go further

- Введите команду `/plan <your task>`, чтобы войти в режим планирования и запустить формулирование задачи одной командой ([Commands](https://code.claude.com/docs/en/commands)).
- [Analyze before you edit with plan mode](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode): подробный разбор планирования, включая редактирование файла плана до его одобрения.

---

<sub>Sources: [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Commands](https://code.claude.com/docs/en/commands) · [Tools reference](https://code.claude.com/docs/en/tools-reference) · [Configure permissions](https://code.claude.com/docs/en/permissions)</sub>

<sub>← [Install, sign in and look around](01-install-and-look-around.md) · [Beginner index](README.md) · [Your first change, from request to commit](03-first-change.md) → · Topic: [Permissions and safety](../topics/permissions-and-safety.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-2)</sub>
