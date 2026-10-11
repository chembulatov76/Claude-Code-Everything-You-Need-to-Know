<a id="advanced"></a>
# Продвинутый уровень (Advanced)

<sub>**Advanced** · about 3 hours 20 minutes, plus run time · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

За рамками одной сессии: параллельные сессии, оркестрация, автономные запуски в скриптах и CI/CD, совместная настройка окружения для команды и жесткие ограничения безопасности для автономной работы. <details style="display:inline"><summary><code>[en]</code></summary>Beyond one session: parallel sessions, orchestration, unattended runs in scripts and CI, a setup your team shares, and limits that keep an autonomous run safe.</details>

<a id="youre-ready-for-this-level-if-you-can"></a>
## You're ready for this level if you can

- [ ] Создавать навык проекта, запускаемый по имени и автоматически подгружающийся при необходимости.
- [ ] Писать хук для строгого соблюдения правила и подтверждать его срабатывание.
- [ ] Делегировать задачи специализированному субагенту с ограниченным набором инструментов.
- [ ] Добавлять MCP-сервер на уровне проекта и устанавливать плагины из каталогов Anthropic.
- [ ] Размещать правила разрешений в нужной области видимости, включать песочницу и подбирать модель и уровень усилий под задачу.
- [ ] Структурировать память проекта и выяснять, какие инструкции загрузились при сбоях в их выполнении.

<a id="practice"></a>
## Practice

Практикуйтесь в собственном репозитории и запускайте каждую проверку из копии [учебного шаблона](https://github.com/wesammustafa/claude-code-practice) командой `npm run check -- <lesson-id> --dir <your repo>`. Исключениями являются урок по GitHub Actions и выпускное задание: они выполняются в отдельной копии шаблона, которую вы затем удаляете, чтобы не оставлять активных воркфлоу или секретов.

Копия, созданная для более раннего уровня, может не содержать проверок уровня Advanced (в выводе `npm run check` отсутствуют уроки с префиксом `a-`). Подтяните их из шаблона в свою копию:

```bash
git fetch https://github.com/wesammustafa/claude-code-practice main
git checkout FETCH_HEAD -- checks capstone
git commit -m "Add the Advanced checks"
```

<a id="lessons"></a>
## Lessons

1. [Параллельные сессии с worktrees](01-parallel-sessions.md) <details style="display:inline"><summary><code>[en]</code></summary>Parallel sessions with worktrees</details>
2. [Выбор паттерна оркестрации](02-orchestration-patterns.md) <details style="display:inline"><summary><code>[en]</code></summary>Choose an orchestration pattern</details>
3. [Запуск и сохранение динамического воркфлоу](03-dynamic-workflows.md) <details style="display:inline"><summary><code>[en]</code></summary>Run and save a dynamic workflow</details>
4. [Скриптинг Claude Code с `claude -p`](04-headless.md) <details style="display:inline"><summary><code>[en]</code></summary>Script Claude Code with `claude -p`</details>
5. [GitHub Actions](05-github-actions.md) <details style="display:inline"><summary><code>[en]</code></summary>GitHub Actions</details>
6. [Совместное использование настроек в команде](06-share-your-setup.md) <details style="display:inline"><summary><code>[en]</code></summary>Share your setup with a team</details>
7. [Установка ограничений для автономных запусков](07-bounded-runs.md) <details style="display:inline"><summary><code>[en]</code></summary>Put bounds on autonomous runs</details>

Каждый урок занимает от 10 до 20 минут, плюс время работы самого агента.

<a id="capstone"></a>
## Capstone

[Добавление CI-ревью для совместной работы в команде](capstone.md): в новой копии шаблона запустите две параллельные сессии в worktrees, создайте скрипт ревью `claude -p` с ограничениями, командный каталог плагинов, сохраненный динамический воркфлоу и воркфлоу ревью pull request с лимитами. Затем удалите тестовый воркфлоу, секреты и созданные ключи API.

<a id="by-the-end-of-this-level-you-can"></a>
## By the end of this level you can

- [ ] Запускать параллельные сессии в git worktrees и объединять результаты их работы.
- [ ] Выбирать между субагентами, воркфлоу или командой агентов в зависимости от задачи и расходов, а также запускать динамические воркфлоу.
- [ ] Писать скрипты автоматизации с `claude -p` и воркфлоу для GitHub Actions.
- [ ] Передавать конфигурацию через версионируемые файлы и плагины командных каталогов.
- [ ] Ограничивать автономные запуски правилами наименьших привилегий, песочницей, лимитами раундов или расходов, шагом подтверждения и предварительной оценкой стоимости.

<a id="electives"></a>
## Electives

Дополнительные практические темы (факультативы):

- [Команды агентов (Agent teams)](electives/agent-teams.md): ведущий агент и участники команды, обменивающиеся сообщениями, сформированные из определений ваших субагентов.
- [Просмотр агентов (Agent view)](electives/agent-view.md): запуск фоновых задач из одного экрана и вмешательство только тогда, когда агенту требуется ответ.
- [`/goal`, `/loop` и рутины](electives/goal-loop-routines.md): работа Claude Code до достижения заданного условия, с регулярным интервалом или по расписанию в облаке.
- [Тестирование и публикация плагина](electives/publish-a-plugin.md): проверка структуры и работы плагина с последующей публикацией в собственном каталоге.
- [Dev containers](electives/dev-containers.md): запуск Claude Code в изолированном контейнере с сетевым экраном, сохранением сессии и возможностью автономной работы.

<a id="official-companions"></a>
## Official companions

- [Run agents in parallel](https://code.claude.com/docs/en/agents): сравнение способов одновременного выполнения нескольких задач в Claude Code от Anthropic.
- [The AI-native SDLC playbook](https://academy.claude.com/courses/ai-native-sdlc-playbook): курс Claude Academy по перестройке жизненного цикла разработки команды вокруг AI.

<a id="beyond-this-guide"></a>
## Beyond this guide

[За рамками этого руководства](beyond-this-guide.md): создание собственных агентов и MCP-серверов, а также развертывание и внедрение Claude Code в организации.

---

<sub>Sources: [Run agents in parallel](https://code.claude.com/docs/en/agents) · [The AI-native SDLC playbook](https://academy.claude.com/courses/ai-native-sdlc-playbook)</sub>

<sub>Up: [Claude Code: Everything You Need to Know](../../README.md#pick-your-level)</sub>
