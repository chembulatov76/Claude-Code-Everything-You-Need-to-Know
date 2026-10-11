# Keep a session on track

<sub>**Beginner** · Lesson 4 of 5 · about 20 minutes · Needs: [Your first change, from request to commit](03-first-change.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого урока вы научитесь удерживать сессию в рамках задачи: отслеживать заполненность контекстного окна, очищать или сжимать контекст (clear/compact), откатывать нежелательные изменения (rewind) и возобновлять сохранённые диалоги (resume). <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can keep a session on track: see how full its context is, clear or compact it, rewind a change, and resume an earlier conversation.</details>

## You need

- Ваша копия учебного шаблона без незакоммиченных изменений и Node.js LTS (в среде Codespace доступно сразу). Чтобы практиковаться в собственном репозитории, сохраните также учебный шаблон (проверочный скрипт вызывается из него). Создайте папку `.practice/` по инструкции [Урока 1](01-install-and-look-around.md#you-need), если не сделали этого ранее. Начните урок с чистым рабочим деревом git: закоммитьте изменения или отложите их командой `git stash push --include-untracked`.
- Расход токенов: минимальный (light).

## The idea

Всё содержимое диалога: ваши промпты, ответы Claude и каждый прочитанный файл: накапливается в контекстном окне сессии. Окно контекста велико, но не бесконечно, и затянувшийся диалог оставляет Claude всё меньше пространства для текущей актуальной работы ([Explore the context window](https://code.claude.com/docs/en/context-window)). Пять встроенных команд помогают держать контекст под контролем ([Commands](https://code.claude.com/docs/en/commands)):

| Команда | Назначение |
|---|---|
| `/context` | Показывает степень заполнения контекстного окна в виде цветной сетки |
| `/compact` | Сжимает и суммаризирует историю диалога для освобождения места |
| `/clear` | Начинает новый диалог с чистым контекстом |
| `/rewind` | Возвращает состояние к более раннему промпту, восстанавливая код, диалог или оба элемента |
| `/resume` | Возобновляет одну из сохранённых ранее сессий |

Используйте `/compact`, когда хотите продолжить работу над текущей задачей. Применяйте `/clear`, когда переходите к принципиально новой задаче. Вызывайте `/rewind`, если изменения пошли по ошибочному сценарию ([Checkpointing](https://code.claude.com/docs/en/checkpointing)).

## Worked example

1. Запустите `claude` в учебной копии репозитория и введите `/context`. Сетка наглядно продемонстрирует объём занятого контекста с детализацией по категориям (включая ваши сообщения и файлы памяти, такие как `CLAUDE.md`, если они были загружены) ([Explore the context window](https://code.claude.com/docs/en/context-window)). В учебной копии файл `CLAUDE.md` пока отсутствует, поэтому секция памяти может быть пустой; [Урок 5](05-project-memory.md) добавит его.
2. Немного наполните контекст вопросом:

   ```text
   Explain how src/check-links.js decides that a link is broken.
   ```

   Снова вызовите `/context`: доля категории сообщений заметно увеличилась.
3. Сожмите контекст, указав Claude, какую информацию важно сохранить:

   ```text
   /compact keep what we learned about check-links.js
   ```

   История сообщений будет заменена кратким саммари: в окне `/context` блок сообщений сократится, а свободное место увеличится. После одного короткого вопроса эффект сжатия невелик, но компактность даёт решающее преимущество в длинных диалогах.
4. Внесите изменение для последующего отката. Механизм Rewind не умеет отменять изменения, сделанные терминальными командами Claude (см. раздел [Watch out](#watch-out)), поэтому в промпте мы прямо требуем использовать инструмент редактирования файлов. Подтвердите операцию, если Claude запросит разрешение:

   ```text
   In src/cli.js, change the success message to "All relative links are fine." Use your file editing tool, not a shell command.
   ```

   Во втором терминале в той же папке сохраните снимок состояния git:

   ```bash
   git status --porcelain > .practice/b-4-before-rewind.txt
   ```

   В файле будет зафиксирован изменённый путь `src/cli.js`.
5. Выполните откат. Введите команду `/rewind` или дважды нажмите клавишу `Esc` при пустой строке ввода. Выберите промпт из шага 4 и укажите вариант **Restore code and conversation**. Если в меню доступны только **Restore conversation**, пункты сжатия и **Never mind**, значит, Claude изменил файл командой терминала (что rewind отменить не в силах): выберите **Never mind**, закройте меню клавишей `Esc` ([Interactive mode](https://code.claude.com/docs/en/interactive-mode)), выполните во втором терминале `git restore src/cli.js` и повторите шаг 4.

   Когда исходный код восстановлен, сохраните новое состояние git во втором терминале:

   ```bash
   git status --porcelain > .practice/b-4-after-rewind.txt
   ```

   Этот файл должен быть пустым: изменение отменено.
6. Откат вернул текст промпта из шага 4 обратно в поле ввода. Дважды быстро нажмите клавишу `Esc`, чтобы очистить поле (после первого нажатия внизу появится подсказка `Esc again to clear`). Затем введите `/clear`. Диалог начнётся заново с пустым контекстом, при этом предыдущая беседа останется сохранена.
7. Введите `/exit`, затем заново откройте последнюю сессию в этой директории ([Manage sessions](https://code.claude.com/docs/en/sessions)):

   ```bash
   claude --continue
   ```

   Если откроется пустая сессия из шага 6, введите `/resume` и выберите более раннюю беседу. Внутри сессии вызов `/resume` без аргументов открывает интерактивный список сохранённых разговоров. В выбранном диалоге может по-прежнему отображаться текст отменённой правки (так как он был сохранён до отката), однако ваши файлы на диске останутся в исходном виде (`git status --porcelain` ничего не выводит), поэтому не просите Claude вносить эту правку снова.

## Your turn

Повторите этот сценарий в учебной копии или собственном репозитории:

1. Вызовите `/context`, задайте один вопрос о коде и повторно просмотрите `/context`.
2. Запустите `/compact` с указанием того, что важно сохранить в памяти.
3. Попросите внести небольшую правку в один файл, закончив промпт фразой "Use your file editing tool, not a shell command.", и подтвердите правку. Сохраните статус: `git status --porcelain > .practice/b-4-before-rewind.txt`.
4. Откатите этот промпт через **Restore code and conversation** и сохраните статус: `git status --porcelain > .practice/b-4-after-rewind.txt`.
5. Дважды нажмите `Esc` для очистки строки ввода, затем выполните последовательно `/clear`, затем `/exit`, затем `claude --continue`.

## Check

Задание считается выполненным, когда соблюдены все условия:

- [ ] Файл `.practice/b-4-before-rewind.txt` содержит путь к файлу, изменённому Claude: `cat .practice/b-4-before-rewind.txt`.
- [ ] Файл `.practice/b-4-after-rewind.txt` пуст: команда `cat .practice/b-4-after-rewind.txt` ничего не выводит.
- [ ] Self-check (not tested): вы можете объяснить разницу в применении команд `/compact` и `/clear`.

Во втором терминале запустите `npm run check -- b-4` в учебной копии. В собственном репозитории выполните: `npm run check -- b-4 --dir <your repo>`.

Если первый файл пуст, значит, снимок был сохранён до выполнения правки: повторите изменение, сохраните снимок и выполните откат заново. Если во втором файле всё ещё числится изменённый файл, откатите промпт через пункт меню, восстанавливающий код, и пересохраните второй файл. Если промпт пропал из списка или не предлагает возврат кода, выполните `git restore <file>` и повторите шаги 3 и 4 блока Your turn. Файлы, изменённые до начала урока, также останутся в статусе, поскольку rewind их не затрагивает: закоммитьте их или отложите в `git stash push --include-untracked`, после чего пересохраните второй файл.

## Watch out

- Механизм Rewind отменяет только те правки, которые Claude внёс с помощью встроенных инструментов редактирования файлов. Изменения, выполненные терминальными командами (например, удаление файлов через `rm`), не отслеживаются контрольными точками ([Checkpointing](https://code.claude.com/docs/en/checkpointing#bash-command-changes-not-tracked)): всегда фиксируйте важный код коммитами, чтобы git мог вернуть его при необходимости.

## Go further

- [Explore the context window](https://code.claude.com/docs/en/context-window): интерактивный разбор структуры контекста сессии.
- [Session management and 1M context](https://claude.com/blog/using-claude-code-session-management-and-1m-context): рекомендации Anthropic по использованию compact, clear и открытию новых сессий.

---

<sub>Sources: [Commands](https://code.claude.com/docs/en/commands) · [Explore the context window](https://code.claude.com/docs/en/context-window) · [Checkpointing](https://code.claude.com/docs/en/checkpointing) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Manage sessions](https://code.claude.com/docs/en/sessions)</sub>

<sub>← [Your first change, from request to commit](03-first-change.md) · [Beginner index](README.md) · [Project memory with CLAUDE.md](05-project-memory.md) → · Topic: [Memory and context](../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-4)</sub>
