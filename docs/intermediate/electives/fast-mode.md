# Fast mode

<sub>**Intermediate** · Elective · about 10 minutes · Needs: [Pick the model and effort](../03-model-and-effort.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

> [!NOTE]
> "Fast mode is in research preview. The feature, pricing, and availability may change based on feedback." ([Speed up responses with fast mode](https://code.claude.com/docs/en/fast-mode))

## Goal

К концу этого факультатива вы сможете понимать, в каких случаях режим Fast mode оправдывает свою стоимость, а также включать и отключать его. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can tell when fast mode is worth its cost, and turn it on and off.</details>

## You need

- Claude Code с авторизацией по тарифному плану Claude либо через учётную запись Claude Console, организация которой имеет доступ к Fast mode. Fast mode недоступен на платформах Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry и Claude Platform on AWS ([Requirements](https://code.claude.com/docs/en/fast-mode#requirements)).
- Для планов Pro, Max, Team или Enterprise: включённые кредиты использования (usage credits). Команда `/usage-credits` открывает страницу их подключения; на планах Team и Enterprise участники без прав биллинга могут запросить включение у администратора.
- Расход токенов: тарифицируется сверх базового плана (usage credits). Fast mode расходует usage credits даже при наличии неизрасходованных лимитов подписки. Выполнение упражнения является опциональным.

## The idea

Fast mode запускает модель Claude Opus со специальной конфигурацией API, обеспечивающей значительно более быстрые ответы по более высокой цене за токен. Это та же самая модель с неизменным качеством рассуждений ([Speed up responses with fast mode](https://code.claude.com/docs/en/fast-mode)). Режим действует только для моделей Opus из списка документации; его включение автоматически переключает сессию на Opus, если была выбрана другая модель. <details style="display:inline"><summary><code>[en]</code></summary>Fast mode runs Claude Opus with an API configuration that answers faster, at a higher price per token. It's the same model, with the same quality</details>

Используйте Fast mode для интерактивной совместной работы, где время ожидания ответа критичнее стоимости токенов: быстрая итеративная отладка, парное программирование. Держите его выключенным для длительных автономных фоновых задач, CI и чувствительных к расходам сценариев. В подписках каждый токен fast mode оплачивается строго из usage credits, а не из включённого объёма плана.

Включение режима в середине беседы приводит к однократной тарификации всего накопленного контекста по ставке fast mode без учёта кэша, поэтому включайте его в самом начале сессии.

## Worked example

1. Запустите `claude` и выполните `/fast`. Если кредиты использования выключены, утилита сообщит: `Fast mode requires usage credits`.
2. Чтобы включить режим, нажмите `Space` в панели `/fast`, затем `Enter`. Claude Code подтвердит активацию `Fast mode ON`, рядом с полем ввода появится значок `↯`, а модель переключится на Opus.
3. Повторно выполните `/fast`, отключите режим и подтвердите выбор. Сессия останется на Opus; при необходимости вернитесь на прежнюю модель командой `/model`.

## Your turn

Это упражнение опционально, так как расходует платные usage credits.

1. Запустите новую сессию, активируйте Fast mode до первого промпта и проведите несколько минут быстрой интерактивной работы.
2. Выключите режим и проверьте списания: в разделе **Usage credits** настроек **Settings > Usage** на сайте claude.ai (для Pro и Max); через `/usage` (для Team и Enterprise); либо в консоли Claude Console на страницах **Usage** и **Cost** ([See where fast mode spend appears](https://code.claude.com/docs/en/fast-mode#see-where-fast-mode-spend-appears)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Команда `/fast` показывает статус режима или сообщает, какие настройки аккаунта требуются для его включения.
- [ ] Self-check (not tested): вы можете назвать задачу, для которой оправдан Fast mode, и задачу, где его использовать не стоит, а также понимаете структуру его тарификации.

Если команда `/fast` выводит `Fast mode has been disabled by your organization`, значит, администраторы вашей организации Team/Enterprise отключили эту функцию в корпоративных политиках.

## Go further

- [Fast mode vs effort level](https://code.claude.com/docs/en/fast-mode#fast-mode-vs-effort-level): сравнение двух способов ускорения ответов и их комбинация.
- [Speed up responses with fast mode](https://code.claude.com/docs/en/fast-mode): официальная документация по возможностям режима.

---

<sub>Sources: [Speed up responses with fast mode](https://code.claude.com/docs/en/fast-mode) · [Manage costs effectively](https://code.claude.com/docs/en/costs)</sub>

<sub>Back to the [Intermediate index](../README.md) · Topic: [Models, effort and cost](../../topics/models-effort-and-cost.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-fast)</sub>
