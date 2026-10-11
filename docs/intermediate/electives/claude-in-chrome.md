# Verify UI changes in the browser with Claude in Chrome

<sub>**Intermediate** · Elective · about 15 minutes · Needs: [Connect a tool with MCP](../07-mcp.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого факультатива вы сможете подключить Claude Code к вашему повседневному браузеру и поручать Claude проверку изменённых веб-страниц: просмотр консоли, содержимого страницы и создание скриншотов. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can connect Claude Code to your own browser, and have Claude check a page you changed: its console, what it shows, and a screenshot.</details>

## You need

- Google Chrome или Microsoft Edge с установленным расширением [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) версии 1.0.36 или новее ([Prerequisites](https://code.claude.com/docs/en/chrome#prerequisites)).
- Тарифный план Pro, Max, Team или Enterprise с авторизацией через `/login`. При работе через прямой API-ключ интеграция с Chrome отключена.
- macOS, Linux или нативная Windows: интеграция с Chrome не поддерживается внутри среды WSL.
- Один из ваших рабочих репозиториев с папкой `.practice/` и утилита `python3` для локального запуска тестового веб-сервера.
- Расход токенов: минимальный (light).

## The idea

Инструмент [Claude in Chrome](https://code.claude.com/docs/en/chrome) связывает Claude Code с браузером, который вы используете каждый день, через официальное расширение. Claude открывает собственные вкладки в видимом окне, позволяя наблюдать за его действиями и при необходимости перехватывать управление для прохождения капчи или двухфакторной аутентификации. <details style="display:inline"><summary><code>[en]</code></summary>Claude in Chrome connects Claude Code to the browser you use every day, through its extension.</details>

В отличие от сервера Chrome DevTools MCP из [урока 7](../07-mcp.md), который запускался в чистом временном профиле, Claude in Chrome разделяет сессии авторизации вашего браузера: Claude имеет доступ ко всем сайтам, где вы уже выполнили вход. Настройки разрешений расширения строго определяют, какие сайты Claude разрешено просматривать, кликать и заполнять.

> [!WARNING]
> Claude действует от имени ваших активных сессий в браузере. Ограничивайте доступ только теми сайтами, которые нужны для текущей задачи, и внимательно читайте каждый диалог вида `Claude in Chrome wants to` перед подтверждением.

## Worked example

1. Создайте страницу с преднамеренной ошибкой и запустите локальный сервер, доступный только с текущего компьютера:

   ```bash
   cat > .practice/ui-demo.html <<'EOF'
   <!doctype html>
   <title>Linkcheck status</title>
   <h1>Linkcheck status</h1>
   <p id="status">Loading...</p>
   <script>document.getElementById('stauts').textContent = 'Ready';</script>
   EOF
   python3 -m http.server 8000 --bind 127.0.0.1 --directory .practice
   ```

2. Во втором терминале запустите Claude Code с флагом поддержки Chrome: `claude --chrome`. При первом старте появится информационный диалог с описанием интеграции; нажмите `Enter`.
3. Выполните команду `/chrome`. Когда в выводе отображаются `Status: Enabled` и `Extension: Installed`, интеграция готова к работе. Если статус иной, закройте Claude Code, перезапустите браузер Chrome и выполните `claude --chrome` повторно: при первом запуске устанавливается файл манифеста нативной связи, считываемый Chrome только в момент старта ([Extension not detected](https://code.claude.com/docs/en/chrome#extension-not-detected)).
4. Отправьте запрос:

   ```text
   Open http://127.0.0.1:8000/ui-demo.html and tell me what the page shows. Read its console for errors, reloading the page if you need to catch errors from page load, and save a screenshot of it to disk.
   ```

   Подтвердите запросы, начинающиеся с `Claude in Chrome wants to`; для данного локального адреса можно разрешить действия на всю сессию. Claude откроет вкладку, сообщит, что на странице остаётся текст `Loading...`, выведет сообщение об ошибке опечатки из консоли и укажет путь к сохранённому скриншоту.

## Your turn

Попросите Claude исправить опечатку в файле `.practice/ui-demo.html`, перезагрузить страницу в Chrome и убедиться, что статус изменился на `Ready`, а консоль чиста.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте пункты самостоятельно.

- [ ] Команда `/chrome` подтверждает активный статус и обнаруженное расширение.
- [ ] Claude открыл тестовую вкладку и выявил ошибку в консоли.
- [ ] Self-check (not tested): вы понимаете разницу в изоляции между Claude in Chrome и временным профилем Chrome DevTools MCP.

## Go further

- [Use Claude Code with Chrome](https://code.claude.com/docs/en/chrome): полный перечень возможностей, управление разрешениями сайтов и безопасность.

---

<sub>Sources: [Use Claude Code with Chrome](https://code.claude.com/docs/en/chrome) · [Connect to MCP servers](https://code.claude.com/docs/en/mcp-quickstart)</sub>

<sub>Back to the [Intermediate index](../README.md) · Topic: [MCP](../../topics/mcp.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-chrome)</sub>
