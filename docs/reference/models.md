<a id="models--specifications--pricing"></a><a id="models-and-effort"></a>
# Модели и глубина рассуждений

<sub>Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

Выбор модели для сессии и настройка глубины ее рассуждений - это два независимых решения. Данная страница служит справочником по обоим аспектам: псевдонимы моделей, уровни рассуждений (effort) и способы их установки. Выбору оптимальной модели и уровня усилий для конкретной задачи посвящен урок промежуточного уровня [Выбор модели и глубины рассуждений](../intermediate/03-model-and-effort.md). Сравнение актуальных моделей и их идентификаторы доступны в [models overview](https://platform.claude.com/docs/en/about-claude/models/overview).

<a id="the-current-lineup-july-2026"></a><a id="the-current-lineup-as-of-october-4-2026"></a><a id="the-headliners"></a><a id="claude-sonnet-5--the-new-default"></a><a id="claude-sonnet-5-legacy"></a><a id="claude-opus-48--the-opus-flagship"></a><a id="claude-opus-48-legacy"></a><a id="claude-fable-5--the-mythos-class"></a><a id="claude-fable-5-legacy"></a><a id="aliases"></a>
## Псевдонимы (Aliases)

Псевдоним автоматически выбирает рекомендуемую модель семейства без необходимости указывать точный номер версии. Задать псевдоним можно командами `/model <alias>`, `claude --model <alias>` или через настройку `model` ([Model configuration](https://code.claude.com/docs/en/model-config#model-aliases)).

| Псевдоним | В Anthropic API соответствует | Когда использовать |
|---|---|---|
| `sonnet` | Актуальная версия Sonnet | Повседневная разработка |
| `opus` | Актуальная версия Opus | Сложный архитектурный анализ и рассуждения |
| `fable` | Модель класса Fable для вашего провайдера | Самые сложные и продолжительные автономные задачи |
| `best` | Fable (если доступна на аккаунте), иначе Opus | Требуется максимально мощная модель из доступных |
| `haiku` | Haiku | Быстрые простые задачи |
| `opusplan` | Opus в режиме планирования, Sonnet для реализации плана | Глубокое проектирование при экономичном исполнении |
| `opus[1m]` | Opus с контекстным окном в 1 миллион токенов | Очень длинные сессии у провайдеров, где у `opus` окно меньше |

- Псевдоним со временем переключается на более новые модели семейства. Чтобы зафиксировать точную версию, укажите ее полное имя (например, `claude-opus-5-5`) из [models overview](https://platform.claude.com/docs/en/about-claude/models/overview).
- Имя `default` не является псевдонимом: оно сбрасывает выбранную модель к значению по умолчанию для вашего аккаунта.
- У сторонних облачных провайдеров псевдоним может указывать на предыдущие поколения моделей: `sonnet` на Claude Platform on AWS, Amazon Bedrock, Google Cloud Agent Platform и Microsoft Foundry, а `opus` на Microsoft Foundry. Полный список приведен в [alias table](https://code.claude.com/docs/en/model-config#model-aliases).
- Модели класса Fable могут расходовать дополнительные кредиты использования в зависимости от вашего плана и типа лицензии ([Fable and usage credits](https://code.claude.com/docs/en/model-config#fable-and-usage-credits)).

Модель по умолчанию зависит от типа учетной записи и провайдера. Проверить текущую модель можно командами `/model` или `/status`, а правила разрешения описаны в [Default model setting](https://code.claude.com/docs/en/model-config#default-model-setting).

<a id="effort-levels"></a>
## Уровни рассуждений (Effort levels)

Уровень усилий (effort) определяет глубину логических рассуждений модели на каждом шаге. Низкий уровень дает быстрые ответы и тратит меньше токенов; высокий уровень выполняет глубокую самопроверку ([Adjust effort level](https://code.claude.com/docs/en/model-config#adjust-effort-level)).

| Уровень | Для каких задач подходит |
|---|---|
| `low` | Быстрые взаимодействия с вашим контролем каждого шага: набросок идеи, переименование |
| `medium` | Повседневная работа с четкими границами (например, новая функциональность) на моделях, где этот уровень дефолтный; в остальных случаях - более экономный режим |
| `high` | Задачи с вероятными граничными случаями (исправление багов в существующем коде) |
| `xhigh` | Глубокий анализ ценой повышенного расхода токенов |
| `max` | Сложнейшие задачи, решаемые Claude автономно; требует предварительного тестирования, так как модель склонна к избыточному анализу |

- Доступные уровни и значение по умолчанию различаются в зависимости от модели: см. [levels per model](https://code.claude.com/docs/en/model-config#adjust-effort-level) в документации. Команда `/effort status` выводит активный уровень.
- При указании неподдерживаемого уровня Claude Code выбирает наивысший доступный уровень ниже указанного.
- Шкала откалибрована индивидуально под каждую модель, поэтому `high` для одной модели не равен объему рассуждений `high` для другой.

<a id="setting-effort"></a>
## Настройка уровня усилий

| Где настраивается | Срок действия настройки |
|---|---|
| `/effort <level>` или `/effort` для вызова ползунка | Сохраняется как настройка по умолчанию для данной модели; клавиша `s` в ползунке применяет только к текущей сессии |
| Ползунок усилий в команде `/model` | Аналогично вызову `/effort` |
| Флаг командной строки `claude --effort <level>` | Только текущая сессия |
| Переменная окружения `CLAUDE_CODE_EFFORT_LEVEL` | Все сессии с этой переменной (переопределяет другие источники, но подчиняется ограничению `maxEffortLevel`) |
| Параметры `modelSettings` или `effortLevel` в конфигурационном файле | Все сессии в области видимости файла. `modelSettings` задает уровень конкретной модели; верхнеуровневый `effortLevel` в пользовательских настройках не действует на новейшие модели |
| Поле `effort` во frontmatter навыка или субагента | На время выполнения этого навыка или субагента |

Команда `/effort auto` сбрасывает сохраненный уровень для текущей модели: после этого модель использует `effortLevel` из настроек (если задан) либо собственное значение по умолчанию ([Set the effort level](https://code.claude.com/docs/en/model-config#set-the-effort-level)).

<a id="gotchas"></a>
## Важные нюансы

- Уровень `max` действует только на текущую сессию, если не задан через `CLAUDE_CODE_EFFORT_LEVEL`. Параметры конфигурации `effortLevel` и `modelSettings` его не принимают.
- Ключевое слово `ultrathink` в запросе включает углубленные рассуждения строго на один шаг и не меняет общий уровень effort. Фразы вроде «подумай получше» воспринимаются как обычный текст ([Use ultrathink](https://code.claude.com/docs/en/model-config#use-ultrathink-for-one-off-deep-reasoning)).
- Ultracode является настройкой поведения, а не уровнем effort: при его включении Claude строит план [динамического воркфлоу](https://code.claude.com/docs/en/workflows) для каждой комплексной задачи на текущем уровне effort сессии ([Model configuration](https://code.claude.com/docs/en/model-config#adjust-effort-level)).
- Смена модели посреди сессии приводит к пересчету всего контекста запроса (на большинстве моделей изменение effort делает то же самое), поэтому Claude Code может предварительно запросить подтверждение ([Prompt caching](https://code.claude.com/docs/en/prompt-caching)).

<a id="fast-mode"></a><a id="related-pages"></a>
## Связанные разделы

- Быстрый режим (Fast mode), отмеченный в документации как research preview: ускоренная конфигурация поддерживаемых моделей Opus, оплачиваемая в тарифах по подписке только кредитными баллами (usage credits). См. [Fast mode](https://code.claude.com/docs/en/fast-mode) и факультатив [Быстрый режим](../intermediate/electives/fast-mode.md).
- Выбор модели и уровня рассуждений для задачи: статья Anthropic [Choosing a Claude model and effort level in Claude Code](https://claude.com/blog/claude-model-and-effort-level-in-claude-code).
- Даты вывода моделей из эксплуатации: [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations).

<a id="pricing"></a><a id="subscription-plans"></a><a id="api-pay-as-you-go"></a>
В данном руководстве не приводятся цены и тарифные лимиты. Актуальные данные доступны на страницах [claude.com/pricing](https://claude.com/pricing) (подписки) и [API pricing](https://platform.claude.com/docs/en/about-claude/pricing) (тарификация токенов).

---

<sub>Sources: [Model configuration](https://code.claude.com/docs/en/model-config) · [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview) · [Prompt caching](https://code.claude.com/docs/en/prompt-caching) · [Fast mode](https://code.claude.com/docs/en/fast-mode) · [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) · [Pricing](https://claude.com/pricing) · [API pricing](https://platform.claude.com/docs/en/about-claude/pricing)</sub>

<sub>Up: [Reference](README.md)</sub>
