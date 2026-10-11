# Organize project memory and see what loaded

<sub>**Intermediate** · Lesson 2 of 8 · about 20 minutes · Needs: [Turn a repeated workflow into a skill](01-first-skill.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого урока вы сможете разделить инструкции проекта на импорты и правила с ограниченной областью действия (scoped rules), а также определять, какие файлы инструкций были фактически загружены в контекст, если Claude не следует указаниям. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can split a project's instructions into imports and scoped rules, and find out which instruction files loaded when Claude doesn't follow one.</details>

## You need

- Один из ваших рабочих репозиториев с закоммиченным файлом `CLAUDE.md` (в состоянии после [урока 5 уровня Beginner](../beginner/05-project-memory.md)), а также ваша копия учебного шаблона с Node.js LTS (проверка запускается из неё). Для выполнения обоих условий можно использовать саму копию шаблона. Настройте папку `.practice/`, как описано в [уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если вы ещё этого не сделали.
- Утилита `jq` для разбора журнала загрузок в разобранном примере.
- Расход токенов: минимальный (light).

## The idea

Слишком длинный файл `CLAUDE.md` трудно поддерживать, и каждая его строка загружается в абсолютно каждую сессию, расходуя контекст. Два механизма позволяют структурировать инструкции ([How Claude remembers your project](https://code.claude.com/docs/en/memory)): <details style="display:inline"><summary><code>[en]</code></summary>A long CLAUDE.md is hard to maintain, and every line of it loads into every session. Two tools split it up</details>

- **Импорт (import)** - строка вида `@docs/conventions.md` внутри `CLAUDE.md`, которая подгружает другой файл вместе с основным. Импорт организует структуру файлов, но не экономит контекст.
- **Правило (rule)** - Markdown-файл в папке `.claude/rules/`. При наличии параметра `paths:` во frontmatter правило подгружается только тогда, когда Claude работает с соответствующим файлом.

| Файл инструкций | Когда загружается |
|---|---|
| `CLAUDE.md` в корне проекта и правила без параметра `paths:` | В момент старта сессии |
| Файл, указанный через импорт `@path` | Вместе с тем файлом, который его импортирует |
| Правило с параметром `paths:` | Когда Claude считывает соответствующий файл инструментом Read |
| Автопамять (Auto memory) - заметки, сохраняемые Claude | В момент старта сессии: первые 200 строк или 25 КБ индекса `MEMORY.md` |

Когда Claude игнорирует инструкцию, первым делом проверьте, был ли загружен соответствующий файл в контекст. Если файл загружен, причину стоит искать в формулировке: сделайте её конкретнее и удалите противоречащие указания ([Debug your configuration](https://code.claude.com/docs/en/debug-your-config#see-what-loaded-into-context)).

## Worked example

1. В вашем репозитории добавьте директиву импорта в `CLAUDE.md`. В учебном шаблоне импортируйте `package.json`; в собственном репозитории - существующий документ команды, например `@docs/architecture.md`:

   ```markdown
   - npm scripts: @package.json
   ```

   Путь импорта считывается до первого пробела, поэтому не ставьте точку непосредственно после пути к файлу.
2. Сохраните правило для тестовых файлов как `.claude/rules/testing.md`. Настройте шаблон под структуру папок ваших тестов:

   ```markdown
   ---
   paths:
     - "test/**/*.js"
   ---

   # Testing

   - Use `node:test` and `node:assert/strict`; no other test libraries.
   ```

3. Сохраните конфигурацию хука логирования в `.practice/i-2-hook.json`. Это хук события [`InstructionsLoaded`](https://code.claude.com/docs/en/hooks#instructionsloaded): каждый раз при загрузке `CLAUDE.md`, правила или импортированного файла он записывает причину и путь к файлу в `.practice/i-2-loaded.txt`. Он ничего больше не логирует и срабатывает только в тех сессиях, где явно указан:

   ```json
   {
     "hooks": {
       "InstructionsLoaded": [
         { "hooks": [ { "type": "command", "command": "jq -r '[.load_reason, .file_path] | @tsv' >> \"$CLAUDE_PROJECT_DIR/.practice/i-2-loaded.txt\"" } ] }
       ]
     }
   }
   ```

4. Запустите сессию с логированием: `claude --settings .practice/i-2-hook.json` ([CLI reference](https://code.claude.com/docs/en/cli-reference#cli-flags)) и выполните `/context all`. Базовый `/context` может сворачивать список, а параметр `all` разворачивает его ([Commands](https://code.claude.com/docs/en/commands)). В блоке **Memory files** перечисляются файлы инструкций, загруженные при старте (`CLAUDE.md` и др.) ([Set up a project CLAUDE.md](https://code.claude.com/docs/en/memory#set-up-a-project-claude-md)). Локализованного правила там нет: сессия ещё не читала тестовые файлы.
5. Попросите Claude прочитать тестовый файл с помощью инструмента Read. Назовите инструмент явно: иногда модель считывает файлы через команды оболочки, что не активирует scoped rules.

   ```text
   Use your Read tool to read test/config.test.js, then tell me in one sentence what it checks.
   ```

   Во втором терминале команда `cat .practice/i-2-loaded.txt` покажет каждую загрузку и её причину: `session_start` для `CLAUDE.md`, `include` для `package.json`, и теперь `path_glob_match` для нашего правила.

## Your turn

Правило коллеги не срабатывает на практике. Найдите причину и исправьте её.

1. Сохраните следующее правило как `.claude/rules/cli.md`. В собственном репозитории напишите правило для одной из ваших рабочих папок и намеренно сделайте опечатку в названии папки в паттерне:

   ```markdown
   ---
   paths:
     - "source/**/*.js"
   ---

   # CLI output

   - Print errors to stderr, starting with `linkcheck:`.
   ```

2. Запустите сессию с логированием `claude --settings .practice/i-2-hook.json` и попросите Claude использовать Read на файле `src/cli.js` (или файле из выбранной папки).
3. Проверьте лог: правило не было загружено. Сравните его шаблон с существующими файлами: `git ls-files src`.
4. Исправьте паттерн, запустите новую сессию с флагом логирования и повторите шаг 2. В логе появится строка `path_glob_match` для `cli.md`.
5. Закоммитьте `CLAUDE.md` и `.claude/rules/`. Удалите `.practice/i-2-hook.json`, когда журнал больше не нужен.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный `CLAUDE.md` импортирует реально существующий файл: команда `git show HEAD:CLAUDE.md | grep '@'` показывает этот импорт.
- [ ] Каждое закоммиченное правило с `paths:` соответствует хотя бы одному файлу в проекте: для каждого паттерна команда `git ls-files ':(glob)<pattern>'` находит файлы. С префиксом `:(glob)` символ `*` в git не выходит за пределы папки, а `**/` соответствует любому числу каталогов, аналогично правилам Claude ([gitglossary: pathspec](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefpathspecapathspec)). Git не раскрывает фигурные скобки, в отличие от правил Claude ([Path-specific rules](https://code.claude.com/docs/en/memory#path-specific-rules)), поэтому шаблоны вроде `src/**/*.{ts,tsx}` проверяйте отдельно по расширениям.
- [ ] Лог фиксирует загрузку импорта и правила по шаблону: `grep -E '^(include|path_glob_match)' .practice/i-2-loaded.txt` выводит как минимум по одной строке каждого типа.
- [ ] Self-check (not tested): вы можете объяснить, какие файлы инструкций загружаются на старте, а какие позже, и как вы увидели загрузку каждого из них.

Запустите `npm run check -- i-2 --dir <your repo>` из учебной копии, либо `npm run check -- i-2` в самой копии.

Если проверка импорта не проходит, проверьте, нет ли знаков препинания сразу после пути или не указан ли путь относительно неверной директории: путь импорта вычисляется относительно файла, содержащего директиву. Если правило не находит файлов, помните, что путь задаётся относительно корня репозитория (например, `source/**` ничего не найдёт в проекте с папкой `src/`). Если в логе нет строки `path_glob_match`, запустите сессию с `--settings .practice/i-2-hook.json` и запросите вызов инструмента Read по имени.

## Watch out

- Импорт, указывающий за пределы рабочей директории (например, `@~/notes.md`), запрашивает подтверждение при первом использовании в проекте; в случае отказа он остаётся отключённым ([Import additional files](https://code.claude.com/docs/en/memory#import-additional-files)).
- Claude Code считывает файл проекта `CLAUDE.md` один раз на старте. Изменения в нём попадут в текущую сессию после команд `/clear`, `/compact` или перезапуска ([Editing CLAUDE.md mid-session](https://code.claude.com/docs/en/prompt-caching#editing-claude-md-mid-session)).
- Для поиска противоречащих друг другу инструкций или ссылок на удалённые файлы выполните `/doctor prompt-audit` ([Audit your instruction files](https://code.claude.com/docs/en/memory#audit-your-instruction-files)). Утилита предлагает исправления и ничего не меняет без вашего согласия.

## Go further

- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory): файлы в папках `.claude/` и `~/.claude`, и момент загрузки каждого из них.
- [Debug your configuration](https://code.claude.com/docs/en/debug-your-config): диагностика ситуаций, когда настройка, хук или MCP-сервер не применяются.
- [Auto memory](https://code.claude.com/docs/en/memory#auto-memory): что Claude сохраняет для себя автоматически, где хранятся эти записи и как отключить автосохранение.

---

<sub>Sources: [How Claude remembers your project](https://code.claude.com/docs/en/memory) · [Hooks reference](https://code.claude.com/docs/en/hooks) · [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) · [How Claude Code uses prompt caching](https://code.claude.com/docs/en/prompt-caching) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Commands](https://code.claude.com/docs/en/commands) · [gitglossary](https://git-scm.com/docs/gitglossary)</sub>

<sub>← [Turn a repeated workflow into a skill](01-first-skill.md) · [Intermediate index](README.md) · [Pick the model and effort](03-model-and-effort.md) → · Topic: [Memory and context](../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-2)</sub>
