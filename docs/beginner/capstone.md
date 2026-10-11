# Beginner capstone: fix a reported bug, from plan to commit

<sub>**Beginner** · Capstone · 30 to 60 minutes · Needs: the five [Beginner lessons](README.md), or their skills · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Это задание объединяет все навыки уровня Beginner. В отличие от стандартных уроков, здесь нет пошагового разобранного примера: шаги описывают, что нужно сделать, а формулировку промптов вы выбираете самостоятельно. Раздел [Beginner Exit](README.md#by-the-end-of-this-level-you-can) перечисляет умения, которыми вы овладеете к концу этого уровня. Скрипт проверки анализирует итоговые файлы и коммит, а оставшиеся пункты вы отмечаете самостоятельно в разделе [What the check can't see](#what-the-check-cant-see). Когда выполнены обе части, требования Beginner Exit считаются подтверждёнными независимо от того, проходили ли вы отдельные уроки. <details style="display:inline"><summary><code>[en]</code></summary>This task puts the whole Beginner level together. Unlike the lessons, it has no worked example: the steps say what to do, and you choose the prompts. The Beginner Exit is the list of what you can do by the end of this level.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений и Node.js LTS (в GitHub Codespace вашей копии всё уже установлено). Если команда `git status --porcelain` выводит файлы, закоммитьте их или отложите с помощью `git stash push --include-untracked`. Итоговое задание выполняется только в учебном шаблоне.
- Если вы пропустили уроки: создайте папку `.practice/` в вашей копии (`mkdir -p .practice`; **Windows:** `New-Item -ItemType Directory -Force .practice`). В шаблоне этот каталог уже добавлен в `.gitignore`.
- Закоммиченный файл `CLAUDE.md` в вашей копии, где указана команда запуска тестов `npm test`: команда `git show HEAD:CLAUDE.md` должна её показывать. Урок 5 создаёт такой файл; если в вашем файле нет команды `npm test`, добавьте её.
- Если вы пропустили урок 5 или выполняли его только в собственном репозитории: запустите `claude` в вашей учебной копии и выполните `/init`. В созданном файле `CLAUDE.md` удалите упоминания о внутренних файлах курса (`checks/`, `capstone/` и `.github/`) и о команде `npm run check`, включая любое правило, запрещающее работать в `main`: это правило нужно только репозиторию-шаблону, а не вашей копии. Убедитесь, что в файле присутствует команда `npm test`.
- Закоммитьте любые изменения в `CLAUDE.md` отдельным коммитом перед началом работы (`git add CLAUDE.md`, затем `git commit -m "Add CLAUDE.md"`): коммит с исправлением бага должен содержать изменения только в тех файлах, которые заявлены в плане.
- Расход токенов: умеренный (moderate).

> [!NOTE]
> В нативной Windows выполняйте все команды в PowerShell. Команды без отдельной строки **Windows** работают в PowerShell без изменений; если под командой указана строка **Windows**, используйте её. Если комбинация `Shift+Tab` не срабатывает, нажмите `Alt+M` ([Interactive mode](https://code.claude.com/docs/en/interactive-mode)).

## The task

Коллега создал отчёт об ошибке в файле `capstone/beginner-bug.md` в вашей учебной копии: если в файле конфигурации `linkcheck.json` указана несуществующая папка, инструмент ошибочно сообщает, что все ссылки корректны, и завершает работу с кодом возврата 0. <details style="display:inline"><summary><code>[en]</code></summary>A teammate filed the bug report in capstone/beginner-bug.md in your practice copy: when linkcheck.json lists a folder that doesn't exist, the tool says every link resolves and exits with code 0.</details>

Чтобы воспроизвести баг вручную, создайте новую папку вне вашей учебной копии и сохраните в ней файл `linkcheck.json` с содержимым `{ "files": ["docs"] }`. Затем запустите `npm run linkcheck -- <that folder>` в учебной копии: пока баг не исправлен, утилита выводит `All relative links resolve.` Не создавайте файл `linkcheck.json` внутри вашей учебной копии, иначе скрипт проверки посчитает это изменением репозитория. **Windows:** сохраняйте файл с помощью `'{ "files": ["docs"] }' | Set-Content <that folder>\linkcheck.json`, а не через оператор `>`. В Windows PowerShell 5.1 оператор `>` записывает файл в кодировке UTF-16 ([about_Character_Encoding](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_character_encoding?view=powershell-5.1)), из-за чего инструмент отклонит файл как некорректный JSON с кодом возврата 2 ещё до исправления.

Исправьте ошибку по методике уроков уровня Beginner:

1. Сохраните вывод команды `claude --version` в файл `.practice/version.txt`.
2. Запустите сессию и переключитесь в режим планирования (plan mode) перед отправкой первого промпта. Затем попросите Claude прочитать баг-репорт и составить план исправления. Исправление затрагивает два исходных файла и один файл тестов. Попросите составить план так, чтобы сначала был добавлен тест и запущен до изменения кода, подтверждая воспроизведение бага падением теста ([Best practices](https://code.claude.com/docs/en/best-practices#provide-specific-context-in-your-prompts): "write a failing test that reproduces the issue, then fix it").
3. Пока итоговый план находится на экране, откройте второй терминал в той же папке. Сохраните вывод `git status --porcelain` в файл `.practice/capstone-before.txt`. Сохраните список файлов, которые план собирается изменить (включая файл тестов), в `.practice/capstone-plan.txt`: по одному относительному пути от корня репозитория на строку с прямыми слэшами, например `src/cli.js`.
4. Утвердите план. Убедитесь, что новый тест сначала падает с ошибкой, а после применения правок успешно проходит.
5. Изучите весь diff целиком (включая файл тестов), как в [уроке 3](03-first-change.md#worked-example). Команда `git diff` не показывает новый файл, пока вы не выполните `git add -N <file>` ([git add](https://git-scm.com/docs/git-add#Documentation/git-add.txt--N)). Попросите Claude внести как минимум одно уточнение или исправление, не выходя за рамки запланированных файлов, и убедитесь, что тесты `npm test` проходят.
6. Закоммитьте исправление вместе с тестом одним коммитом, после чего сохраните хеш коммита: `git rev-parse HEAD > .practice/capstone-commit.txt`. Проверка считывает только этот коммит, поэтому при внесении последующих изменений не забудьте обновить файл.

Контролируйте ход сессии по ходу работы: проверяйте `/context`, используйте `/compact` или `/rewind`, когда это необходимо.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Файл `.practice/version.txt` содержит вывод команды `claude --version`.
- [ ] Файл `.practice/capstone-before.txt`, сохранённый во время показа плана на экране, не содержит изменённых файлов.
- [ ] Коммит из `.practice/capstone-commit.txt` изменяет только те файлы, которые перечислены в `.practice/capstone-plan.txt`, включая файл теста.
- [ ] Ошибка исправлена: с файлом `linkcheck.json` из баг-репорта в папке вне учебной копии команда `npm run linkcheck -- <that folder>` завершается с кодом возврата 2.
- [ ] Тесты `npm test` проходят успешно.
- [ ] Закоммиченный файл `CLAUDE.md` содержит команду запуска тестов `npm test`.
- [ ] Команда `git status --porcelain` ничего не выводит (рабочее дерево чистое).

Запустите `npm run check -- b-capstone` в вашей учебной копии. Каждый непройденный пункт выведет подсказку с тем, что требуется исправить.

## What the check can't see

Автоматическая проверка не может оценить следующие навыки из Beginner Exit. Отметьте их самостоятельно и честно:

- [ ] Я могу определить с помощью `/status`, какую модель и учётную запись использует текущая сессия.
- [ ] Я могу переключать режимы разрешений комбинацией `Shift+Tab` и понимаю, какой режим требуется для конкретной задачи.
- [ ] Я внимательно просмотрел весь diff перед коммитом и попросил Claude внести хотя бы одну корректировку.
- [ ] Я умею удерживать сессию в рабочем русле с помощью `/context`, `/clear`, `/compact`, `/rewind` и `/resume`.
- [ ] Я видел, как новая сессия следует правилу из `CLAUDE.md`, которое не упоминалось в промпте. Если вы пропустили урок 5, в его [Worked example](05-project-memory.md#worked-example) показано, как это работает.

Когда каждый пункт на этой странице отмечен галочкой, вы готовы к переходу на уровень [Intermediate](../intermediate/README.md). Его список предварительных требований как раз и составляет Beginner Exit.

Если этот уровень вам помог, звёздочка репозиторию поможет другим разработчикам найти это руководство.

---

<sub>Sources: [Best practices](https://code.claude.com/docs/en/best-practices) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [git add](https://git-scm.com/docs/git-add) · [about_Character_Encoding](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_character_encoding?view=powershell-5.1) · the [Beginner lessons](README.md), which cite the official pages for each step.</sub>

<sub>← [Project memory with CLAUDE.md](05-project-memory.md) · [Beginner index](README.md) · [Intermediate index](../intermediate/README.md) → · [Stuck on this capstone?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-capstone)</sub>
