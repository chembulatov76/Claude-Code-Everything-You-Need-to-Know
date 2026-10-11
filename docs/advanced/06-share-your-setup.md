# Share your setup with a team

<sub>**Advanced** · Lesson 6 of 7 · about 20 minutes · Needs: [GitHub Actions](05-github-actions.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого урока вы сможете передавать настроенное окружение Claude Code всей команде: коммитить общие конфигурационные файлы, исключать из git личные настройки и предоставлять команде общий плагин, подгружаемый из локального каталога плагинов прямо внутри репозитория. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can share your Claude Code setup with a team: commit the settings everyone should get, keep your personal files out of git, and give the team a plugin that loads from a marketplace inside the repository.</details>

## You need

- Ваш собственный репозиторий с настройками уровня Intermediate: закоммиченный `.claude/settings.json` из [Урока 4 уровня Intermediate](../intermediate/04-permissions-and-sandbox.md), а также навыки, хуки и субагенты из уроков 1, 5 и 6. Настройте папку `.practice/` в соответствии с описанием в [Уроке 1 уровня Beginner](../beginner/01-install-and-look-around.md#you-need), если еще не сделали этого.
- Копия учебного шаблона с Node.js LTS и проверками уровня Advanced ([Практика](README.md#practice)): проверка вызывается из копии шаблона.
- bash, `curl` и git в среде macOS, Linux или WSL 2.
- Расход токенов: минимальный (light).

## The idea

Файлы, предназначенные для всей команды, фиксируются в системе контроля версий: `.claude/settings.json`, `CLAUDE.md`, `.claude/rules/`, общие навыки, субагенты и файл `.mcp.json` ([Explore the .claude directory](https://code.claude.com/docs/en/claude-directory#file-reference)). Личные настройки остаются вне git: `.claude/settings.local.json` и `CLAUDE.local.md` ([What's not shown](https://code.claude.com/docs/en/claude-directory#what%E2%80%99s-not-shown)). Локальный файл имеет более высокий приоритет, поэтому каждый разработчик может переопределить командную настройку под себя без правок в git, при этом списки (например, правила разрешений) объединяются ([Settings files and precedence](https://code.claude.com/docs/en/settings#settings-precedence)).

Плагин позволяет объединить навыки, субагентов и хуки в единый модуль для совместного использования в команде ([Create a Claude Code plugin](https://code.claude.com/docs/en/plugins/create#decide-when-to-use-a-plugin)), а каталог плагинов (marketplace) представляет собой директорию с файлом `.claude-plugin/marketplace.json`, перечисляющим доступные плагины ([Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace)). Закоммитьте каталог плагинов прямо внутри репозитория, укажите его в `extraKnownMarketplaces` через относительный путь с источником `directory` и активируйте плагин в `enabledPlugins`. Каждый участник команды получает плагин автоматически после подтверждения диалога доверия для папки репозитория ([Manage Claude Code plugins for your organization](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository)), без необходимости вручную запускать команды установки: плагин с относительным путем подгружается непосредственно из каталога ([Plugin loading reference](https://code.claude.com/docs/en/plugins/loading#enabled-in-project-settings-but-not-installed)).

## Worked example

Коллеги регулярно интересуются, как устроена конфигурация Claude Code в вашем репозитории. Предоставьте общий навык с объяснениями для каждого, кто клонирует проект.

1. В корне вашего репозитория загрузите [пример каталога плагинов](../../examples/advanced/06-share-your-setup/README.md) в папку `team-marketplace/`. Просмотрите каждый файл перед коммитом:

   ```bash
   base=https://raw.githubusercontent.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/main/examples/advanced/06-share-your-setup
   curl -fsSL --create-dirs "$base/dot-claude-plugin/marketplace.json" -o team-marketplace/.claude-plugin/marketplace.json
   for f in plugins/team-kit/.claude-plugin/plugin.json plugins/team-kit/skills/onboard/SKILL.md plugins/team-kit/agents/config-reviewer.md; do
     curl -fsSL --create-dirs "$base/$f" -o "team-marketplace/$f"
   done
   ```

   Каталог `team-tools` содержит один плагин `team-kit` по относительному пути `./plugins/team-kit`. Плагин содержит навык `onboard`, объясняющий совместную конфигурацию без правки файлов, и субагента `config-reviewer` с правами только на чтение и поиск. В репозитории руководства манифест хранится в `dot-claude-plugin/`, чтобы сам репозиторий не определялся как маркетплейс; команда `curl` сохраняет его в правильный путь `.claude-plugin/marketplace.json` ([Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace)).
2. Проверьте валидность каталога, а затем плагина:

   ```bash
   claude plugin validate ./team-marketplace
   claude plugin validate ./team-marketplace/plugins/team-kit --strict
   ```

   Каждая команда завершается выводом `✔ Validation passed` ([Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace#create-a-marketplace)). Проверка маркетплейса анализирует манифест и записи каталога, но не открывает файлы навыков и агентов внутри плагинов ([Validate a directory](https://code.claude.com/docs/en/plugins/cli-reference#validate-a-directory)), поэтому вторая команда валидирует непосредственно папку плагина, а флаг `--strict` превращает предупреждения в ошибки ([Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate)).
3. Подключите каталог в файле `.claude/settings.json`. Добавьте следующие ключи рядом с настройками из уроков Intermediate:

   ```json
   "extraKnownMarketplaces": {
     "team-tools": {
       "source": { "source": "directory", "path": "./team-marketplace" }
     }
   },
   "enabledPlugins": {
     "team-kit@team-tools": true
   }
   ```

   Ключом внутри `extraKnownMarketplaces` выступает поле `name` маркетплейса ([Manage Claude Code plugins for your organization](https://code.claude.com/docs/en/plugins/org#require-a-marketplace-and-its-plugins)), а идентификатор плагина формируется по шаблону `<entry name>@<marketplace name>`, в данном случае `team-kit@team-tools` ([Entry name and manifest name](https://code.claude.com/docs/en/plugins/loading#entry-name-and-manifest-name)). Источник `directory` задает путь к локальной папке в параметре `path` ([All settings](https://code.claude.com/docs/en/settings-reference#extraknownmarketplaces)). Относительный путь `path` разрешается относительно основного чекаута репозитория, поэтому сессия в worktree корректно загружает `team-marketplace/` основного дерева ([Require plugins per repository](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository)).
4. Закоммитьте оба компонента:

   ```bash
   git add team-marketplace .claude/settings.json
   git commit -m "Share the team's Claude Code setup"
   ```

   Если git сообщает об игнорировании пути, команда `git check-ignore -v <path>` покажет совпавшее правило; добавьте файл через `git add -f`.
5. Запустите `claude` в корне репозитория и примите диалог доверия, если он появится. После старта Claude Code регистрирует каталог из настроек и подгружает плагин `team-kit` без вызова команд установки ([Plugin loading reference](https://code.claude.com/docs/en/plugins/loading#plugins-and-marketplaces-that-aren%E2%80%99t-on-disk-at-session-start)). Наберите `/team-kit:`, чтобы увидеть команды плагина ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#commands)), и выполните `/team-kit:onboard` ([Add components to a plugin](https://code.claude.com/docs/en/plugins/components#skills)). Claude прочитает закоммиченные конфигурации и объяснит, какие настройки являются общими, а какие личными. Субагент плагина доступен под именем `team-kit:config-reviewer` ([Add components to a plugin](https://code.claude.com/docs/en/plugins/components#agents)): введите `@` и выберите его в списке для вызова ([Create custom subagents](https://code.claude.com/docs/en/sub-agents#invoke-subagents-explicitly)).
6. Отключите плагин только для себя. Добавьте следующий блок в файл `.claude/settings.local.json`:

   ```json
   "enabledPlugins": {
     "team-kit@team-tools": false
   }
   ```

   Сохраните файл и выполните команду `/reload-plugins` ([Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference#reload-plugins)). Команда `/team-kit:onboard` пропадет из вашей сессии, в то время как закоммиченный файл по-прежнему включает плагин для всех остальных коллег: локальный файл настроек имеет преимущество ([All settings](https://code.claude.com/docs/en/settings-reference#enabledplugins)). Затем удалите запись из локального файла, сохраните его и снова выполните `/reload-plugins`. Если `git status` показывает файл `.claude/settings.local.json`, не коммитьте его: раздел Ваша очередь добавляет его в `.gitignore`.

## Your turn

**Задание:** перенесите один компонент из ваших настроек Intermediate в состав плагина `team-kit`, чтобы каждый клонирующий репозиторий коллега получал его автоматически. Выберите навык (например, `tdd` из Урока 1 Intermediate) либо субагента (например, `test-gaps` из Урока 6). В качестве более сложного варианта можно перенести хук из Урока 5: запись размещается в `hooks/hooks.json` плагина в формате файла настроек, а сам скрипт вызывается через переменную `${CLAUDE_PLUGIN_ROOT}` ([Add components to a plugin](https://code.claude.com/docs/en/plugins/components#hooks)).

**Критерии приемки:**

- Компонент размещен в `team-marketplace/plugins/team-kit/` по пути `skills/<name>/SKILL.md`, `agents/<name>.md` либо в `hooks/hooks.json`, а исходный файл удален из `.claude/`; перенесенный хук также удален из `.claude/settings.json`.
- Команда `claude plugin validate ./team-marketplace/plugins/team-kit --strict` завершается успешно. Сохраните отчет валидации для проверки из корня репозитория: `claude plugin validate ./team-marketplace/plugins/team-kit --strict --json > .practice/a-6-validate.json`.
- Закоммиченный `.gitignore` содержит `.claude/settings.local.json` и `CLAUDE.local.md`, и ни один из этих файлов не отслеживается в git. Автоматическое правило Claude Code для локальных настроек добавляется в глобальный файл исключений (не разделяемый с коллегами) ([Settings files and precedence](https://code.claude.com/docs/en/settings#keep-personal-settings-out-of-a-repository)), а файл `CLAUDE.local.md` создается вручную и добавляется в `.gitignore` самостоятельно ([Explore the .claude directory](https://code.claude.com/docs/en/claude-directory#what%E2%80%99s-not-shown)). Добавьте правило даже при отсутствии `CLAUDE.local.md`.
- Все изменения закоммичены, и в новой сессии компонент вызывается под префиксом плагина (например, `/team-kit:tdd` или субагент `team-kit:test-gaps`).

Так как плагин `team-kit` указан по относительному пути в локальном каталоге на диске, Claude Code загружает его прямо с места: правки в `team-marketplace/` вступают в силу при следующем старте сессии или после вызова `/reload-plugins` без изменения версии ([Plugin loading reference](https://code.claude.com/docs/en/plugins/loading#in-place-and-copied-plugins)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] Закоммиченный каталог плагинов указывает на плагин по относительному пути, и имя записи совпадает с полем `name` в манифесте `plugin.json`.
- [ ] Закоммиченный файл `.claude/settings.json` регистрирует маркетплейс по относительному пути и включает плагин.
- [ ] Перенесенный компонент находится внутри плагина и удален из `.claude/`.
- [ ] Закоммиченный `.gitignore` исключает файлы `.claude/settings.local.json` и `CLAUDE.local.md`, и git не отслеживает их.
- [ ] Файл `.practice/a-6-validate.json` содержит успешно пройденную строгую валидацию директории плагина.
- [ ] Самопроверка (не тестируется): в новой сессии в вашем репозитории после подтверждения доверия перенесенный компонент подгружается из `team-kit` без команд установки.

Запустите команду `npm run check -- a-6 --dir <your repo>` из вашей копии учебного шаблона.

Если первый пункт завершается ошибкой, выполните `claude plugin validate ./team-marketplace` и сопоставьте имя записи `name` с `plugin.json`; папка с именем `dot-claude-plugin/` не станет маркетплейсом, пока вы не переименуете ее в `.claude-plugin/`. Если второй пункт не проходит, проверьте ключ маркетплейса в `extraKnownMarketplaces`, относительный путь (`./team-marketplace`) и булево значение `true` (не строку `"true"`). Если не проходит третий пункт, проверьте `git status`: исходный файл мог остаться в `.claude/` или копия в плагине не закоммичена. Если четвертый пункт не проходит, выполните `git rm --cached` для отслеживаемых файлов, внесите строки в `.gitignore` и сделайте коммит. Если не проходит пятый пункт, повторите валидацию с флагом `--strict` из корня репозитория и сохраните JSON-отчет.

## Watch out

- Любой разработчик, подтверждающий доверие репозиторию, выполняет содержимое `team-kit` со своими правами. Хуки плагина и процессы MCP-серверов выполняются вне песочницы, а правила доступа не ограничивают их исполнение ([Plugin security and trust](https://code.claude.com/docs/en/plugins/security#understand-what-a-plugin-can-do)). Проводите ревью изменений в `team-marketplace/` так же строго, как ревью исходного кода.
- Каталог плагинов подключается только после того, как пользователь примет диалог доверия для папки, а запуск `claude -p` подключает его только при ранее принятом доверии в интерактивной сессии или наличии флага `hasTrustDialogAccepted` в `~/.claude.json` ([Manage Claude Code plugins for your organization](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository)); облачные сессии никогда не подключают такие плагины, так как в них нет диалога доверия ([Plugin loading reference](https://code.claude.com/docs/en/plugins/loading#plugins-shared-through-a-repository)). Закоммиченные правила доступа ожидают того же диалога ([Configure permissions](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust)).
- Компонент, оставленный одновременно в `.claude/` и внутри плагина, будет загружен дважды: команды `/tdd` и `/team-kit:tdd` сработают обе, а хук сработает дважды на каждое событие ([Create a Claude Code plugin](https://code.claude.com/docs/en/plugins/create#convert-an-existing-claude-setup)).

## Go further

- [Хостинг и поддержка каталога плагинов](https://code.claude.com/docs/en/plugins/host-marketplace): перенос каталога в собственный репозиторий при использовании в нескольких проектах и публикация версий.
- [Управление плагинами Claude Code в организации](https://code.claude.com/docs/en/plugins/org): обязательная установка или ограничение каталогов и плагинов на рабочих машинах через managed settings.
- [Пример общих настроек для команды](https://code.claude.com/docs/en/settings-example#a-teams-shared-settings): готовый пример закоммиченного файла настроек для сопоставления с вашей конфигурацией.

---

<sub>Sources: [Settings files and precedence](https://code.claude.com/docs/en/settings) · [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory) · [Create a Claude Code plugin](https://code.claude.com/docs/en/plugins/create) · [Manage Claude Code plugins for your organization](https://code.claude.com/docs/en/plugins/org) · [Plugin loading reference](https://code.claude.com/docs/en/plugins/loading) · [Create a marketplace](https://code.claude.com/docs/en/plugins/create-marketplace) · [Plugin commands reference](https://code.claude.com/docs/en/plugins/cli-reference) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Add components to a plugin](https://code.claude.com/docs/en/plugins/components) · [Create custom subagents](https://code.claude.com/docs/en/sub-agents) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Plugin security and trust](https://code.claude.com/docs/en/plugins/security) · [Configure permissions](https://code.claude.com/docs/en/permissions)</sub>

<sub>← [GitHub Actions](05-github-actions.md) · [Advanced index](README.md) · [Установка ограничений для автономных запусков](07-bounded-runs.md) → · Topic: [Плагины](../topics/plugins.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-6)</sub>
