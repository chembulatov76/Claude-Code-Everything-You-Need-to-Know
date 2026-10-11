# Project memory with CLAUDE.md

<sub>**Beginner** · Lesson 5 of 5 · about 20 minutes · Needs: [Your first change, from request to commit](03-first-change.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого урока вы сможете создать файл `CLAUDE.md` для проекта и убедиться, что новая сессия соблюдает описанные в нём правила. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can create a project CLAUDE.md and show that a new session follows it.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений и Node.js LTS (в GitHub Codespace вашей копии всё уже готово). Чтобы практиковаться в собственном репозитории, сохраните также копию шаблона: проверка запускается из неё. Настройте папку `.practice/`, как описано в [уроке 1](01-install-and-look-around.md#you-need), если вы ещё этого не сделали.
- Расход токенов: минимальный (light).

## The idea

Каждая сессия начинается с чистого контекстного окна, из-за чего приходится снова и снова напоминать Claude о принятых в проекте правилах. Файл `CLAUDE.md` в корне проекта решает эту проблему: Claude Code автоматически считывает его в начале каждой сессии ([How Claude remembers your project](https://code.claude.com/docs/en/memory)). <details style="display:inline"><summary><code>[en]</code></summary>Each session starts with a fresh context window, so you can end up repeating your project's habits. A CLAUDE.md file at the project root fixes that: Claude Code reads it at the start of every session</details>

- Поместите в него то, о чём должна знать каждая сессия: команды сборки и запуска тестов, соглашения по коду, структуру репозитория и правила формата "всегда делай X". Держите файл компактным: официальная документация рекомендует укладываться в объём до 200 строк.
- Закоммитьте его в git, чтобы все участники проекта получали одинаковые инструкции.
- Команда `/init` автоматически генерирует первую версию файла на основе анализа кодовой базы.
- Claude воспринимает `CLAUDE.md` как контекст, а не как жёсткую обязательную конфигурацию. Правило, которое ни при каких обстоятельствах нельзя нарушать, требует использования хуков ([hooks](https://code.claude.com/docs/en/hooks-guide)), о которых рассказывается на [уровне Intermediate](../intermediate/README.md).

Сессия считывает `CLAUDE.md` только в момент запуска, поэтому после изменения этого файла начните новую сессию, чтобы гарантированно применить правки.

## Worked example

1. Запустите `claude` в вашей учебной копии и выполните команду:

   ```text
   /init
   ```

   Claude проанализирует код проекта и сформирует файл `CLAUDE.md` с найденными командами и правилами. Подтвердите создание файла, если появится запрос. Во втором терминале прочитайте его содержимое с помощью `cat CLAUDE.md`.
2. Откройте `CLAUDE.md` в редакторе кода или введите команду `/memory` и выберите пункт **Project instructions** (`./CLAUDE.md` в текущей папке). Не выбирайте пользовательский файл в `~/.claude/`: он влияет на все ваши проекты одновременно ([View and edit with `/memory`](https://code.claude.com/docs/en/memory#view-and-edit-with-%2Fmemory)). Терминальный редактор (например, Vim) займёт терминал до тех пор, пока вы не сохраните файл и не выйдете.

   В учебной копии команда `/init` может также описать служебные файлы курса: `checks/`, `capstone/` и `.github/`. Удалите упоминания о них и о команде `npm run check`, включая любое правило, запрещающее работать в ветке `main`: это правило относится к родительскому репозиторию шаблона, а не к вашей локальной копии. Сохраните остальные команды, включая `npm test`, которую проверяет итоговое задание ([capstone](capstone.md)).

   Затем добавьте правило, которое проявляется непосредственно в создаваемых файлах, чтобы легко проверить его исполнение. Разместите его под заголовком `## Conventions` (добавьте этот заголовок в конец файла, если `/init` его не создал):

   ```markdown
   - Every new source file starts with the line `// Part of linkcheck.`
   ```

   Во втором терминале сохраните точную строку правила в файл для запуска проверки:

   ```bash
   echo '// Part of linkcheck.' > .practice/b-5-rule.txt
   ```

   **Windows:** `'// Part of linkcheck.' | Set-Content .practice/b-5-rule.txt`
3. Сохраните `CLAUDE.md`, затем закоммитьте его во втором терминале:

   ```bash
   git add CLAUDE.md
   git commit -m "Add project instructions for Claude Code"
   ```

4. Введите `/exit` и запустите новую сессию с помощью `claude`, чтобы агент загрузил обновлённый `CLAUDE.md` с вашим правилом. Выполните `/context all` и найдите `CLAUDE.md` в блоке **Memory files**: сессия успешно его загрузила. Базовая команда `/context` может свернуть этот список, а параметр `all` разворачивает его полностью ([Commands](https://code.claude.com/docs/en/commands)).
5. Попросите агента выполнить изменение, создающее новый файл:

   ```text
   Add src/stats.js with a function countLinks(markdown) that returns how many links findLinks finds. Add a test in test/stats.test.js, run the tests and commit.
   ```

   Подтвердите запросы разрешений, которые запросит Claude.
6. Во втором терминале проверьте первую строку созданного файла, сохраните хеш коммита для проверки и убедитесь в чистоте рабочей директории:

   ```bash
   head -1 src/stats.js
   git rev-parse HEAD > .practice/b-5-commit.txt
   git status
   ```

   **Windows:** замените `head -1` на `Get-Content src/stats.js -TotalCount 1`.

   Первая строка файла должна содержать `// Part of linkcheck.`, хотя в текущей сессии вы об этом явно не просили. Команда `git status` должна сообщить: `nothing to commit, working tree clean`. Если в выводе отображается `src/stats.js`, Claude ещё не закоммитил изменения: попросите его сделать коммит, после чего повторно сохраните хеш коммита.

## Your turn

Повторите пример в учебной копии или в собственном репозитории с новым правилом и другим файлом. В учебной копии файл `CLAUDE.md` уже существует, поэтому шаг `/init` можно пропустить:

1. Откройте `CLAUDE.md` и замените правило из примера на новое:

   ```markdown
   - Every new source file starts with the line `// linkcheck source.`
   ```

2. Во втором терминале сохраните точную строку нового правила: `echo '// linkcheck source.' > .practice/b-5-rule.txt` (**Windows:** `'// linkcheck source.' | Set-Content .practice/b-5-rule.txt`).
3. Закоммитьте `CLAUDE.md` в git.
4. Выйдите из сессии, запустите новую сессию и убедитесь, что `/context all` выводит `CLAUDE.md` в блоке **Memory files**.
5. Попросите создать новый файл и тест к нему, позволив Claude закоммитить изменения:

   ```text
   Add src/external.js with a function countExternal(markdown) that returns how many of the links findLinks finds are external, according to isExternal. Add a test in test/external.test.js, run the tests and commit.
   ```

6. Как и в шаге 6 примера, проверьте первую строку `src/external.js`, сохраните коммит в `.practice/b-5-commit.txt` и выполните `git status`.

В собственном репозитории начните с `/init`; если в репозитории уже есть `CLAUDE.md`, `/init` предложит улучшения вместо перезаписи. Напишите правило в стиле комментариев используемого вами языка программирования (например, `# Part of my-project.`), сохраните эту точную строку в `.practice/b-5-rule.txt` на шаге 2 и попросите создать небольшой файл. Если в репозитории уже используется `AGENTS.md`, поместите `@AGENTS.md` на первой строке `CLAUDE.md`: по умолчанию, как только появляется `CLAUDE.md`, Claude читает его вместо `AGENTS.md` ([AGENTS.md](https://code.claude.com/docs/en/memory#agents-md)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный `CLAUDE.md` содержит правило, сохранённое в `.practice/b-5-rule.txt`: команда `git show HEAD:CLAUDE.md` отображает его.
- [ ] Коммит из `.practice/b-5-commit.txt` добавляет как минимум один файл (кроме тестов и Markdown), и каждый такой файл начинается с этого правила: `git show --stat $(cat .practice/b-5-commit.txt)` выводит эти файлы. **Windows:** `git show --stat (Get-Content .practice/b-5-commit.txt)`.
- [ ] Self-check (not tested): вы перезапустили сессию после добавления правила в `CLAUDE.md` и можете объяснить, почему это необходимо.

Запустите `npm run check -- b-5` в учебной копии. В собственном репозитории выполните из учебной копии: `npm run check -- b-5 --dir <your repo>`.

Если `CLAUDE.md` не закоммичен, выполните `git add CLAUDE.md` и сделайте коммит. Если новый файл не начинается с заданного правила, запустите новую сессию и проверьте вывод `/context all`. Когда `CLAUDE.md` отсутствует в блоке **Memory files**, сессия его не видит: убедитесь, что запускаете `claude` в той папке, где находится `CLAUDE.md`. Если файл отображается в списке, значит, Claude прочитал правило, но не применил его: попросите сессию создать другой файл (например, `src/summary.js` с тестом) и закоммитить его, затем сохраните хеш коммита командой `git rev-parse HEAD > .practice/b-5-commit.txt` и запустите проверку снова. Правка старого файла на месте не поможет пройти тест, так как проверка анализирует только файлы, добавленные указанным коммитом.

## Go further

- [Write an effective CLAUDE.md](https://code.claude.com/docs/en/best-practices#write-an-effective-claude-md): рекомендации по наполнению файла и исключению лишнего контекста.
- [Using CLAUDE.md files](https://claude.com/blog/using-claude-md-files): руководство Anthropic с наглядными практическими примерами.

---

<sub>Sources: [How Claude remembers your project](https://code.claude.com/docs/en/memory) · [Commands](https://code.claude.com/docs/en/commands) · [Best practices](https://code.claude.com/docs/en/best-practices) · [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide)</sub>

<sub>← [Keep a session on track](04-keep-a-session-on-track.md) · [Beginner index](README.md) · [Beginner capstone](capstone.md) → · Topic: [Memory and context](../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-5)</sub>
