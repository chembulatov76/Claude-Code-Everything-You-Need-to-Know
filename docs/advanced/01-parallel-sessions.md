# Parallel sessions with worktrees

<sub>**Advanced** · Lesson 1 of 7 · about 20 minutes · Needs: [Advanced index](README.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете одновременно запускать две сессии Claude Code, каждую в собственном git worktree, и объединять результаты их работы в одну общую ветку. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can run two Claude Code sessions at once, each in its own git worktree, and merge their work into one branch.</details>

## You need

- Один из ваших репозиториев с настроенной командой тестов и без незакоммиченных изменений, открытый на ветке по умолчанию и синхронизированный с удаленным репозиторием, в который вы можете делать push (раздел Предостережения поясняет причины), а также копия учебного шаблона с Node.js LTS и проверками уровня Advanced ([Практика](README.md#practice)): проверка запускается из шаблона. Настройте папку `.practice/` в соответствии с описанием в [Уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если еще не сделали этого.
- Если вы ранее не запускали `claude` в этом репозитории, запустите его один раз и примите диалог доверия: до этого вызов `claude --worktree` завершится с ошибкой ([Start Claude in a worktree](https://code.claude.com/docs/en/worktrees#start-claude-in-a-worktree)).
- macOS, Linux или WSL 2, а также три открытых терминала в корне репозитория: два для сессий Claude и один для команд git.
- Расход токенов: умеренный (moderate). Две сессии работают одновременно.

## The idea

[git worktree](https://git-scm.com/docs/git-worktree) представляет собой дополнительную рабочую директорию на отдельной ветке, разделяющую историю коммитов репозитория. Команда `claude --worktree <name>` (или `-w <name>`) создает такую папку в `.claude/worktrees/<name>/` на новой ветке `worktree-<name>` и запускает в ней сессию ([Start Claude in a worktree](https://code.claude.com/docs/en/worktrees#start-claude-in-a-worktree)). Каждая сессия редактирует свою копию файлов. Кроме того, Claude Code предотвращает изменение основной рабочей копии инструментами сессии worktree и блокирует запуск команд, направленных на нее ([How Claude Code enforces isolation](https://code.claude.com/docs/en/worktrees#how-claude-code-enforces-isolation)).

Коммиты и ветки являются общими. Незакоммиченные правки, установленные зависимости и файлы из `.gitignore` (например, `.env`) не переносятся: каждый worktree является чистым чекаутом. Файл `.worktreeinclude` в корне репозитория позволяет копировать перечисленные в нем игнорируемые файлы в каждый новый worktree ([Copy gitignored files into worktrees](https://code.claude.com/docs/en/worktrees#copy-gitignored-files-into-worktrees)); указывайте в нем только действительно необходимые файлы, так как каждый worktree получает собственную копию каждого секрета.

Результаты объединяются через git: каждая сессия делает коммит в своей ветке, вы сливаете ветки из основного чекаута, после чего удаляете рабочие деревья worktree.

## Worked example

1. В основном рабочем дереве добавьте директорию рабочих деревьев в игнорирование, после чего закоммитьте и отправьте правило. Без этого содержимое worktree будет отображаться как неотслеживаемые файлы в основном дереве. Отправка в remote переносит правило в ветку по умолчанию, от которой создаются новые worktrees (подробнее в разделе Предостережения):

   ```bash
   printf '\n.claude/worktrees/\n' >> .gitignore
   git add .gitignore && git commit -m "Ignore Claude Code worktrees"
   git push
   ```

   Символ `\n` в начале помещает правило на отдельную строку, даже если ваш `.gitignore` не заканчивался переводом строки. После этого `git status --short` ничего не выводит, а `git log -1 --oneline` показывает созданный коммит.
2. В первом терминале запустите сессию в новом worktree и поставьте задачу по документации:

   ```bash
   claude --worktree docs-pass
   ```

   ```text
   Tidy the wording of the README's first section, then wait for me before you commit.
   ```

   Claude Code создаст `.claude/worktrees/docs-pass/` на ветке `worktree-docs-pass`, и сессия начнет работу в ней. Подтвердите редактирование при запросе. Просьба подождать удерживает коммит до шага 4 даже в тех случаях, когда запрос разрешений не появляется: сессия worktree использует файл `.claude/settings.local.json` из основного чекаута ([Settings files and precedence](https://code.claude.com/docs/en/settings#where-claude-code-keeps-the-local-file-in-a-git-repository)), поэтому при включенном режиме auto-allow песочницы из [Урока 4 уровня Intermediate](../intermediate/04-permissions-and-sandbox.md) изолированные команды, такие как `git commit`, выполняются без подтверждения ([Sandbox modes](https://code.claude.com/docs/en/sandboxing#sandbox-modes)).
3. Во втором терминале запустите вторую сессию с коротким флагом и дайте задачу по тестам:

   ```bash
   claude -w test-pass
   ```

   ```text
   This worktree is a fresh checkout: install the dependencies, then add one missing test for a small exported function and run the tests. Wait for me before you commit.
   ```

   Claude Code создаст `.claude/worktrees/test-pass/` на ветке `worktree-test-pass`; сессия сначала установит зависимости, а затем добавит тест. Подтверждайте действия аналогичным образом.
4. В третьем терминале в основном рабочем дереве осмотрите все три директории, пока обе сессии ожидают вашего указания:

   ```bash
   git worktree list
   git status --short
   git -C .claude/worktrees/docs-pass status --short
   git -C .claude/worktrees/test-pass status --short
   ```

   Команда `git worktree list` сначала показывает основной чекаут, затем `.claude/worktrees/docs-pass` на `[worktree-docs-pass]` и `.claude/worktrees/test-pass` на `[worktree-test-pass]`; git может помечать их как `locked` на время работы сессии. Команда `git status --short` не отображает правки ни одной из сессий, так как они происходят в изолированных папках. Каждая команда `git -C` показывает изменения только сессии в указанной папке (например, README в `docs-pass`) и никогда не затрагивает другую сессию. Теперь скажите каждой сессии закоммитить изменения: команда `git log --oneline --graph --all` отобразит по новому коммиту в каждой ветке.
5. Находясь в основном чекауте, слейте обе ветки и запустите тесты:

   ```bash
   git merge worktree-docs-pass
   git merge worktree-test-pass
   ```

   Команда [`git merge`](https://git-scm.com/docs/git-merge) переносит коммиты каждой ветки в вашу текущую ветку. Первое слияние будет перемещением указателя вперед (fast-forward): ветка `worktree-docs-pass` началась с коммита, отправленного на шаге 1, поэтому в вашей ветке нет недостающих коммитов. Второе слияние создаст мердж-коммит, так как в ветке теперь есть коммит, отсутствующий в `worktree-test-pass`, и откроет текстовый редактор для сообщения коммита: сохраните и закройте его. Выполняйте этот шаг самостоятельно: пока сессия изолирована в worktree, Claude Code блокирует команды git, обращающиеся к основному дереву. Если обе задачи затронули одни и те же строки, git остановится с конфликтом: разрешите его вручную или запустите `claude` в основном дереве и попросите помочь с разрешением.
6. В каждой сессии выполните команду `/exit`. Так как ветка worktree содержит новый коммит, Claude Code спросит, сохранить или удалить рабочее дерево: выберите remove ([Clean up worktrees](https://code.claude.com/docs/en/worktrees#clean-up-worktrees)). Команда `git worktree list` больше не будет показывать удаленные деревья.

> [!CAUTION]
> Удаление worktree стирает его директорию и его ветку со всеми изменениями, которые вы еще не успели слить. Сначала выполните merge или выберите вариант keep.

## Your turn

Выберите два небольших изменения в вашем репозитории, затрагивающих разные файлы: например, фикс в одном модуле и недостающий тест для другого. Выполните каждое в собственной сессии worktree при их одновременной работе и объедините обе ветки в свою основную ветку.

Если работа, на которой они должны базироваться, еще не отправлена в ветку по умолчанию удаленного репозитория (см. Предостережения), сначала отправьте ее туда или добавьте `"worktree": {"baseRef": "head"}` в `.claude/settings.local.json` (ваш персональный файл настроек из [Урока 4 уровня Intermediate](../intermediate/04-permissions-and-sandbox.md)), чтобы новые worktrees стартовали от вашего локального `HEAD` ([`worktree.baseRef`](https://code.claude.com/docs/en/settings-reference#worktree-baseref)).

Критерии приемки:

- Закоммиченный файл `.gitignore` игнорирует `.claude/worktrees/` и не игнорирует остальную часть `.claude/`.
- Каждое изменение выполнено в собственной сессии, запущенной через `claude --worktree <name>`, и закоммичено в ветку этого worktree.
- Файл `.practice/a-1-worktrees.txt` содержит вывод команды `git worktree list --porcelain`, сохраненный из основного дерева, когда оба worktree существовали и в обоих были сделаны коммиты:

  ```bash
  git worktree list --porcelain > .practice/a-1-worktrees.txt
  ```

- Обе ветки влиты в вашу ветку с помощью обычной команды `git merge`, а не [`git merge --squash`](https://git-scm.com/docs/git-merge#Documentation/git-merge.txt---squash).
- Ваши тесты проходят после обоих слияний (не проверяется скриптом).
- Оба worktree удалены, а любая сохраненная ветка удаляется только после того, как она была слита.
- Опционально, не проверяется: если для запуска проекта необходим игнорируемый git файл (например, локальный конфиг), укажите его в файле `.worktreeinclude` в корне репозитория.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный `.gitignore` игнорирует `.claude/worktrees/` и не затрагивает остальное содержимое `.claude/`.
- [ ] `.practice/a-1-worktrees.txt` перечисляет два worktree в папке `.claude/worktrees/`, коммиты которых разошлись, и оба влиты в вашу ветку.
- [ ] Ни одно из этих рабочих деревьев больше не существует: `git worktree list` не выводит их.
- [ ] Самопроверка (не тестируется): во время одновременной работы ни одна сессия не видела незакоммиченных правок другой.

Запустите команду `npm run check -- a-1 --dir <your repo>` из вашей учебной копии шаблона.

Если первый пункт не проходит, выполните `git -c core.excludesFile=/dev/null check-ignore -v .claude/worktrees/x` в вашем репозитории (параметр `-c` временно подменяет глобальный файл исключений [`core.excludesFile`](https://git-scm.com/docs/gitignore) на пустой, как это делает скрипт проверки): команда должна вывести имя вашего `.gitignore` и совпавшую строку ([git-check-ignore](https://git-scm.com/docs/git-check-ignore#Documentation/git-check-ignore.txt---verbose)). Правило только в `.git/info/exclude` или глобальном файле не является закоммиченным, а слишком широкое правило вроде `.claude/` скроет настройки и навыки, которые вы коммитите для команды. Если не проходит второй пункт, прочтите подсказку проверки: сохраняйте файл после коммитов в обеих сессиях и до удаления любого worktree, и выполняйте слияние веток через `git merge`. Если не проходит третий пункт, выполните `git worktree remove <path>` из основного дерева для каждого пути из ошибки (с предварительной разблокировкой `git worktree unlock <path>`, если git сообщает о блокировке). После этого останется сама ветка: удалите ее после слияния командой `git merge-base --is-ancestor worktree-<name> HEAD && git branch -D worktree-<name>`. Первая команда завершается успешно только тогда, когда ветка уже стала частью `HEAD` ([git-merge-base](https://git-scm.com/docs/git-merge-base#Documentation/git-merge-base.txt---is-ancestor)), а флаг `-D` удаляет ветку без проверки upstream в git ([git-branch](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt--D)).

## Watch out

- В большинстве репозиториев на macOS, Linux и WSL 2 выбор варианта "Yes, and don't ask again" для команды Bash в сессии worktree сохраняет правило в файл `.claude/settings.local.json` основного рабочего дерева. В результате оно действует и в основном дереве, и во всех worktrees репозитория, оставаясь в силе даже после удаления worktree ([What worktrees share with the main checkout](https://code.claude.com/docs/en/worktrees#what-worktrees-share-with-the-main-checkout)). Разрешайте на постоянной основе только те команды, которые безопасны в любом месте проекта.
- Одновременный запуск нескольких сессий пропорционально увеличивает расход токенов ([Run agents in parallel](https://code.claude.com/docs/en/agents)). Давайте каждой сессии одну небольшую задачу и завершайте сессию сразу после слияния ее ветки.
- Новый worktree ответвляется от ветки по умолчанию удаленного репозитория, а не от вашего локального `HEAD` ([Choose the base branch](https://code.claude.com/docs/en/worktrees#choose-the-base-branch)). Коммиты, которых еще нет в этой удаленной ветке (неотправленные коммиты или работа вашей текущей ветки), не войдут в worktree, а слияние его ветки в ветку, отстающую от удаленной ветки по умолчанию, принесет с собой отсутствующие коммиты. При отсутствии настроенного remote новый worktree стартует от локального `HEAD`.

## Go further

- [Изоляция субагентов через worktrees](https://code.claude.com/docs/en/worktrees#isolate-subagents-with-worktrees): параметр `isolation: worktree` во frontmatter субагента выделяет под каждый запуск собственную рабочую копию.
- [Обмен сообщениями между сессиями Claude Code](https://code.claude.com/docs/en/cross-session-messaging): возможность сессий в разных worktree передавать найденные данные друг другу.
- [Запуск нескольких сессий Claude](https://code.claude.com/docs/en/best-practices#run-multiple-claude-sessions): паттерн Автор / Ревьюер (Writer/Reviewer), где свежая сессия проверяет код, написанный другой сессией.

---

<sub>Sources: [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Settings files and precedence](https://code.claude.com/docs/en/settings) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [Run agents in parallel](https://code.claude.com/docs/en/agents) · [git-worktree](https://git-scm.com/docs/git-worktree) · [git-merge](https://git-scm.com/docs/git-merge) · [git-check-ignore](https://git-scm.com/docs/git-check-ignore) · [gitignore](https://git-scm.com/docs/gitignore) · [git-merge-base](https://git-scm.com/docs/git-merge-base) · [git-branch](https://git-scm.com/docs/git-branch)</sub>

<sub>← [Advanced index](README.md) · [Выбор паттерна оркестрации](02-orchestration-patterns.md) → · Topic: [Субагенты и параллельная работа](../topics/subagents-and-parallel-work.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-1)</sub>
