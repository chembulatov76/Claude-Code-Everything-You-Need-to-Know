<a id="beyond-this-guide"></a>
# За рамками этого руководства (Beyond this guide)

<sub>**Advanced** · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

Куда двигаться дальше после завершения уровня Advanced. Это руководство обучает работе одного разработчика с Claude Code - индивидуально или в команде. Создание собственных агентов и MCP-серверов, развертывание Claude Code в масштабе всей организации и внедрение инструмента в процессы команды представляют собой отдельные масштабные задачи с собственной официальной документацией. На этой странице собраны ссылки на ключевые материалы. <details style="display:inline"><summary><code>[en]</code></summary>Where to go once you finish Advanced. The guide teaches one developer's use of Claude Code, alone or with a team. Building your own agents and MCP servers, running Claude Code for a whole organization, and getting a team to adopt it are separate jobs with their own official docs, so this page links them and doesn't teach them.</details>

Все материалы опубликованы Anthropic, за исключением страниц проекта MCP на modelcontextprotocol.io.

Ресурсы проверены по критериям включения 2026-10-09.

<a id="build-your-own-agents"></a>
## Создание собственных агентов

Для интеграции цикла агентов Claude Code в собственное приложение:

- [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview): инструменты, цикл агента и управление контекстом, лежащие в основе Claude Code, в виде библиотеки для Python или TypeScript. В первой таблице приведено сопоставление с другими инструментами Claude.
- [Agent SDK quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart): создание агента, находящего и исправляющего ошибки в существующем коде.
- [Intro to Claude](https://platform.claude.com/docs/en/intro): документация Claude API с пошаговым путем от первого вызова до рабочей интеграции.

Для управления аналогичным циклом агента из языков, отличных от Python или TypeScript, документация рекомендует запускать CLI как дочерний процесс с флагами `-p` и `--output-format json` ([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview#compare-the-agent-sdk-to-other-claude-tools)), как показано в [Уроке 4](04-headless.md).

Продукт или сервис, созданный на базе Agent SDK, должен использовать аутентификацию по API-ключу через Claude Console или поддерживаемого облачного провайдера ([Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance#authentication-and-credential-use)). Без предварительного согласования Anthropic не разрешает сторонним разработчикам использовать авторизацию claude.ai в своих продуктах ([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview#get-started)).

Когда вы пишете код, вызывающий Claude API, встроенный навык `/claude-api` подгружает справочные материалы для языка вашего проекта; Claude также загружает его автоматически при импорте библиотек `anthropic` или `@anthropic-ai/sdk` ([Skills](https://code.claude.com/docs/en/skills#work-on-claude-api-projects)).

<a id="build-mcp-servers"></a>
## Создание MCP-серверов

В [Уроке 7 уровня Intermediate](../intermediate/07-mcp.md) использовался готовый сервер. Для создания собственного сервера:

- [Find and build MCP servers](https://code.claude.com/docs/en/mcp#find-and-build-mcp-servers): отправная точка для разработчиков серверов в документации Claude Code, описывающая основы протокола, аутентификацию коннекторов, тестирование и публикацию в каталоге Anthropic Directory.
- [Build an MCP server](https://modelcontextprotocol.io/docs/develop/build-server) (Разработчик: проект MCP): официальное руководство проекта MCP и доступные [SDK](https://modelcontextprotocol.io/docs/sdk) для создания серверов и клиентов.
- [Introduction to Model Context Protocol](https://academy.claude.com/courses/introduction-to-model-context-protocol): курс Claude Academy по созданию серверов и клиентов с нуля на Python SDK.
- [Model Context Protocol: Advanced topics](https://academy.claude.com/courses/model-context-protocol-advanced-topics): курс Claude Academy по сэмплированию, уведомлениям и roots.

<a id="run-claude-code-for-an-organization"></a>
## Развертывание Claude Code в организации

Урок [Совместное использование настроек в команде](06-share-your-setup.md) сохраняет конфигурацию в репозитории проекта. На уровне организации также настраиваются общие политики, аутентификация, сетевой доступ и отчетность для всех рабочих машин:

- [Set up Claude Code for your organization](https://code.claude.com/docs/en/admin-setup): карта решений для администраторов от Anthropic и отправная точка внедрения. Охватывает выбор API-провайдеров, управляемые настройки, применение политик, мониторинг использования, обработку данных, а также настройку SSO и распределение мест.
- [Deploy managed settings](https://code.claude.com/docs/en/managed-settings): доставка корпоративных настроек на машины разработчиков для каждой ОС, и [Configure server-managed settings](https://code.claude.com/docs/en/server-managed-settings) для централизованного управления настройками из консоли claude.ai без MDM-систем.
- [Authentication](https://code.claude.com/docs/en/authentication#set-up-team-authentication): варианты настройки доступа команды или организации к Claude Code, и [Enterprise deployment overview](https://code.claude.com/docs/en/third-party-integrations) для выбора между развертыванием через Anthropic и через облачного провайдера.
- [Enterprise network configuration](https://code.claude.com/docs/en/network-config): прокси-серверы, корпоративные центры сертификации (CA) и mTLS; [Run Claude Code through a gateway](https://code.claude.com/docs/en/gateways) для централизованных учетных данных, мониторинга и контроля расходов.
- [Monitoring](https://code.claude.com/docs/en/monitoring-usage): экспорт метрик использования, расходов и активности инструментов через OpenTelemetry; [Track team usage with analytics](https://code.claude.com/docs/en/analytics) - дашборд использования и внедрения.
- [Data usage](https://code.claude.com/docs/en/data-usage): политики хранения данных и обучения моделей Anthropic, маршруты движения данных сессий.

Если в вашей организации развернуты управляемые настройки (managed settings), параметры из уроков могут не применяться на рабочих машинах: настройки пользователя, проекта, локальные файлы или параметр `--settings` не могут переопределить их, за редкими исключениями более строгих политик безопасности ([Deploy managed settings](https://code.claude.com/docs/en/managed-settings), [Configure server-managed settings](https://code.claude.com/docs/en/server-managed-settings#platform-availability)). При подключении разработчиков через шлюз (gateway) с учетными данными шлюза расходы тарифицируются по аккаунту провайдера организации по ставкам API, а индивидуальные подписки claude.ai не задействуются ([Subscriptions and gateways](https://code.claude.com/docs/en/gateways#subscriptions-and-gateways)).

<a id="roll-it-out-to-a-team"></a>
## Внедрение в команде

После настройки инфраструктуры эти материалы помогут коллегам освоить инструмент:

- [Champion kit](https://code.claude.com/docs/en/champion-kit): материалы для инженера, уже освоившего Claude Code и помогающего своей команде внедрить инструмент.
- [Communications kit](https://code.claude.com/docs/en/communications-kit): шаблоны анонсов запуска, цепочки писем и ответы на частые вопросы для администраторов и технических руководителей.
- [The AI-native SDLC playbook](https://academy.claude.com/courses/ai-native-sdlc-playbook): курс Claude Academy для руководителей инженерных направлений, платформенных команд и специалистов по безопасности, чьи организации уже используют Claude Code.
- [Running an AI-native engineering org](https://claude.com/resources/articles/running-an-ai-native-engineering-org): как изменились процессы и структура команды Claude Code после того, как разработка через агентов стала основным способом работы.

---

<sub>Sources: the pages linked above, each checked on 2026-10-09.</sub>

<sub>Up: [Продвинутый уровень (Advanced)](README.md)</sub>
