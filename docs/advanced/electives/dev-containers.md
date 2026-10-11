# Dev containers

<sub>**Advanced** · Elective · about 20 minutes, plus run time · Needs: [Put bounds on autonomous runs](../07-bounded-runs.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

## Goal

К концу этого факультатива вы сможете запускать Claude Code внутри dev-контейнера, сохранять авторизацию при пересборках контейнера и обеспечивать его автономную работу под защитой фаервола.

## You need

- Для раздела Worked example: ваша копия учебного шаблона на GitHub и активный GitHub Codespaces на вашем аккаунте. На компьютер ничего устанавливать не нужно: dev-контейнер шаблона устанавливает Claude Code автоматически.
- Для Your turn: один из ваших собственных репозиториев без папки `.devcontainer/` с настроенной папкой `.practice/` (как описано в [уроке 1 для начинающих](../../beginner/01-install-and-look-around.md#you-need)).
- Также для Your turn: Docker, VS Code с расширением [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ([System requirements](https://code.visualstudio.com/docs/devcontainers/containers#_system-requirements)) и утилита `jq` на macOS, Linux или WSL 2. Без локального Docker выполните Worked example и самопроверку из раздела Check.
- Расход лимитов: небольшой. Codespace также расходует вычислительные ресурсы и хранилище учетной записи GitHub Codespaces во время работы и хранения ([GitHub Codespaces billing](https://docs.github.com/en/billing/concepts/product-billing/github-codespaces)).

## The idea

[Dev-контейнер](https://code.claude.com/docs/en/devcontainer) - это Docker-контейнер, к которому подключается ваш редактор локально или в облаке (например, в GitHub Codespaces). При интеграции с Claude Code команды Claude выполняются внутри контейнера, а результаты редактирования файлов сразу попадают в ваш рабочий репозиторий. В отличие от Bash-песочницы из [урока 4 среднего уровня](../../intermediate/04-permissions-and-sandbox.md), оборачивающей только команды оболочки, контейнер изолирует весь процесс Claude Code целиком: файловые операции, MCP-серверы и хуки ([Compare sandboxing approaches](https://code.claude.com/docs/en/sandbox-environments#compare-sandboxing-approaches)).

Конфигурация строится из трех последовательных элементов:

- **Фича Claude Code Dev Container** добавляет CLI в любой dev-контейнер ([Add Claude Code to your dev container](https://code.claude.com/docs/en/devcontainer#add-claude-code-to-your-dev-container)).
- **Именованный том (named volume)** по пути `~/.claude` с переменной окружения `CLAUDE_CONFIG_DIR`, указывающей на него, сохраняет вашу авторизацию при пересборках контейнера, когда домашняя папка сбрасывается ([Persist authentication and settings across rebuilds](https://code.claude.com/docs/en/devcontainer#persist-authentication-and-settings-across-rebuilds)).
- **Эталонный контейнер (reference container)** добавляет пользователя без прав root и фаервол, фильтрующий исходящий трафик строго по разрешенному списку адресов. Документация рекомендует его для автономных запусков с флагом `--dangerously-skip-permissions` (режим обхода подтверждений); при запуске от root этот флаг блокируется ([Run without permission prompts](https://code.claude.com/docs/en/devcontainer#run-without-permission-prompts), [Dev containers](https://code.claude.com/docs/en/sandbox-environments#dev-containers)).

> [!WARNING]
> При использовании `--dangerously-skip-permissions` «dev-контейнеры не защищают от утечки данных, доступных внутри контейнера, включая учетные данные Claude Code в `~/.claude`» ([Development containers](https://code.claude.com/docs/en/devcontainer)). Расширение VS Code Dev Containers также пробрасывает внутрь контейнера локальные учетные данные Git и работающий SSH-агент, а эталонный фаервол разрешает доступ к GitHub и SSH, поэтому агент может выполнять push ([Sharing Git credentials with your container](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials)). Используйте контейнеры только с доверенными репозиториями и никогда не монтируйте внутрь секреты хоста вроде `~/.ssh` или файлов облачных провайдеров.

## Worked example

1. На GitHub откройте `.devcontainer/devcontainer.json` в вашей копии учебного шаблона:

   ```json
   {
     "name": "Claude Code practice",
     "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
     "features": {
       "ghcr.io/devcontainers/features/node:1": { "version": "lts" },
       "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {}
     },
     "postCreateCommand": "sudo apt-get update && sudo apt-get install -y bubblewrap socat"
   }
   ```

   Тег `:1.0` фиксирует версию скрипта установки фичи, а не самого Claude Code: фича устанавливает последний доступный релиз, который в дальнейшем обновляется сам ([Add Claude Code to your dev container](https://code.claude.com/docs/en/devcontainer#add-claude-code-to-your-dev-container)). Команда `postCreateCommand` устанавливает `bubblewrap` и `socat`, необходимые для песочницы Bash в Linux ([Set up Linux and WSL2](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2)).
2. Нажмите **Code**, перейдите на вкладку **Codespaces** и создайте Codespace на ветке `main` ([Creating a codespace for a repository](https://docs.github.com/en/codespaces/developing-in-a-codespace/creating-a-codespace-for-a-repository)). Он откроется в редакторе по умолчанию из ваших настроек ([Setting your default editor for GitHub Codespaces](https://docs.github.com/en/codespaces/setting-your-user-preferences/setting-your-default-editor-for-github-codespaces)); шаги 3-6 выполняются в VS Code (в браузере или в настольном приложении).
3. В терминале Codespace выполните:

   ```bash
   echo "$CODESPACES"; whoami; claude --version
   ```

   Команда выводит три строки: `true` (GitHub устанавливает переменную `CODESPACES` в каждом codespace, см. [Default environment variables for your codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/default-environment-variables-for-your-codespace)), `vscode` (пользователь без root из базового образа, [Ubuntu image](https://github.com/devcontainers/images/tree/main/src/base-ubuntu)) и версию утилиты, которая может быть свежее проверочной отметки страницы.
4. Запустите `claude`, авторизуйтесь и подтвердите доверие к папке. Если браузер отображает код вместо прямого перенаправления в терминал, вставьте его в приглашение `Paste code here if prompted` ([Log in to Claude Code](https://code.claude.com/docs/en/authentication#log-in-to-claude-code)). Затем задайте вопрос:

   ```text
   Run uname -a and whoami, then tell me which machine you're running on.
   ```

   Подтвердите выполнение команд при запросе. Claude опишет окружение Linux-контейнера, а не ваш компьютер (формулировка ответа может варьироваться). Завершите сессию командой `/exit`.
5. Проверьте статус авторизации: `claude auth status --text` (вы авторизованы, см. [CLI commands](https://code.claude.com/docs/en/cli-reference#cli-commands)). Пересоберите контейнер: откройте палитру команд (`Ctrl+Shift+P` или `Cmd+Shift+P` на Mac), выберите команду **Codespaces: Rebuild Container** и подтвердите **Rebuild** ([Rebuilding the container in a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/rebuilding-the-container-in-a-codespace)). После повторного подключения выполните:

   ```bash
   claude auth status --text; echo "exit $?"
   ```

   Команда завершается с `exit 1` (нет авторизации). Пересборка сохраняет рабочую директорию `/workspaces` с вашим кодом, но сбрасывает домашнюю папку `~/.claude` ([Persisting data over a rebuild](https://docs.github.com/en/codespaces/developing-in-a-codespace/rebuilding-the-container-in-a-codespace#persisting-data-over-a-rebuild)).
6. Остановите codespace: вызовите команду **Codespaces: Stop Codespace** из палитры или выберите **Stop codespace** в меню **...** на странице [github.com/codespaces](https://github.com/codespaces). Простое закрытие вкладки браузера не останавливает контейнер ([Stopping and starting a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/stopping-and-starting-a-codespace)).

## Your turn

**Задание:** поместите один из ваших собственных репозиториев в [эталонный контейнер](https://code.claude.com/docs/en/devcontainer#try-the-reference-container) со включенным фаерволом и фиксированной версией Claude Code. Сохраните авторизацию между пересборками и выполните одну небольшую задачу в автономном режиме, после чего проверьте результат на хост-машине.

В официальной документации эталонный контейнер разворачивается для репозитория самого Claude Code; для использования в своем проекте скопируйте папку `.devcontainer/` и адаптируйте Dockerfile под инструменты вашего стека ([Try the reference container](https://code.claude.com/docs/en/devcontainer#try-the-reference-container)). На новой ветке скачайте [три файла](https://github.com/anthropics/claude-code/tree/d945a61bc6346abce607252d5667df8a3bf0461a/.devcontainer) с коммита, на котором проверялось руководство:

```bash
git switch -c try-devcontainer
mkdir -p .devcontainer
ref=d945a61bc6346abce607252d5667df8a3bf0461a
for f in devcontainer.json Dockerfile init-firewall.sh; do
  curl -fsSL "https://raw.githubusercontent.com/anthropics/claude-code/$ref/.devcontainer/$f" -o ".devcontainer/$f"
done
```

**Критерии приемки:**

- Вы прочитали все три файла перед первой сборкой. Скрипт `init-firewall.sh` выполняется от root при каждом старте контейнера: файл `devcontainer.json` запускает его в `postStartCommand` ([Dev Container metadata reference](https://containers.dev/implementors/json_reference/)), а Dockerfile разрешает пользователю `node` запускать его через `sudo`.
- До первой сборки вы добавили домены `claude.ai` и `platform.claude.com` в список резолвинга `init-firewall.sh` рядом с `api.anthropic.com`: они требуются для входа и обновления токенов ([Network access requirements](https://code.claude.com/docs/en/network-config#network-access-requirements)). Хосты могут иметь одинаковый IP, а скрипт прерывается при попытке повторного добавления адреса, поэтому в том же цикле замените `ipset add allowed-domains "$ip"` на `ipset add -exist allowed-domains "$ip"`: флаг `-exist` игнорирует уже существующие записи ([ipset](https://ipset.netfilter.org/ipset.man.html)).
- Файл `devcontainer.json` фиксирует версию Claude Code: аргумент сборки `CLAUDE_CODE_VERSION` указывает конкретную версию (например, `2.1.285`) вместо `latest`, а параметр `containerEnv` содержит `"DISABLE_AUTOUPDATER": "1"`. Это стандартный подход: установка фиксированной версии в Dockerfile и отключение автообновлений ([Enforce organization policy](https://code.claude.com/docs/en/devcontainer#enforce-organization-policy)). Все три файла с вашими правками закоммичены в ветку.
- Команда **Dev Containers: Reopen in Container** выполняет сборку и открывает проект. Предупреждения `npm warn EBADENGINE` в логах сборки нормальны: эталонный образ использует Node.js 20, а npm-пакет запрашивает версию 22+, но npm лишь выводит предупреждение, установка завершается, и утилита `claude` запускается без сбоев ([Install with npm](https://code.claude.com/docs/en/setup#install-with-npm)). В терминале контейнера (`` Ctrl+` ``) команда `whoami` выводит `node`, `claude --version` выводит зафиксированную версию, запрос `curl -sS --connect-timeout 5 https://example.com` падает по тайм-ауту, а `curl -sS --connect-timeout 5 https://api.github.com/zen` завершается успешно.
- Вы выполнили авторизацию в `claude` один раз. После команды **Dev Containers: Rebuild Container** вызов `claude auth status` по-прежнему завершается с кодом `0`: эталон монтирует именованный том в `/home/node/.claude` и направляет туда `CLAUDE_CONFIG_DIR`, а Docker сохраняет тома при удалении контейнеров ([Volumes](https://docs.docker.com/engine/storage/volumes/)).
- В терминале контейнера завершен один автономный прогон на задаче, не требующей отсутствующих в образе инструментов, например: `claude -p "Add a one-line summary of this repository to the top of README.md" --dangerously-skip-permissions --model sonnet --max-turns 10` ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)).
- В терминале хост-машины (после проверок из первого пункта Watch out) сохранено состояние: `git status --porcelain > .practice/a-dc-status.txt`, после чего вы просмотрели `git diff` (включая папку `.devcontainer/`) перед коммитом или отменой правок.

После успешного выполнения всех пунктов Check выполните демонтаж окружения:

1. В терминале контейнера выполните `claude auth logout` ([CLI commands](https://code.claude.com/docs/en/cli-reference#cli-commands)). До выхода учетные данные сохраняются в томе конфигурации, который переживает удаление контейнера.
2. Закройте окно VS Code, подключенное к контейнеру. Это останавливает контейнер по умолчанию (`shutdownAction` в [Dev Container metadata reference](https://containers.dev/implementors/json_reference/)).
3. Удалите контейнер: в Remote Explorer редактора VS Code выберите **Containers**, нажмите правой кнопкой мыши по контейнеру и выберите удаление ([Managing containers](https://code.visualstudio.com/docs/devcontainers/containers#_managing-containers)) либо выполните `docker rm <container>` ([docker container rm](https://docs.docker.com/reference/cli/docker/container/rm/)).
4. Удалите два тома с префиксами `claude-code-config-` и `claude-code-bashhistory-`: команда `docker volume ls` покажет их имена, после чего удалите их через `docker volume rm <name>`. Docker не позволит удалить том, пока контейнер существует, поэтому данный шаг выполняется строго после шага 3 ([docker volume rm](https://docs.docker.com/reference/cli/docker/volume/rm/)).

## Check

Задание выполнено, когда выполняются все условия:

- [ ] На компьютере `git ls-tree --name-only try-devcontainer .devcontainer/` содержит три файла, а `git show try-devcontainer:.devcontainer/devcontainer.json | jq -r '.build.args.CLAUDE_CODE_VERSION, .containerEnv.DISABLE_AUTOUPDATER'` выводит выбранную версию и `1`.
- [ ] Внутри контейнера `whoami` выводит `node`, `claude --version` выводит вашу версию, запрос к `example.com` завершается ошибкой, а запрос к `api.github.com/zen` успешен.
- [ ] Внутри контейнера после пересборки команда `claude auth status >/dev/null; echo $?` возвращает `0`.
- [ ] На основном компьютере файл `.practice/a-dc-status.txt` содержит список файлов, измененных автономным прогоном.
- [ ] Самопроверка (не проверяется скриптом): в codespace команда `claude auth status` возвращала `1` после пересборки, и вы можете назвать две вещи, которые контейнер не защищает: любые доступные внутри файлы (включая `~/.claude` и проброшенные ключи Git) и файлы смонтированного проекта.

Если jq выдает ошибку парсинга, удалите комментарии или лишние запятые в `devcontainer.json`. Если `example.com` отвечает или `api.github.com` недоступен, фаервол не применился: выполните `sudo /usr/local/bin/init-firewall.sh` в контейнере и изучите ошибку. Сообщение `Element cannot be added to the set: it's already added` означает отсутствие ключа `-exist`. Если после пересборки `claude auth status` возвращает `1`, путь монтирования тома `target` и переменная `CLAUDE_CONFIG_DIR` в `containerEnv` должны указывать на один и тот же путь ([Persist authentication and settings across rebuilds](https://code.claude.com/docs/en/devcontainer#persist-authentication-and-settings-across-rebuilds)). Если файл статуса пуст, автономная сессия ничего не изменила, сделала коммит сама (`git log -1` покажет его) или уперлась в лимит `--max-turns` с ошибкой ([CLI flags](https://code.claude.com/docs/en/cli-reference#cli-flags)): прочитайте вывод и задайте более компактную задачу.

## Watch out

- При указании `--dangerously-skip-permissions` Claude может перезаписывать `.devcontainer/` и `.git/`: формально это защищенные пути, но режим обхода изменяет их без запроса подтверждения ([Protected paths](https://code.claude.com/docs/en/permission-modes#protected-paths)). Через монтирование папки эти правки мгновенно попадают на хост-машину: следующая пересборка задействует измененный `.devcontainer/`, а git будет запускать хуки из `.git/hooks` или пути из `.git/config` при локальных командах вроде `git commit` ([githooks](https://git-scm.com/docs/githooks)). До пересборки или вызова git на компьютере обязательно проверьте `.git/config` и содержимое `.git/hooks`, а затем сравните `.devcontainer/` с коммитом (`git log --oneline`, затем `git diff <your commit> -- .devcontainer`).
- Созданный codespace работает до ручной остановки или тайм-аута, а в остановленном виде расходует квоту дискового пространства ([Stopping and starting a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/stopping-and-starting-a-codespace)). Удаляйте его после завершения работы в меню **...** на [github.com/codespaces](https://github.com/codespaces) или через команду `gh codespace delete` ([Deleting a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/deleting-a-codespace)).
- Фаервол блокирует весь исходящий трафик, не разрешенный в скрипте, для любых процессов внутри контейнера. Если вашему проекту требуются другие хосты (PyPI, приватный репозиторий пакетов и т.д.), добавьте их домены в скрипт и пересоберите контейнер ([Dev containers](https://code.claude.com/docs/en/sandbox-environments#dev-containers)). Список служебных адресов Claude Code приведен в [Network access requirements](https://code.claude.com/docs/en/network-config#network-access-requirements).

## Go further

- [Выбор среды песочницы](https://code.claude.com/docs/en/sandbox-environments): сравнение dev-контейнеров, песочницы Bash, виртуальных машин и облачных сессий.
- [Применение политик организации](https://code.claude.com/docs/en/devcontainer#enforce-organization-policy): управляемые настройки внутри образа и причины, по которым файл в репозитории не является строгой границей безопасности.
- [Обзор каталога .claude](https://code.claude.com/docs/en/claude-directory): структура данных в `~/.claude`, включая токены авторизации, конфигурацию и историю сессий.

---

<sub>Sources: [Development containers](https://code.claude.com/docs/en/devcontainer) · [Choose a sandbox environment](https://code.claude.com/docs/en/sandbox-environments) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [Authentication](https://code.claude.com/docs/en/authentication) · [Advanced setup](https://code.claude.com/docs/en/setup) · [CLI reference](https://code.claude.com/docs/en/cli-reference) · [Enterprise network configuration](https://code.claude.com/docs/en/network-config) · Claude Code: [reference dev container at d945a61](https://github.com/anthropics/claude-code/tree/d945a61bc6346abce607252d5667df8a3bf0461a/.devcontainer) · GitHub: [Creating a codespace for a repository](https://docs.github.com/en/codespaces/developing-in-a-codespace/creating-a-codespace-for-a-repository), [Setting your default editor for GitHub Codespaces](https://docs.github.com/en/codespaces/setting-your-user-preferences/setting-your-default-editor-for-github-codespaces), [Default environment variables for your codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/default-environment-variables-for-your-codespace), [Rebuilding the container in a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/rebuilding-the-container-in-a-codespace), [Stopping and starting a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/stopping-and-starting-a-codespace), [Deleting a codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/deleting-a-codespace), [GitHub Codespaces billing](https://docs.github.com/en/billing/concepts/product-billing/github-codespaces) · VS Code: [Developing inside a Container](https://code.visualstudio.com/docs/devcontainers/containers), [Sharing Git credentials with your container](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials) · [Dev Container metadata reference](https://containers.dev/implementors/json_reference/) · Dev Container images: [Ubuntu](https://github.com/devcontainers/images/tree/main/src/base-ubuntu) · Docker: [Volumes](https://docs.docker.com/engine/storage/volumes/), [docker container rm](https://docs.docker.com/reference/cli/docker/container/rm/), [docker volume rm](https://docs.docker.com/reference/cli/docker/volume/rm/) · Git: [githooks](https://git-scm.com/docs/githooks) · [ipset](https://ipset.netfilter.org/ipset.man.html)</sub>

<sub>Back to the [Advanced index](../README.md) · Topic: [Разрешения и безопасность](../../topics/permissions-and-safety.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-dev-containers)</sub>
