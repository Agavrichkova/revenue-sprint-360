# Revenue Sprint 360° — Telegram Leads
Два способа отправки заявки в Telegram из статического сайта (GitHub Pages).

## A) Быстро (для тестов, но небезопасно)
Файл: **index_quick.html** — отправка прямо в Telegram Bot API.
- Замените:
  - `PASTE_YOUR_BOT_TOKEN_HERE` → ваш Bot Token из @BotFather
  - `PASTE_GROUP_CHAT_ID_HERE` → chat_id группы (пример: `-1001234567890`)
- Минус: токен виден в исходнике страницы.

## B) Надёжно (в прод)
Файл: **index_secure.html** — отправка на Google Apps Script (Web App), который уже дергает Telegram с токеном в секретах.
- В коде замените `PASTE_DEPLOYMENT_ID` на URL вашего скрипта.

### Как получить chat_id группы (через @RawDataBot)
1. Откройте диалог с **@RawDataBot** и нажмите Start.
2. В вашей группе напишите любое сообщение, например `test`.
3. Перешлите это сообщение **@RawDataBot**.
4. В ответе найдите блок `forward_from_chat` → `id`. Он будет вида **-100...** — это и есть chat_id вашей группы.

Альтернатива: добавьте @RawDataBot в группу и отправьте `/id` — бот пришлет `chat.id`.

### Пример Google Apps Script (secure)
1. Перейдите на https://script.google.com/ → New project.
2. Вставьте код ниже в `Code.gs`:
```
function doPost(e) {
  var data = JSON.parse(e.postData.contents);
  var token = PropertiesService.getScriptProperties().getProperty('BOT_TOKEN');
  var chatId = PropertiesService.getScriptProperties().getProperty('CHAT_ID');
  var text = '📝 Новая заявка на Revenue Sprint 360°\n' +
            'Имя: ' + (data.name||'') + '\n' +
            'Email: ' + (data.email||'') + '\n' +
            'Telegram: ' + (data.telegram||'') + '\n' +
            'Комментарий: ' + (data.message||'—');
  var url = 'https://api.telegram.org/bot' + token + '/sendMessage';
  var payload = {chat_id: chatId, text: text};
  var params = {method: 'post', contentType: 'application/json', payload: JSON.stringify(payload)};
  UrlFetchApp.fetch(url, params);
  return ContentService.createTextOutput('OK').setMimeType(ContentService.MimeType.TEXT);
}
```
3. **Project Settings → Script properties** добавьте:
   - `BOT_TOKEN` = ваш токен
   - `CHAT_ID` = `-100...`
4. **Deploy → New deployment → Web app**:
   - Execute as: Me
   - Who has access: Anyone
   - Скопируйте URL и вставьте в `index_secure.html` вместо `PASTE_DEPLOYMENT_ID`.

Собрано: 2025-10-07 19:50:01
