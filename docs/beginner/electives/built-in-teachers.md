# Learn from Claude Code itself

<sub>**Beginner** · Elective · about 20 minutes · Needs: [Install, sign in and look around](../01-install-and-look-around.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого факультатива вы сможете использовать встроенные обучающие инструменты Claude Code: команду `/powerup`, стили вывода Learning и Explanatory, а также встроенного субагента для ответов на вопросы о возможностях инструмента. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can use Claude Code's built-in teaching aids: /powerup, the Learning and Explanatory output styles, and its guide to its own features.</details>

## You need

- Ваша учебная копия репозитория без незакоммиченных изменений.
- Расход токенов: минимальный (light).

## The idea

Claude Code может обучать вас прямо в процессе работы. Для этого предусмотрены три встроенных инструмента: <details style="display:inline"><summary><code>[en]</code></summary>Claude Code can teach you while you work. Three aids are built in</details>

- **`/powerup`** запускает быстрые интерактивные уроки по возможностям Claude Code с анимированными демонстрациями ([Commands](https://code.claude.com/docs/en/commands)).
- **Стили вывода (Output styles)** изменяют формат и детальность ответов Claude ([Output styles](https://code.claude.com/docs/en/output-styles)). В стиле **Explanatory** Claude добавляет краткие блоки `Insight`, объясняющие архитектурные решения в коде. В стиле **Learning** к этим блокам добавляется режим активного обучения: агент оставляет ключевые решения для самостоятельной реализации пользователем.
- **Справочный субагент по Claude Code**: `claude-code-guide` - встроенный помощник для консультаций по возможностям самого Claude Code ([Create custom subagents](https://code.claude.com/docs/en/sub-agents)). Укажите его имя в запросе, иначе Claude может попытаться ответить из общих базовых знаний.

Зачем писать часть кода самостоятельно? В исследовании Anthropic разработчики, изучавшие новую библиотеку исключительно с помощью ИИ-ассистента, набрали в среднем более низкие баллы в проверочном тесте по сравнению с теми, кто писал код вручную. Те разработчики, кто показал высокие результаты, использовали ассистента для вопросов и подробных разъяснений, а не просто для генерации готового кода ([How AI assistance impacts the formation of coding skills](https://www.anthropic.com/research/AI-assistance-coding-skills)). Режим Learning как раз вырабатывает эту полезную привычку.

## Worked example

1. Запустите `claude` в вашей учебной копии и введите команду `/powerup`. Выберите урок, опробуйте показанные действия и следуйте подсказкам на экране. Урок считается завершённым, когда вы нажимаете `Enter` на его карточке (`Enter to mark done`). Если в уроке `/powerup` упоминается режим `default`, речь идёт о режиме Manual ([Choose a permission mode](https://code.claude.com/docs/en/permission-modes)), рассмотренном в [уроке 2](../02-permission-modes-and-plan-mode.md).
2. Переключите стиль вывода:

   ```text
   /output-style learning
   ```

   Claude Code сохраняет выбор в файле `.claude/settings.local.json` этого проекта, поэтому каждая последующая сессия здесь сохранит этот стиль, пока вы не выполните `/output-style default`. Раздел Your turn завершается этим шагом; если вы закончите работу раньше, выполните возврат к стилю по умолчанию вручную.
3. Попросите выполнить небольшое изменение, оставив выбор решения за вами:

   ```text
   Add a function to src/links.js that counts the external links on a page, with a test. Leave the decision about repeated links to me.
   ```

   Claude объяснит свои решения в блоках `Insight` (хотя они могут появляться не в каждом ответе). Дойдя до запланированной развилки, агент добавит комментарий `TODO(human)` в файл и остановится с запросом `Learn by Doing`, в котором опишет, что именно нужно написать вам ([Output styles](https://code.claude.com/docs/en/output-styles#learning)). Откройте указанный файл в редакторе, допишите код на месте комментария `TODO(human)`, сохраните файл и сообщите Claude о завершении. Claude прокомментирует ваш код и завершит задачу; если тесты не запустились автоматически, попросите их запустить. Стиль является инструкцией, а не абсолютной гарантией: если Claude случайно напишет весь код сам, попросите его явно передать решение о повторяющихся ссылках вам.
4. Когда тесты пройдут успешно, отложите изменения в сторону для чистоты следующих уроков. Откройте второй терминал в той же папке и выполните:

   ```bash
   git stash push --include-untracked -m b-teachers
   ```

5. Вернувшись в сессию Claude Code, задайте вопрос о самом Claude Code, явно назвав субагента, чтобы Claude перенаправил вопрос ему ([Invoke subagents explicitly](https://code.claude.com/docs/en/sub-agents#invoke-subagents-explicitly)):

   ```text
   Use the claude-code-guide agent: how do I make my own slash command in Claude Code?
   ```

   Когда Claude перенаправит запрос, в логе диалога появится строка `claude-code-guide(...)` с кратким описанием задачи в скобках. Если такой строки нет, Claude ответил без субагента: отправьте промпт повторно. Формулировки ответа могут отличаться, но он должен направить вас к механизму навыков (skills) и файлу `SKILL.md` в папке `.claude/skills/` ([Extend Claude with skills](https://code.claude.com/docs/en/skills)). Ответ может также упомянуть устаревшую папку `.claude/commands/`, которая всё ещё поддерживается.

## Your turn

1. Запустите `/powerup` и пройдите ещё один урок.
2. Переключитесь на объясняющий стиль командой `/output-style explanatory`, попросите Claude объяснить устройство файла `src/check-links.js` и изучите блоки `Insight`. Затем во втором терминале сохраните настройки для проверки:

   ```bash
   cp .claude/settings.local.json .practice/b-teachers-style.json
   ```

3. Задайте `claude-code-guide` вопрос о функции Claude Code, которую вы ещё не пробовали: начните промпт с фразы `Use the claude-code-guide agent:` и найдите соответствующую строку вызова в логе. Если вы уже задавали вопрос этому субагенту ранее в этой сессии, Claude может направить вопрос тому же помощнику: в этом случае в логе появится `Resuming agent` вместо новой строки `claude-code-guide(...)`, что также считается успешным вызовом.
4. По завершении вернитесь к обычному режиму: `/output-style default`.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте каждый пункт самостоятельно. Задание выполнено, когда выполняются все условия:

- [ ] Файл `.practice/b-teachers-style.json`, сохранённый на шаге 2 раздела Your turn, содержит запись `outputStyle` со значением Explanatory. Проверьте командой `cat .practice/b-teachers-style.json`.
- [ ] Self-check (not tested): вы завершили два урока в `/powerup`.

Если файл отсутствует или указывает другой стиль, выполните `/output-style explanatory` в Claude Code, повторите команду `cp` во втором терминале, после чего выполните `/output-style default`.

## Go further

- [Output styles](https://code.claude.com/docs/en/output-styles): полный перечень встроенных стилей вывода и руководство по созданию собственных.

---

<sub>Sources: [Commands](https://code.claude.com/docs/en/commands) · [Output styles](https://code.claude.com/docs/en/output-styles) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Extend Claude with skills](https://code.claude.com/docs/en/skills) · [How AI assistance impacts the formation of coding skills](https://www.anthropic.com/research/AI-assistance-coding-skills)</sub>

<sub>Back to the [Beginner index](../README.md) · Topic: [Memory and context](../../topics/memory-and-context.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-teachers)</sub>
