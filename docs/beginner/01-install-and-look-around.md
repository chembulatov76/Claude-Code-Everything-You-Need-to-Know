# Install, sign in and look around

<sub>**Beginner** · Lesson 1 of 5 · about 20 minutes · Needs: [Beginner index](README.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-05</sub>

## Goal

К концу этого урока вы сможете установить Claude Code, войти в свою учётную запись, запустить инструмент в репозитории и определить, какую версию, модель и аккаунт использует текущая сессия. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this lesson you can install Claude Code, sign in, start it in a repository, and tell which version, model and account a session is using.</details>

## You need

- Терминал и git, а также платная подписка Claude (Pro, Max, Team или Enterprise; бесплатный план не включает Claude Code), аккаунт Claude Console либо доступ через поддерживаемого облачного провайдера ([Quickstart](https://code.claude.com/docs/en/quickstart), [Advanced setup](https://code.claude.com/docs/en/setup)). <details style="display:inline"><summary><code>[en]</code></summary>A terminal and git, plus a Claude subscription (Pro, Max, Team or Enterprise; the free plan doesn't include Claude Code), a Claude Console account, or access through a supported cloud provider</details>
- Ваша клонированная копия учебного шаблона (как описано в [Beginner index](README.md#practice)) и Node.js LTS для запуска скриптов проверки. В GitHub Codespace вашей копии Claude Code уже предустановлен: начинайте сразу с шага 3.
- Чтобы практиковаться в собственном рабочем репозитории, сохраните также копию шаблона: проверка запускается из неё. В своём репозитории создайте папку, куда уроки будут сохранять снимки состояния, и добавьте её в локальные исключения git без создания коммита ([gitignore](https://git-scm.com/docs/gitignore)):

  ```bash
  mkdir -p .practice
  echo '.practice/' >> "$(git rev-parse --git-path info/exclude)"
  ```

  **Windows:** `New-Item -ItemType Directory -Force .practice` и `Add-Content (git rev-parse --git-path info/exclude) '.practice/'`.
- Расход токенов: минимальный (light), пара коротких промптов в рамках одной сессии. Внутри Claude Code команда `/usage` показывает стоимость сессии и лимиты использования тарифного плана ([Commands](https://code.claude.com/docs/en/commands)).

> [!NOTE]
> В нативной среде Windows запускайте каждую команду в PowerShell. Если строка **Windows** отсутствует, команда выполняется как написано; если после команды указана строка **Windows**, используйте её. Установка [Git for Windows](https://git-scm.com/downloads/win) позволяет Claude Code использовать Bash; без него Claude Code обращается к PowerShell ([Quickstart](https://code.claude.com/docs/en/quickstart)).
>
> Если PowerShell блокирует запуск `npm` с сообщением `running scripts is disabled on this system`, вводите `npm.cmd` везде, где в уроке указано `npm` (например, `npm.cmd run check -- b-1`), либо разрешите локальные скрипты для своего аккаунта, как показано в [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install#running-scripts-is-disabled-on-this-system).

## The idea

Claude Code: это консольная программа, запускаемая в терминале внутри папки проекта. Она считывает файлы репозитория и отвечает на вопросы о коде; в следующих уроках инструмент начнёт вносить в них изменения. <details style="display:inline"><summary><code>[en]</code></summary>Claude Code is a program you run in a terminal, inside a project folder. It reads the files there and answers questions about them; in the next lessons it changes them too.</details>

Прежде чем полагаться на ответ, необходимо знать три факта о сессии:

- **версию (the version)**: возможности и настройки по умолчанию меняются от релиза к релизу;
- **модель (the model)**: модели отличаются возможностями и уровнем рассуждений;
- **аккаунт (the account)**: именно тарифный план и аккаунт определяют, какие функции и модели вам доступны.

Информационная шапка, которую Claude Code отображает при старте, указывает версию и модель. Команда `/status` показывает версию, модель и активный аккаунт в едином окне ([Commands](https://code.claude.com/docs/en/commands)).

## Worked example

1. Установите Claude Code ([Quickstart](https://code.claude.com/docs/en/quickstart)):

   ```bash
   curl -fsSL https://claude.ai/install.sh | bash
   ```

   **Windows:** `irm https://claude.ai/install.ps1 | iex`

   После завершения установки откройте новое окно терминала.
2. Проверьте корректность установки. При успешной установке выводится номер версии и пояснение `(Claude Code)`:

   ```bash
   claude --version
   ```

   Если оболочка сообщает, что команда `claude` не найдена или не распознана, путь к исполняемому файлу ещё не добавлен в переменную окружения `PATH`: устраните проблему по инструкции [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install#command-not-found-claude-after-installation), прежде чем продолжать.
3. Перейдите в учебную копию репозитория и сохраните этот вывод для проверочного скрипта:

   ```bash
   cd <your practice copy>
   mkdir -p .practice
   claude --version > .practice/version.txt
   ```

   **Windows:** строка `New-Item -ItemType Directory -Force .practice` заменяет вызов `mkdir`.
4. Запустите сессию в этой же папке:

   ```bash
   claude
   ```

   При первом запуске Claude Code предложит авторизоваться. В появившемся запросе терминала выберите тип аккаунта; при наличии подписки Claude или аккаунта Console процесс продолжится в браузере. Если браузер показал одноразовый код вместо автоматического возврата в терминал, вставьте его по приглашению `Paste code here if prompted` ([Authentication](https://code.claude.com/docs/en/authentication)). При авторизации по API-ключу или через облачного провайдера следуйте инструкции [Quickstart](https://code.claude.com/docs/en/quickstart).

   В репозитории, которому вы ещё не выдали разрешение, Claude Code покажет диалог доверия рабочей области (workspace trust dialog), уточняя, доверяете ли вы содержимому папки ([Security](https://code.claude.com/docs/en/security#additional-safeguards)). По умолчанию подсвечен безопасный вариант **No, exit**, и нажатие Enter приведёт к выходу. Поскольку вы сами клонировали этот репозиторий, нажмите стрелку вниз, чтобы выбрать **Yes, I trust this folder**, и подтвердите нажатием Enter. Claude Code сохранит статус доверия для всего репозитория и больше не будет задавать этот вопрос в данной папке ([Configure permissions](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust)).

   В заголовке над строкой ввода отобразятся версия, модель и путь к рабочей директории.
5. Введите слэш-команду `/status`. На вкладке **Status** будут указаны версия, модель и аккаунт, под которым выполнен вход (строки **Login method** и **Email**). Нажмите клавишу `Esc`, чтобы закрыть окно ([Interactive mode](https://code.claude.com/docs/en/interactive-mode#general-controls)). Ваша версия может быть новее указанной в шапке руководства: это руководство выверяется по стабильному каналу `stable`, а Claude Code по умолчанию обновляется по каналу `latest` ([Configure the release channel](https://code.claude.com/docs/en/setup#configure-release-channel)).
6. Задайте два вопроса о кодовой базе:

   ```text
   what does this project do?
   ```

   ```text
   which file parses linkcheck.json, and how do I run the tests?
   ```

   Claude прочитает файлы проекта, чтобы подготовить ответ. Ваши точные формулировки могут отличаться, но в ответах должны быть названы `src/config.js` и команда `npm test`.
7. Завершите сессию командой `/exit`.

## Your turn

Повторите эти шаги в своей учебной копии (или в собственном репозитории), задав собственные вопросы к коду:

1. В целевой папке сохраните вывод `claude --version` в файл `.practice/version.txt`.
2. Запустите `claude`. В вашем собственном репозитории снова появится диалог доверия: выбирайте вариант **Yes, I trust this folder** только для проверенного кода.
3. Откройте окно `/status` и обратите внимание на активную модель и аккаунт.
4. Задайте два собственных вопроса о структуре проекта, затем закройте сессию командой `/exit`.

## Check

Задание считается выполненным, когда соблюдены все условия:

- [ ] Файл `.practice/version.txt` содержит строку вида `X.Y.Z (Claude Code)`. Проверьте командой `cat .practice/version.txt`.
- [ ] Self-check (not tested): вы можете чётко назвать модель и аккаунт, которые показал `/status`.

Запустите команду `npm run check -- b-1` в учебной копии репозитория. При практике в собственном репозитории запускайте команду с указанием директории: `npm run check -- b-1 --dir <your repo>`. Ошибка npm вида `Could not read package.json` или `Missing script: "check"` означает, что команда вызвана вне учебной копии: перейдите в папку шаблона через `cd` и повторите запуск.

Если первый пункт не проходит проверку, вызовите команду `claude --version` отдельно: если она выводит ошибку, исправьте её по шагу 2 из Worked example; если версия выводится корректно, сохраните её заново в папку, которую проверяет скрипт (учебный шаблон или директория, переданная в `--dir`).

## Watch out

- Новая сессия по умолчанию способна редактировать большинство файлов и запускать команды без предварительного запроса подтверждения ([Quickstart](https://code.claude.com/docs/en/quickstart)). В этом вводном уроке задавайте только вопросы; в следующем уроке будет показано, как тонко управлять правами Claude. Даже обычный вопрос может побудить Claude выполнить команду с записью на диск (например, запуск тестов для проверки ответа): выполните `git status` после завершения сессии и до перехода к уроку 2 закоммитьте или удалите любые появившиеся несохранённые файлы.

## Go further

- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works): архитектура взаимодействия между вашим вопросом и ответом инструмента.
- [Configure the release channel](https://code.claude.com/docs/en/setup#configure-release-channel): переключение на канал `stable`, защищающий от регрессий свежих версий.
- [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install): способы устранения ошибок установки и аутентификации.

---

<sub>Sources: [Quickstart](https://code.claude.com/docs/en/quickstart) · [Advanced setup](https://code.claude.com/docs/en/setup) · [Authentication](https://code.claude.com/docs/en/authentication) · [Security](https://code.claude.com/docs/en/security) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Commands](https://code.claude.com/docs/en/commands) · [Interactive mode](https://code.claude.com/docs/en/interactive-mode) · [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install) · [gitignore](https://git-scm.com/docs/gitignore)</sub>

<sub>← [Beginner index](README.md) · [Permission modes and plan mode](02-permission-modes-and-plan-mode.md) → · Topic: [Models, effort and cost](../topics/models-effort-and-cost.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=b-1)</sub>
