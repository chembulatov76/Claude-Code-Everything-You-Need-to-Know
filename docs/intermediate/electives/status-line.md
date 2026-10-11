# Customize the status line

<sub>**Intermediate** · Elective · about 15 minutes · Needs: [Pick the model and effort](../03-model-and-effort.md) · Verified against Claude Code v2.1.285 (stable) on 2026-10-06</sub>

## Goal

К концу этого факультатива вы сможете настроить собственную строку состояния в нижней части интерфейса Claude Code с помощью скрипта, отображающего модель, текущую директорию и процент заполнения контекстного окна. <details style="display:inline"><summary><code>[en]</code></summary>By the end of this Elective you can give Claude Code a status line of your own, from a script that shows the model, the folder and how full the context window is.</details>

## You need

- Пакет `jq`, а также macOS или Linux (скрипт написан на bash). В Windows документация [Windows configuration](https://code.claude.com/docs/en/statusline#windows-configuration) предлагает версии для PowerShell и Git Bash.
- Расход токенов: отсутствует. Скрипт строки состояния выполняется локально на вашем компьютере и не расходует API-токены.

## The idea

[Строка состояния (status line)](https://code.claude.com/docs/en/statusline) - это информационная панель в нижней части экрана Claude Code, отображающая то, что выводит ваш локальный скрипт. Claude Code передаёт скрипту через stdin JSON с метаданными о текущей сессии (название модели, процент использованного контекста и др.). Скрипт выполняется в момент старта сессии и обновляется при ключевых событиях, например получении нового сообщения от Claude. <details style="display:inline"><summary><code>[en]</code></summary>The status line is a bar at the bottom of Claude Code that shows whatever your script prints.</details>

Настроить строку состояния можно двумя путями: команда `/statusline` с описанием желаемого формата поручает Claude Code сгенерировать скрипт и настройки за вас; либо вы создаёте скрипт вручную и регистрируете ключ `statusLine` в настройках, что мы и сделаем в этом факультативе для понимания внутреннего устройства.

## Worked example

1. Сохраните скрипт как `~/.claude/statusline.sh` и сделайте его исполняемым командой `chmod +x ~/.claude/statusline.sh`:

   ```bash
   #!/bin/bash
   input=$(cat)
   MODEL=$(echo "$input" | jq -r '.model.display_name')
   DIR=$(echo "$input" | jq -r '.workspace.current_dir')
   PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
   echo "[$MODEL] ${DIR##*/} | ${PCT}% context"
   ```

2. Протестируйте скрипт без Claude Code, передав тестовый JSON:

   ```bash
   echo '{"model":{"display_name":"Opus"},"workspace":{"current_dir":"/home/me/linkcheck"},"context_window":{"used_percentage":12.5}}' | ~/.claude/statusline.sh
   ```

   Скрипт выведет: `[Opus] linkcheck | 12% context`.
3. Добавьте настройку в ваш персональный файл конфигурации `~/.claude/settings.json`:

   ```json
   "statusLine": {
     "type": "command",
     "command": "~/.claude/statusline.sh"
   }
   ```

4. Запустите `claude` и отправьте любой запрос. Внизу отобразится ваша строка состояния, и процент заполнения будет динамически расти по мере диалога. Claude Code выполняет скрипт только в каталогах, где было подтверждено доверие.

## Your turn

Дополните скрипт выводом имени текущей ветки git с помощью команды `git branch --show-current 2>/dev/null` и проверьте отображение при смене директорий.

## Check

В этом факультативе нет скрипта `npm run check`: отметьте пункты самостоятельно.

- [ ] Скрипт `~/.claude/statusline.sh` исполняем и корректно форматирует тестовый JSON.
- [ ] В сессии Claude Code строка состояния отображает актуальные параметры сессии.
- [ ] Self-check (not tested): вы знаете, какие события вызывают повторное выполнение скрипта строки состояния.

## Go further

- [Customize your status line](https://code.claude.com/docs/en/statusline): формат входного JSON, управление цветами вывода и обработка ошибок.

---

<sub>Sources: [Customize your status line](https://code.claude.com/docs/en/statusline) · [Settings](https://code.claude.com/docs/en/settings)</sub>

<sub>Back to the [Intermediate index](../README.md) · Topic: [Models, effort and cost](../../topics/models-effort-and-cost.md) · [Stuck on this lesson?](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know/issues/new?template=lesson-feedback.yml&lesson=i-statusline)</sub>
