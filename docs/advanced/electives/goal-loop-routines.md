# `/goal`, `/loop` and routines

<sub>**Advanced** · Elective · about 20 minutes, plus run time · Needs: [Put bounds on autonomous runs](../07-bounded-runs.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-09</sub>

> [!NOTE]
> «Рутины находятся в статусе research preview. Поведение, лимиты и API могут меняться.» ([Automate work with routines](https://code.claude.com/docs/en/routines))

## Goal

К концу этого факультатива вы сможете поддерживать автономную работу Claude без постоянных подсказок на каждом шаге: движение к целевому состоянию с `/goal`, периодическое повторение с `/loop` и выполнение по расписанию в облаке с помощью рутин.

## You need

- Ваша копия учебного шаблона с Node.js LTS для раздела Worked example и один из ваших собственных репозиториев для Your turn.
- Для работы с рутиной (только шаги 7-9): тарифный план Pro, Max, Team или Enterprise с авторизацией через `/login` ([Availability by subscription plan](https://code.claude.com/docs/en/feature-availability#availability-by-subscription-plan)). Рутинам требуется подписка Claude, поэтому при использовании API-ключа Console или облачного провайдера пропустите эти шаги ([Feature availability](https://code.claude.com/docs/en/feature-availability#features-that-require-a-claude-subscription)). На планах Team и Enterprise владелец организации (Owner) может отключить рутины для всех ([Automate work with routines](https://code.claude.com/docs/en/routines)).
- Также для рутины: доступ облачных сессий к вашему GitHub-репозиторию через приложение Claude GitHub App или команду `/web-setup`; для приватной копии требуется установленное приложение ([GitHub authentication options](https://code.claude.com/docs/en/claude-code-on-the-web#github-authentication-options)). На планах Team и Enterprise в обоих случаях требуется первоначальная настройка владельцем: коннектор GitHub для авторизации в браузере или быстрая настройка Quick setup для `/web-setup` ([Connect GitHub](https://code.claude.com/docs/en/web-quickstart#connect-github), [Quick setup for Team and Enterprise](https://code.claude.com/docs/en/claude-code-on-the-web#quick-setup-for-team-and-enterprise)).
- Расход лимитов: умеренный. Цель (`/goal`) выполняет несколько шагов, каждая итерация цикла (`/loop`) повторно передает полный контекст сессии ([Why usage climbs in a long session](https://code.claude.com/docs/en/costs#why-usage-climbs-in-a-long-session)), а запуск рутины представляет собой полноценную облачную сессию по вашему тарифу ([Usage and limits](https://code.claude.com/docs/en/routines#usage-and-limits)). Шаги урока минимизируют затраты: модель Sonnet, ограничение числа шагов, короткий интервал цикла и однократный запуск рутины. Экономный вариант: пропустите рутину (шаги 7-9).

## The idea

Команды `/goal`, `/loop` и рутины запускают следующие действия Claude без вашего прямого участия ([Compare ways to keep a session running](https://code.claude.com/docs/en/goal#compare-ways-to-keep-a-session-running), [Compare scheduling options](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options)):

| Механизм | Следующий запуск начинается, когда | Остановка происходит, когда |
|---|---|---|
| `/goal` | Шаг завершен, и вторая модель-оценщик (evaluator) считает, что критерий еще не достигнут | Критерий признан достигнутым или невыполнимым, возникла ошибка или введена команда `/goal clear` |
| `/loop` | Истек заданный интервал времени, пока сессия простаивает | Вы отменили цикл, Claude завершил саморегулируемый цикл или истек тайм-аут |
| Рутина | По расписанию, через вызов API или по событию GitHub в облаке | Вы приостановили или удалили ее; однократная рутина выключается сама |

Модель-оценщик анализирует исключительно историю диалога, поэтому ваше условие должно явно указывать проверку, вывод которой появляется в ответе Claude (например, "`npm test` exits 0"), определять неизменные части системы и завершаться ограничением на число шагов ([Write an effective condition](https://code.claude.com/docs/en/goal#write-an-effective-condition)). Цели и циклы сохраняют текущий режим разрешений: в режиме auto они работают автономно, а в режиме Manual агент сначала спрашивает подтверждение ([Set a goal](https://code.claude.com/docs/en/goal#set-a-goal), [Compare scheduling options](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options)).

> [!WARNING]
> Рутина выполняется как полноценная облачная сессия без пауз на подтверждения (за исключением некоторых операций с артефактами) и действует от вашего имени: ее коммиты и пулреквесты создаются под вашей учетной записью GitHub ([Create a routine](https://code.claude.com/docs/en/routines#create-a-routine)). Она может вызывать любые инструменты подключенных коннекторов (включая запись) без предупреждения, а все коннекторы аккаунта claude.ai подключаются по умолчанию ([Create from the web](https://code.claude.com/docs/en/routines#create-from-the-web), [Connectors](https://code.claude.com/docs/en/routines#connectors)). Отключайте ненужные коннекторы и на этапе обучения ограничивайте запросы рутин операциями чтения.

## Worked example

Работайте в вашей учебной копии на новой ветке.

1. Создайте ветку и папку для логов (уже игнорируемую git в шаблоне), затем запустите Claude Code на модели Sonnet:

   ```bash
   git switch -c keep-working
   mkdir -p .practice
   claude --model sonnet
   ```

   **Windows:** строка `New-Item -ItemType Directory -Force .practice` заменяет команду `mkdir`.

   Строка состояния отображает текущий режим разрешений ([Switch permission modes](https://code.claude.com/docs/en/permission-modes#switch-permission-modes)). Индикатор `⏸ manual mode on` означает подтверждение каждого вызова инструмента; индикатор `⏵⏵ auto mode on` означает автономное выполнение шагов цели и итераций цикла.
2. Задайте цель. Скрипт проверки ссылок считает ссылку вида `setup.md?plain=1` битой даже при наличии файла `setup.md`, так как удаляет только фрагмент после `#`:

   ```text
   /goal npm test exits 0, a new test in test/links.test.js shows that a link such as setup.md?plain=1 to a file that exists is not reported as broken, no existing test changes, and the fix is committed; or stop after 8 turns
   ```

   Claude начинает работу немедленно, а индикатор `◎ /goal active` отображает время работы цели. После каждого шага в протоколе выводится вердикт оценщика; нажмите `Ctrl+O`, чтобы увидеть причины решения ([Set a goal](https://code.claude.com/docs/en/goal#set-a-goal)).
3. Когда оценщик признает условие выполненным, Claude Code снимает цель и записывает отметку о достижении в протокол ([How evaluation works](https://code.claude.com/docs/en/goal#how-evaluation-works)). Вызовите `/goal`: команда покажет достигнутое условие с длительностью, количеством шагов и расходом токенов ([Check status](https://code.claude.com/docs/en/goal#check-status)). Во втором терминале `git log --oneline -1` покажет коммит, а `npm test` успешно завершится. Если `npm test` завершается ошибкой или коммита нет, цель остановилась до завершения (например, исчерпав лимит шагов или будучи признанной невыполнимой). Команда `/goal` покажет статус активности: попросите Claude довести правку до конца и закоммитить ее.
4. Запустите цикл, логирующий результаты тестов каждую минуту:

   ```text
   /loop 1m run npm test and append one line to .practice/loop-log.txt with the time and the number of passing and failing tests. Don't change any other file.
   ```

   Claude регистрирует расписание и подтверждает интервал с идентификатором задачи ([Run on a fixed interval](https://code.claude.com/docs/en/scheduled-tasks#run-on-a-fixed-interval)). Каждая итерация запускается между вашими репликами, никогда не прерывая генерацию ответа ([How scheduled tasks run](https://code.claude.com/docs/en/scheduled-tasks#how-scheduled-tasks-run)).
5. В редакторе (не через Claude) сломайте один тест: в `test/config.test.js` замените в первом тесте `files: ['.']` на `files: ['docs']`. Подождите, пока в логе появится строка с падением тестов, затем верните файл в исходное состояние: `git restore test/config.test.js`.
6. Попросите Claude `cancel the npm test loop`, затем спросите `what scheduled tasks do I have?`: список должен быть пуст ([Manage scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks#manage-scheduled-tasks)). Команда `cat .practice/loop-log.txt` выводит по строке на запуск, включая хотя бы один неудачный результат (точные формулировки могут различаться).
7. Шаги 7-9 требуют активной подписки; при ее отсутствии перейдите к шагу 10. Запланируйте разовый запуск рутины через 5 минут:

   ```text
   /schedule in 5 minutes, run npm test in this repository and reply with the number of passing and failing tests. Don't edit files, commit or push.
   ```

   Claude задаст уточняющие вопросы о расписании, репозитории и запросе, затем подтвердит точное время перед сохранением ([Create from the CLI](https://code.claude.com/docs/en/routines#create-from-the-cli), [Schedule a one-off run](https://code.claude.com/docs/en/routines#schedule-a-one-off-run)). Укажите, что подключать коннекторы не нужно; если отключить сразу не удалось, удалите их после сохранения через команду `/schedule update` или форму редактирования на `claude.ai/code/routines` ([Connectors](https://code.claude.com/docs/en/routines#connectors)).\
   Если Claude выведет инструкцию по настройке доступа к GitHub, выполните ее до наступления времени запуска; если время прошло, отредактируйте рутину и задайте новое время ([Repositories and branch permissions](https://code.claude.com/docs/en/routines#repositories-and-branch-permissions), [Edit and control routines](https://code.claude.com/docs/en/routines#edit-and-control-routines)). Если Claude просит авторизоваться, рутина не была сохранена; если команда `/schedule` отсутствует в меню, см. раздел [`/schedule` returns "Unknown command"](https://code.claude.com/docs/en/routines#schedule-returns-unknown-command).
8. Команда `/schedule list` отображает созданную рутину ([Manage routines from the CLI](https://code.claude.com/docs/en/routines#manage-routines-from-the-cli)). После ее срабатывания откройте страницу `claude.ai/code/routines`, найдите нужный запуск и прочитайте протокол с количеством пройденных и упавших тестов ([View and interact with runs](https://code.claude.com/docs/en/routines#view-and-interact-with-runs)). Облачная сессия клонировала вашу ветку по умолчанию, поэтому ее счетчики не учитывают тест, добавленный в ветку `keep-working` ([Repositories and branch permissions](https://code.claude.com/docs/en/routines#repositories-and-branch-permissions)). Разовая рутина теперь отмечена статусом **Ran**.
9. Удалите рутину: откройте меню рядом с ее названием и выберите **Delete** ([Edit and control routines](https://code.claude.com/docs/en/routines#edit-and-control-routines)).\
   Если вы подключали GitHub только ради рутины, отключите коннектор на `claude.ai/customize/connectors`, что удалит учетные данные (как из браузера, так и из `/web-setup`); токен `gh` при этом останется активным на GitHub ([Remove the `/web-setup` token](https://code.claude.com/docs/en/web-quickstart#remove-the-web-setup-token)). Если вы устанавливали Claude GitHub App только для этого шага, уберите репозиторий из списка разрешенных или удалите приложение, если оно больше нигде не используется ([Reviewing and modifying installed GitHub Apps](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps)).
10. Команда `git switch -` возвращает рабочую копию на исходную ветку; ветка `keep-working` сохраняет исправление.

## Your turn

В одном из ваших собственных репозиториев выберите рутинную задачу, конечное состояние которой проверяется командой терминала (упавший тест или ошибка линтера). Решите ее с помощью цели (`/goal`). Затем закоммитьте файл `.claude/loop.md` для непрерывного отслеживания: этот файл заменяет встроенный запрос команды `/loop`, запущенной без явного аргумента-подсказки ([Customize the default prompt with loop.md](https://code.claude.com/docs/en/scheduled-tasks#customize-the-default-prompt-with-loop-md)). При наличии подписки настройте разовую рутину для запуска этой проверки в облаке после попадания изменений в основную ветку или явно укажите имя ветки в запросе рутины ([Repositories and branch permissions](https://code.claude.com/docs/en/routines#repositories-and-branch-permissions)).

Задание выполнено, когда:

- Условие цели содержит команду проверки конечного состояния, как минимум одно ограничение на неизменность частей системы и лимит шагов.
- Изменения закоммичены, а проверочная команда завершается с кодом 0.
- Файл `.claude/loop.md` закоммичен. В нем описано, что именно проверять и как вывести результат одной строкой без модификации файлов. Команда `/loop 10m` (интервал без дополнительного текста) выполнила его не менее двух раз до отмены ([Run the built-in maintenance prompt](https://code.claude.com/docs/en/scheduled-tasks#run-the-built-in-maintenance-prompt)).
- Никаких активных фоновых задач или рутин не осталось: вопрос `what scheduled tasks do I have?` возвращает пустой список, а созданные рутины удалены.

## Check

Задание выполнено, когда выполняются все условия:

- [ ] В вашем репозитории результаты работы цели закоммичены (`git log -1` показывает коммит), а указанная в условии команда завершается с кодом 0.
- [ ] В учебной копии файл `.practice/loop-log.txt` содержит строки минимум от двух запусков цикла, включая хотя бы одну строку с ошибкой.
- [ ] Файл `.claude/loop.md` закоммичен в репозитории (`git ls-files .claude/loop.md` находит его), а `git status --porcelain` ничего не выводит.
- [ ] Самопроверка (не проверяется скриптом): условие содержало команду, неизменные ограничения и лимит шагов; `/loop 10m` выполнил `loop.md` как минимум дважды до отмены; `what scheduled tasks do I have?` сообщает об отсутствии задач; а если вы создавали рутину, вы проверили отчет о ее выполнении и удалили ее.

Если проверочная команда падает, цель завершилась раньше времени (по лимиту шагов или признанию невыполнимой): нажмите `Ctrl+O` для чтения причин последнего вердикта ([How evaluation works](https://code.claude.com/docs/en/goal#how-evaluation-works)), устраните причину или сформулируйте новую цель. Если `/goal` не запускается, проверьте вывод (чаще всего это недоверенная папка или параметр `disableAllHooks`, см. [Requirements](https://code.claude.com/docs/en/goal#requirements)). Если в логе нет строк с ошибками, цикл не успел сработать во время поломки теста: запустите цикл снова по шагу 4, сломайте тест по шагу 5, дождитесь строки с падением, восстановите файл и остановите цикл по шагу 6. Если `git ls-files` ничего не выводит, выполните `git check-ignore -v .claude/loop.md` для поиска блокирующего правила игнорирования, добавьте файл через `git add -f` и сделайте коммит.

## Watch out

- Цикл срабатывает даже во время вашего отсутствия, отправляя полный контекст сессии на каждом шаге, и продолжается до отмены или истечения срока ([Stop a loop](https://code.claude.com/docs/en/scheduled-tasks#stop-a-loop)). На планах Pro, Max, Team и Enterprise команда `/usage` показывает самые ресурсоемкие циклы ([Plan usage breakdown](https://code.claude.com/docs/en/costs#plan-usage-breakdown)).
- Оценщик не может подтвердить размытые условия, не отраженные в выводе инструментов (например, «код стал чище»). Всегда указывайте конкретную команду и ожидаемый результат ([How evaluation works](https://code.claude.com/docs/en/goal#how-evaluation-works)).
- Зеленый статус в списке рутин означает лишь то, что облачная сессия стартовала и завершилась без инфраструктурного сбоя. Обязательно открывайте протокол, чтобы проверить содержательный результат работы ([View and interact with runs](https://code.claude.com/docs/en/routines#view-and-interact-with-runs)).

## Go further

- [Инженерия циклов: начало работы с циклами](https://claude.com/resources/articles/getting-started-with-loops): статья команды Anthropic Claude Code о пошаговых, целевых, временных и проактивных циклах.
- [Хуки на основе промптов](https://code.claude.com/docs/en/hooks-guide#prompt-based-hooks): разновидность хука Stop, которую обертывает команда `/goal` ([How evaluation works](https://code.claude.com/docs/en/goal#how-evaluation-works)), для проверок в каждой сессии.
- [Расписание повторяющихся задач в Claude Code Desktop](https://code.claude.com/docs/en/desktop-scheduled-tasks): запуск задач по расписанию локально на вашем компьютере при открытом приложении Desktop.

---

<sub>Sources: [Keep Claude working toward a goal](https://code.claude.com/docs/en/goal) · [Run prompts on a schedule](https://code.claude.com/docs/en/scheduled-tasks) · [Automate work with routines](https://code.claude.com/docs/en/routines) · [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) · [Manage costs effectively](https://code.claude.com/docs/en/costs) · [Feature availability](https://code.claude.com/docs/en/feature-availability) · [Use Claude Code in the cloud](https://code.claude.com/docs/en/claude-code-on-the-web) · [Get started with Claude Code in the cloud](https://code.claude.com/docs/en/web-quickstart) · [Reviewing and modifying installed GitHub Apps](https://docs.github.com/en/apps/using-github-apps/reviewing-and-modifying-installed-github-apps)</sub>

<sub>Back to the [Advanced index](../README.md) · Topic: [Автоматизация](../../topics/automation.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=a-goal-loop)</sub>
