# Your first change, from request to commit

<sub>**Beginner** · Lesson 3 of 5 · about 20 minutes · Needs: [Permission modes and plan mode](02-permission-modes-and-plan-mode.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого урока вы сможете провести одно изменение от формулирования задачи до коммита: исследование, планирование, реализация, тестирование, просмотр diff, корректировка и коммит. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can take one change from request to commit: explore, plan, implement, test, review the diff, correct, and commit.</details>

## You need

- Ваша копия учебного шаблона и Node.js LTS (в среде Codespace доступно по умолчанию).
- При выполнении в собственном репозитории сохраните также учебный шаблон (скрипт проверки запускается из него). Создайте папку `.practice/` в соответствии с инструкцией [Урока 1](01-install-and-look-around.md#you-need), если не сделали этого ранее.
- Отсутствие незакоммиченных изменений в рабочем каталоге. Если изменения из Урока 2 всё ещё присутствуют, отложите их командой `git stash push --include-untracked -m b-2`; в собственном репозитории закоммитьте их, если хотите сохранить. В последующих уроках эти данные не используются, а просмотреть их список можно через `git stash list`.
- Расход токенов: минимальный (light).

## The idea

Запрашивать изменение и без проверки соглашаться со всем предложенным кодом: верный путь к ошибкам и деградации сессии. Руководство по эффективным практикам Anthropic ([best practices](https://code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code)) рекомендует разделять процесс на четыре фазы: исследование (explore), планирование (plan), реализация (implement) и фиксация (commit). Этот урок добавляет точку контроля на каждом шаге:

1. **Исследование (Explore)** в режиме plan mode: Claude читает файлы и отвечает на вопросы, ничего не изменяя.
2. **Планирование (Plan)**: план утверждается только тогда, когда он в точности соответствует вашим ожиданиям.
3. **Реализация (Implement)**: подтверждение правок по мере их внесения.
4. **Тестирование (Test)**: именно прохождение тестов доказывает работоспособность кода, а не текстовое резюме, составленное моделью.
5. **Просмотр diff (Review the diff)** построчно.
6. **Корректировка (Correct)**: формулирование необходимых правок обычными словами при наличии замечаний.
7. **Коммит (Commit)**: фиксация изменений для сохранения результата и возможности лёгкого отката.

## Worked example

Учебное приложение проверяет ссылки в Markdown-файлах, но игнорирует ссылки с атрибутом title, например `[setup](setup.md "Setup guide")`. В этом примере вы исправите это поведение.

1. Запустите `claude` в учебной копии и нажимайте `Shift+Tab`, пока в строке статуса не появится `⏸ plan mode on`. Исследуйте кодовую базу:

   ```text
   How does this app find links in a Markdown file? Which kinds of links would it miss?
   ```

   Ваша точная формулировка может отличаться, но Claude укажет на `src/links.js` и отметит ссылки с атрибутом title.
2. Спланируйте задачу:

   ```text
   Plan a change so links with a title, like [setup](setup.md "Setup guide"), are checked too. Add a test for it to test/links.test.js.
   ```

   Внимательно прочтите план. Если в нём указаны только `src/links.js` и `test/links.test.js`, выберите **Yes, manually approve edits**. Если план затрагивает лишние файлы, верните его на доработку (как в [Уроке 2](02-permission-modes-and-plan-mode.md#worked-example)): в строке **Tell Claude what to change** попросите ограничить изменения только этими двумя файлами.
3. Реализация и запуск тестов. Одобряйте каждую правку. Claude также запрашивает подтверждение перед большинством команд (например, перед запуском тестов): внимательно читайте каждую команду. Отвечайте простым **Yes**, когда действие соответствует плану. Отвечайте **No** с указанием альтернативы, если команда делает лишнее (например, прячет изменения в stash, перезаписывает или перемещает файлы). Не выбирайте пункт **Yes, and switch to auto mode**: иначе фоновые проверки безопасности начнут принимать решения вместо вас на всех оставшихся шагах ([Choose a permission mode](https://code.claude.com/docs/en/permission-modes#switch-permission-modes)). Затем явно попросите запустить тесты (даже если Claude уже выполнил их ранее), чтобы лично увидеть результат:

   ```text
   run the tests
   ```

   Claude Code сворачивает длинный вывод команд в краткое резюме. Нажмите `Ctrl+O`, чтобы развернуть полный вывод ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#general-controls)), найдите итоговую строку тестов в конце и нажмите `Esc` для возврата. Должно отображаться `fail 0`.
4. Просмотр diff. Введите слэш-команду `/diff`, чтобы просмотреть изменения, сделанные Claude, наряду с другими незакоммиченными файлами ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#review-changes-with-%2Fdiff)). Откроется одно из двух представлений:
   - Боковая панель рядом с диалогом. Список скрывает тестовые файлы, пока вы не кликнете по строке с их количеством: кликните по ней. Панель может открываться автоматически при редактировании файлов: если она уже открыта, изучайте изменения в ней (повторный ввод `/diff` закроет панель).
   - Список файлов над полем ввода. Выберите файл клавишами со стрелками, нажмите `Enter` для просмотра diff и `Esc` для возврата.

   Проверьте каждую строку изменений, включая тестовый файл. Во втором терминале команда `git diff` выведет незакоммиченные изменения для уже отслеживаемых файлов git. Если Claude создал новый файл, предварительно выполните `git add -N <file>`, чтобы он отображался в diff ([git add](https://git-scm.com/docs/git-add#Documentation/git-add.txt--N)).
5. Корректировка. Прочтите добавленный тест. Если он покрывает только заголовки в двойных кавычках, отправьте запрос:

   ```text
   The test covers only double-quoted titles. Add a case for a single-quoted title, like [faq](faq.md 'FAQ'), and run the tests again.
   ```

   Если тест уже учитывает одинарные кавычки, попросите о другом небольшом улучшении (например, вынести проверку каждого типа кавычек в отдельный тест). После этого снова проверьте diff.
6. Коммит:

   ```text
   commit this with a message that says what changed
   ```

   Claude запросит подтверждение перед запуском коммита. Прочтите сформированную команду целиком и выберите **Yes**. Если инструмент создаст отдельную ветку, это допустимо: скрипт проверки работает на любой ветке. Если коммит требуется сохранить в текущей ветке, укажите это в тексте промпта. Затем во втором терминале сохраните хеш коммита для проверки и убедитесь в чистоте репозитория:

   ```bash
   git rev-parse HEAD > .practice/b-3-commit.txt
   git status
   ```

   Вывод `git status` должен завершаться строкой `nothing to commit, working tree clean`.

## Your turn

Повторите этот цикл для второй недоработки: ссылок, целевой путь которых заключён в угловые скобки, например `[my notes](<my notes.md>)`.

1. Исследуйте задачу в режиме plan mode: спросите, как приложение парсит целевой адрес ссылки.
2. Спланируйте изменение с добавлением теста в `test/links.test.js`. Отправьте план на доработку при попытке изменить лишние файлы, затем подтвердите через **Yes, manually approve edits**.
3. Подтверждайте правки и запуск тестов выбором **Yes**, затем попросите Claude выполнить тесты.
4. Просмотрите весь diff через `/diff`, включая файл с тестами.
5. Попросите внести хотя бы одну корректировку (пусть даже небольшую) и проверьте diff повторно.
6. Когда правка внесена, попросите Claude закоммитить код, тест и вашу корректировку единым коммитом. Затем во втором терминале выполните: `git rev-parse HEAD > .practice/b-3-commit.txt`.

В собственном репозитории выберите небольшое самостоятельное изменение, сопровождаемое тестом.

## Check

Задание считается выполненным, когда соблюдены все условия:

- [ ] Коммит, зафиксированный в `.practice/b-3-commit.txt`, изменяет тестовый файл и как минимум один файл исходного кода: `git show --stat $(cat .practice/b-3-commit.txt)`. **Windows:** `git show --stat (Get-Content .practice/b-3-commit.txt)`.
- [ ] Все тесты успешно проходят: команда `npm test` сообщает `fail 0`.
- [ ] В репозитории нет незакоммиченных изменений: `git status --porcelain` ничего не выводит.
- [ ] Self-check (not tested): вы полностью просмотрели diff перед коммитом и запросили у Claude как минимум одну корректировку.

Во втором терминале запустите команду `npm run check -- b-3` в учебной копии. В собственном репозитории выполните из учебной копии: `npm run check -- b-3 --dir <your repo>`. Если в целевом репозитории отсутствует скрипт `npm test`, предварительно сохраните код возврата вашей команды тестирования: `<your test command>; echo $? > .practice/b-3-tests.txt` (**Windows:** `<your test command>; $LASTEXITCODE > .practice/b-3-tests.txt`).

Если коммит не затрагивает тестовый файл (или содержит только тесты без исходного кода), значит, код и тесты не попали в один коммит: попросите Claude дополнить коммит недостающим тестом (`git commit --amend`) либо объединить два последних коммита в один (если вы ещё не отправляли их в удалённый репозиторий). Затем сохраните хеш заново через `git rev-parse HEAD > .practice/b-3-commit.txt`. Если тесты падают, прочтите первую ошибку и попросите Claude исправить код, а не ослаблять тест. Если `git status --porcelain` отображает файлы, закоммитьте их или отложите через `git stash push --include-untracked`.

## Go further

- [Common workflows](https://code.claude.com/docs/en/common-workflows): пошаговые руководства по исправлению багов, рефакторингу и написанию тестов.
- [Prompt library](https://code.claude.com/docs/en/prompt-library): готовые шаблоны промптов с разбивкой по задачам.
- [Give Claude a way to verify its work](https://code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work): почему реальный запуск тестов надёжнее текстового резюме.

---

<sub>Sources: [Best practices](https://code.claude.com/docs/en/best-practices) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Commands](https://code.claude.com/docs/en/commands) · [git add](https://git-scm.com/docs/git-add)</sub>

<sub>← [Permission modes and plan mode](02-permission-modes-and-plan-mode.md) · [Beginner index](README.md) · [Keep a session on track](04-keep-a-session-on-track.md) → · Topic: [Permissions and safety](../topics/permissions-and-safety.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-3)</sub>
