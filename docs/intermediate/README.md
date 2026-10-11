# Intermediate

<sub>**Intermediate** · about 3 hours · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Рабочее окружение инженера. Вы настраиваете Claude Code под специфику своего проекта: навыки (skills), память проекта, выбор модели и уровня усилий, права доступа и песочницу (sandbox), хуки (hooks), субагентов, MCP-серверы и плагины. <details style="display:inline"><summary><code>[en]</code></summary>One developer's setup. You shape Claude Code to your own project: skills, project memory, model and effort, permissions and the sandbox, hooks, subagents, MCP servers and plugins.</details>

## You're ready for this level if you can

- [ ] Установить Claude Code, войти в аккаунт, запустить инструмент в репозитории и определить используемые версию, модель и аккаунт.
- [ ] Выбирать и переключать режимы разрешений, а также утверждать план в plan mode перед внесением изменений.
- [ ] Проводить задачу от формулировки до коммита: исследование, планирование, реализация, тесты, проверка diff, корректировка, коммит.
- [ ] Управлять контекстом сессии с помощью команд context, clear, compact, rewind и resume.
- [ ] Создавать файл `CLAUDE.md` и подтверждать, что новая сессия соблюдает описанные правила.

## Practice

Практикуйтесь в собственном репозитории: адаптация Claude Code под реальный проект - главная цель этого уровня. Запускайте каждую проверку из учебного шаблона [practice template](https://github.com/wesammustafa/claude-code-practice) командой `npm run check -- <lesson-id> --dir <your repo>`. Если вы предпочитаете не менять рабочий репозиторий, используйте саму копию шаблона; итоговое задание capstone запускается непосредственно в ней.

Копия, созданная для уровня Beginner, может не содержать проверок уровня Intermediate (в выводе `npm run check` отсутствуют уроки с префиксом `i-`). Подтяните их из шаблона в свою копию:

```bash
git fetch https://github.com/wesammustafa/claude-code-practice main
git checkout FETCH_HEAD -- checks capstone
git commit -m "Add the Intermediate checks"
```

## When to use what

Пять основных способов расширения возможностей Claude Code; полный список приведён в обзоре [Extend Claude Code](https://code.claude.com/docs/en/features-overview). Каждый пункт ссылается на соответствующий урок: <details style="display:inline"><summary><code>[en]</code></summary>Five of the ways to extend Claude Code; Extend Claude Code lists the rest. Each links its lesson.</details>

- **Навыки (Skills):** вы раз за разом повторяете одни и те же инструкции или шаги рабочего процесса. Навык проекта хранится в файле `.claude/skills/<name>/SKILL.md` ([Extend Claude with skills](https://code.claude.com/docs/en/skills); урок: [Превращение повторяющегося процесса в навык](01-first-skill.md)).
- **Хуки (Hooks):** действие должно выполняться строго в определённый момент сессии (например, перед вызовом инструмента) независимо от решений модели. Хуки описываются в файле конфигурации, например `.claude/settings.json` ([Hooks](https://code.claude.com/docs/en/hooks); урок: [Обеспечение правил с помощью хуков](05-hooks.md)).
- **Субагенты (Subagents):** вспомогательная задача должна выполниться в собственном изолированном контекстном окне и вернуть только резюме. Субагент проекта описывается в `.claude/agents/` ([Create custom subagents](https://code.claude.com/docs/en/sub-agents); урок: [Делегирование кастомному субагенту](06-subagents.md)).
- **Динамические воркфлоу (Dynamic workflows):** задача требует координации большего числа агентов, чем возможно внутри одного диалога. Сохранённый воркфлоу хранится в `.claude/workflows/` ([Workflows](https://code.claude.com/docs/en/workflows); урок: [Запуск и сохранение динамического воркфлоу](../advanced/03-dynamic-workflows.md)).
- **MCP-серверы:** Claude необходим доступ к инструментам или данным вне локальных файлов (браузер, база данных или внешний API). Сервер проектного уровня описывается в файле `.mcp.json` в корне репозитория ([Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp); урок: [Подключение инструмента через MCP](07-mcp.md)).

Эти механизмы объединяются: [плагин (plugin)](https://code.claude.com/docs/en/plugins) упаковывает навыки, агентов, хуки и MCP-серверы, позволяя устанавливать их единым набором ([Установка плагинов и управление ими](08-plugins.md)).

## Lessons

1. [Превращение повторяющегося процесса в навык](01-first-skill.md)
2. [Организация памяти проекта и проверка загруженного](02-organize-memory.md)
3. [Выбор модели и уровня усилий мысли](03-model-and-effort.md)
4. [Разрешения, области видимости настроек и песочница](04-permissions-and-sandbox.md)
5. [Обеспечение правил с помощью хуков](05-hooks.md)
6. [Делегирование кастомному субагенту](06-subagents.md)
7. [Подключение инструмента через MCP](07-mcp.md)
8. [Установка плагинов и управление ими](08-plugins.md)

Каждый урок занимает от 10 до 20 минут.

## Capstone

[Подготовка репозитория для нового коллеги](capstone.md): подготовка репозитория для нового коллеги - навык, защищающий папку хук, субагент-ревьюер только для чтения, проектный MCP-сервер и официальный плагин с настройками проекта, изолирующими секреты и включающими песочницу.

## By the end of this level you can

- [ ] Создавать навык проекта, запускаемый по имени и автоматически загружающийся при подходящем запросе.
- [ ] Писать хук для строгого соблюдения правила и проверять его срабатывание.
- [ ] Делегировать задачи специализированному субагенту с ограниченным набором инструментов.
- [ ] Подключать MCP-сервер на уровне проекта и устанавливать плагины из каталогов Anthropic.
- [ ] Размещать правила доступа в корректных областях видимости, активировать песочницу и подбирать модель и уровень усилий под задачу.
- [ ] Грамотно структурировать память проекта и определять, какие файлы инструкций были загружены при возникновении расхождений.

## Electives

Дополнительные уроки:

- [Паттерны навыков](electives/skill-patterns.md): динамические данные, вспомогательные файлы и собственный контекст навыка.
- [Claude в Chrome](electives/claude-in-chrome.md): проверка изменений пользовательского интерфейса в реальном браузере.
- [Настройка строки статуса](electives/status-line.md): вывод модели, текущей папки и заполненности контекста в собственной строке состояния.
- [Быстрый режим (Fast mode)](electives/fast-mode.md): ускоренные ответы Opus с оплатой через usage credits сверх тарифного плана.

## Official companions

- [Claude Code in action](https://academy.claude.com/courses/claude-code-in-action): курс Claude Academy по проведению длительных автономных сессий, результатам которых можно доверять.
- [Introduction to agent skills](https://academy.claude.com/courses/introduction-to-agent-skills) и [Introduction to subagents](https://academy.claude.com/courses/introduction-to-subagents): курсы Claude Academy по созданию навыков и субагентов.

## Next level

[Advanced](../advanced/README.md): за пределами одной терминальной сессии.

---

<sub>Sources: [Extend Claude Code](https://code.claude.com/docs/en/features-overview) · [Extend Claude with skills](https://code.claude.com/docs/en/skills) · [Hooks](https://code.claude.com/docs/en/hooks) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [Workflows](https://code.claude.com/docs/en/workflows) · [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp) · [Plugins](https://code.claude.com/docs/en/plugins) · [Claude Code in action](https://academy.claude.com/courses/claude-code-in-action) · [Introduction to agent skills](https://academy.claude.com/courses/introduction-to-agent-skills) · [Introduction to subagents](https://academy.claude.com/courses/introduction-to-subagents)</sub>

<sub>Up: [Claude Code: Everything You Need to Know](../../README.md#pick-your-level)</sub>
