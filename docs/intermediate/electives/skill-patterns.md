# Skill patterns: live data, supporting files and a context of its own

<sub>**Intermediate** · Elective · about 20 minutes · Needs: [Turn a repeated workflow into a skill](../01-first-skill.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого факультатива вы сможете создавать навыки, подставляющие динамические данные в промпт, выносящие обширные справочные материалы в отдельные файлы и запускаемые в собственном изолированном субагенте. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can write a skill that pulls live data into its prompt, keeps long reference material in a separate file, and runs in its own subagent.</details>

## You need

- Репозиторий с незакоммиченными изменениями для проведения ревью и навыки из [урока 1](../01-first-skill.md).
- Расход токенов: минимальный (light).

## The idea

В уроке 1 навыки состояли из инструкций и переменной `$ARGUMENTS`. Ряд дополнительных паттернов позволяет навыкам решать более сложные задачи при меньшем расходе контекста ([Extend Claude with skills](https://code.claude.com/docs/en/skills)): <details style="display:inline"><summary><code>[en]</code></summary>Lesson 1's skills were instructions and $ARGUMENTS. A few more patterns make a skill do more with less context</details>

| Паттерн | Как записывается | Что делает |
|---|---|---|
| Live data (актуальные данные) | `` !`git diff HEAD` `` в теле навыка | Claude Code выполняет команду до чтения навыка моделью и подставляет её вывод на место вызова |
| Supporting files (вспомогательные файлы) | Файлы рядом с `SKILL.md`, ссылки на них | Объёмные справочные данные, считываемые Claude только при реальной необходимости |
| A context of its own (собственный контекст) | `context: fork` во frontmatter | Навык становится промптом субагента, запускаемого в фоне без доступа к текущему диалогу; результат возвращается по завершении |
| Named arguments (именованные аргументы) | `arguments: [focus]`, затем `$focus` | Каждое слово после имени навыка подставляется в собственный плейсхолдер |
| Tool grants (предварительное одобрение) | `allowed-tools: Read, Grep` | Claude может вызывать перечисленные инструменты без подтверждений во время выполнения хода навыка |

Держите `SKILL.md` компактным: после вызова навыка весь его текст остаётся в истории диалога. Документация рекомендует укладываться в 500 строк, вынося подробности во вспомогательные файлы.

## Worked example

1. Сохраните навык как `.claude/skills/changes-review/SKILL.md`:

   ```markdown
   ---
   name: changes-review
   description: Reviews the uncommitted changes in this repository against the team checklist. Use when the user asks for a review before committing.
   context: fork
   allowed-tools: Bash(git diff *)
   ---

   ## Changes

   !`git diff HEAD`

   ## Your task

   Review the changes above against [checklist.md](checklist.md). Report each problem with its file and line, then one line on whether the change is ready to commit.
   ```

2. Сохраните чек-лист рядом: `.claude/skills/changes-review/checklist.md`:

   ```markdown
   # Team change checklist

   - No console.log or temporary debug code.
   - Tests exist for new code.
   - Comments explain why, not what.
   ```

3. Запустите `claude` и вызовите навык: `/changes-review`. Он выполнится в изолированном контексте субагента, проверит вывод `git diff HEAD` по чек-листу и вернёт компактный отчёт.

## Your turn

Создайте собственный навык с именованным аргументом и динамическими данными.

1. Создайте `.claude/skills/explain-file/SKILL.md`, принимающий путь к файлу в аргументе `$target`:

   ```markdown
   ---
   name: explain-file
   description: Explains a file's purpose and architecture. Use when asked to explain a file.
   argument-hint: "[file-path]"
   arguments: [target]
   ---

   Explain the architecture and responsibilities of this file: $target.
   ```

2. Вызовите его: `/explain-file src/config.js`.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте пункты самостоятельно.

- [ ] Навык `changes-review` запускается и считывает diff без ручного копирования.
- [ ] Вспомогательный файл `checklist.md` прочитан только при необходимости.
- [ ] Self-check (not tested): вы понимаете разницу между выполнением в основном диалоге и в режиме `context: fork`.

## Go further

- [Extend Claude with skills](https://code.claude.com/docs/en/skills): продвинутые паттерны организации навыков и работа со сложными аргументами.

---

<sub>Sources: [Extend Claude with skills](https://code.claude.com/docs/en/skills) · [Tools reference](https://code.claude.com/docs/en/tools-reference)</sub>

<sub>Back to the [Intermediate index](../README.md) · Topic: [Skills](../../topics/skills.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-skills)</sub>
