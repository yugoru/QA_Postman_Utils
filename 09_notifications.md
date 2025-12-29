# 09 — Уведомления (Slack / Telegram webhooks)

Все скрипты ниже вставляются в **Tests** и отправляют уведомления при определённых условиях.

**Как использовать**: 
- Установите webhook URL или токен/chat_id в Globals
- Вставьте скрипт в **Tests** вашего запроса
- Уведомление отправится при выполнении условий (например, ошибка или статус >= 400)

**ВАЖНО**: Не хардкодите секреты в скриптах! Используйте только переменные из Globals.

---

## Переменные (Globals)

### Входные (обязательно для отправки)
- `slack_webhook_url` — URL webhook для Slack
- `tg_bot_token` — токен бота Telegram
- `tg_chat_id` — ID чата Telegram

### Выходные
- `notify_slack_ok` — результат отправки в Slack (true/false)
- `notify_tg_ok` — результат отправки в Telegram (true/false)

---

## 1) Отправка уведомления в Slack

**Что делает**: Отправляет текстовое сообщение в Slack через webhook.

**Входные переменные**:
- `slack_webhook_url` — URL webhook Slack (обязательно)
- `notify_text` — текст сообщения (опционально, по умолчанию используется информация о запросе)

**Выходная переменная**: `notify_slack_ok` (true/false)

**Пример использования**: Отправка при ошибке или статусе >= 400

```javascript
const webhookUrl = pm.globals.get("slack_webhook_url");

if (!webhookUrl) {
  console.warn("⚠️ slack_webhook_url не установлен в Globals");
  pm.globals.set("notify_slack_ok", false);
} else {
  const customText = pm.globals.get("notify_text");
  const status = pm.response.code;
  const method = pm.request.method;
  const url = pm.request.url.toString();
  
  const text = customText || `Запрос ${method} ${url}\nСтатус: ${status} ${pm.response.status}`;
  
  const payload = {
    text: text,
    username: "Postman Bot",
    icon_emoji: ":postman:"
  };
  
  pm.sendRequest({
    url: webhookUrl,
    method: "POST",
    header: {
      "Content-Type": "application/json"
    },
    body: {
      mode: "raw",
      raw: JSON.stringify(payload)
    }
  }, (err, res) => {
    if (err) {
      console.error("❌ Ошибка отправки в Slack:", err);
      pm.globals.set("notify_slack_ok", false);
    } else {
      console.log("✅ Уведомление отправлено в Slack");
      pm.globals.set("notify_slack_ok", true);
    }
  });
}
```

---

## 2) Отправка уведомления в Telegram

**Что делает**: Отправляет текстовое сообщение в Telegram через Bot API.

**Входные переменные**:
- `tg_bot_token` — токен бота Telegram (обязательно)
- `tg_chat_id` — ID чата Telegram (обязательно)
- `notify_text` — текст сообщения (опционально, по умолчанию используется информация о запросе)

**Выходная переменная**: `notify_tg_ok` (true/false)

**Пример использования**: Отправка при ошибке или статусе >= 400

```javascript
const botToken = pm.globals.get("tg_bot_token");
const chatId = pm.globals.get("tg_chat_id");

if (!botToken || !chatId) {
  console.warn("⚠️ tg_bot_token или tg_chat_id не установлены в Globals");
  pm.globals.set("notify_tg_ok", false);
} else {
  const customText = pm.globals.get("notify_text");
  const status = pm.response.code;
  const method = pm.request.method;
  const url = pm.request.url.toString();
  
  const text = customText || `Запрос ${method} ${url}\nСтатус: ${status} ${pm.response.status}`;
  
  const apiUrl = `https://api.telegram.org/bot${botToken}/sendMessage`;
  
  const payload = {
    chat_id: chatId,
    text: text,
    parse_mode: "HTML"
  };
  
  pm.sendRequest({
    url: apiUrl,
    method: "POST",
    header: {
      "Content-Type": "application/json"
    },
    body: {
      mode: "raw",
      raw: JSON.stringify(payload)
    }
  }, (err, res) => {
    if (err) {
      console.error("❌ Ошибка отправки в Telegram:", err);
      pm.globals.set("notify_tg_ok", false);
    } else {
      const response = res.json();
      if (response.ok) {
        console.log("✅ Уведомление отправлено в Telegram");
        pm.globals.set("notify_tg_ok", true);
      } else {
        console.error("❌ Ошибка Telegram API:", response.description);
        pm.globals.set("notify_tg_ok", false);
      }
    }
  });
}
```

---

## 3) Пример: отправка при ошибке или статусе >= 400

**Что делает**: Отправляет уведомление в Slack и/или Telegram, если тест упал или статус >= 400.

**Где использовать**: **Tests** (в конце, после всех тестов)

```javascript
// Проверяем условия для отправки уведомления
const shouldNotify = pm.response.code >= 400 || pm.test.failed;

if (shouldNotify) {
  const status = pm.response.code;
  const method = pm.request.method;
  const url = pm.request.url.toString();
  const statusText = pm.response.status;
  
  // Формируем текст уведомления
  let notifyText = `⚠️ Проблема с запросом\n\n`;
  notifyText += `Метод: ${method}\n`;
  notifyText += `URL: ${url}\n`;
  notifyText += `Статус: ${status} ${statusText}\n`;
  
  if (pm.test.failed) {
    notifyText += `\nТесты: ❌ Упали`;
  }
  
  // Сохраняем текст в Globals для использования в скриптах уведомлений
  pm.globals.set("notify_text", notifyText);
  
  // Отправляем в Slack (если настроен)
  const slackUrl = pm.globals.get("slack_webhook_url");
  if (slackUrl) {
    // Вставьте сюда скрипт "1) Отправка уведомления в Slack"
  }
  
  // Отправляем в Telegram (если настроен)
  const tgToken = pm.globals.get("tg_bot_token");
  const tgChatId = pm.globals.get("tg_chat_id");
  if (tgToken && tgChatId) {
    // Вставьте сюда скрипт "2) Отправка уведомления в Telegram"
  }
}
```

---

## Быстрый пример

```javascript
// Установите в Globals:
// pm.globals.set("slack_webhook_url", "https://hooks.slack.com/services/...");
// pm.globals.set("tg_bot_token", "123456:ABC...");
// pm.globals.set("tg_chat_id", "123456789");

// В Tests вставьте скрипт "3) Пример: отправка при ошибке"

// Результат: уведомление отправится при статусе >= 400 или падении теста
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
