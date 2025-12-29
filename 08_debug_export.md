# 08 — Отладка и экспорт (cURL / дампы response)

Все скрипты ниже вставляются в **Pre-request Script** или **Tests** и выводят информацию в console.log или сохраняют в Globals.

**Как использовать**: 
- Для cURL — вставьте в **Pre-request Script**
- Для дампов response — вставьте в **Tests**

---

## Переменные (Globals)

### Выходные
- `dbg_curl` — команда cURL текущего запроса
- `dbg_last_status` — последний HTTP статус
- `dbg_last_response_snippet` — фрагмент последнего response (первые 500 символов)

---

## 1) Вывод cURL текущего запроса

**Что делает**: Формирует и выводит команду cURL для текущего запроса в console.log и сохраняет в Globals.

**Где использовать**: **Pre-request Script**

**Выходная переменная**: `dbg_curl` (строка с командой cURL)

```javascript
function buildCurlCommand(request) {
  const method = request.method || "GET";
  const url = request.url.toString();
  
  let curl = `curl -X ${method}`;
  
  // Headers
  if (request.headers && request.headers.count() > 0) {
    request.headers.each((header) => {
      curl += ` \\\n  -H "${header.key}: ${header.value}"`;
    });
  }
  
  // Body
  if (request.body && request.body.raw) {
    const body = request.body.raw;
    curl += ` \\\n  -d '${body.replace(/'/g, "'\\''")}'`;
  } else if (request.body && request.body.urlencoded) {
    const params = [];
    request.body.urlencoded.each((param) => {
      params.push(`${encodeURIComponent(param.key)}=${encodeURIComponent(param.value)}`);
    });
    if (params.length > 0) {
      curl += ` \\\n  -d '${params.join("&")}'`;
    }
  } else if (request.body && request.body.formdata) {
    const params = [];
    request.body.formdata.each((param) => {
      if (param.type === "file") {
        params.push(`${param.key}=@${param.src}`);
      } else {
        params.push(`${param.key}=${param.value}`);
      }
    });
    if (params.length > 0) {
      curl += ` \\\n  -F '${params.join("&")}'`;
    }
  }
  
  curl += ` \\\n  "${url}"`;
  
  return curl;
}

const curlCommand = buildCurlCommand(pm.request);

console.log("=== cURL команда ===");
console.log(curlCommand);
console.log("===================");

pm.globals.set("dbg_curl", curlCommand);
```

---

## 2) Дамп response: status, headers, body

**Что делает**: Выводит в console.log информацию о response (статус, заголовки, фрагмент body) и сохраняет в Globals.

**Где использовать**: **Tests**

**Выходные переменные**:
- `dbg_last_status` — HTTP статус
- `dbg_last_response_snippet` — фрагмент body (первые 500 символов)

```javascript
const status = pm.response.code;
const statusText = pm.response.status;
const headers = pm.response.headers.toObject();
const body = pm.response.text();

// Ограничиваем длину body для вывода
const maxLength = 500;
const bodySnippet = body.length > maxLength 
  ? body.substring(0, maxLength) + "... (обрезано)"
  : body;

console.log("=== Response Info ===");
console.log("Status:", status, statusText);
console.log("\nHeaders:");
Object.keys(headers).forEach(key => {
  console.log(`  ${key}: ${headers[key]}`);
});
console.log("\nBody (первые", maxLength, "символов):");
console.log(bodySnippet);
console.log("===================");

pm.globals.set("dbg_last_status", status);
pm.globals.set("dbg_last_response_snippet", bodySnippet);
```

---

## 3) Полный дамп request + response

**Что делает**: Выводит полную информацию о request и response в структурированном виде.

**Где использовать**: **Tests**

**Выходные переменные**: не сохраняет, только выводит в console

```javascript
console.log("=== REQUEST ===");
console.log("Method:", pm.request.method);
console.log("URL:", pm.request.url.toString());
console.log("Headers:", JSON.stringify(pm.request.headers.toObject(), null, 2));

if (pm.request.body && pm.request.body.raw) {
  console.log("Body:", pm.request.body.raw);
} else if (pm.request.body && pm.request.body.urlencoded) {
  const params = {};
  pm.request.body.urlencoded.each((param) => {
    params[param.key] = param.value;
  });
  console.log("Body (urlencoded):", JSON.stringify(params, null, 2));
}

console.log("\n=== RESPONSE ===");
console.log("Status:", pm.response.code, pm.response.status);
console.log("Response Time:", pm.response.responseTime, "ms");
console.log("Headers:", JSON.stringify(pm.response.headers.toObject(), null, 2));

try {
  const jsonBody = pm.response.json();
  console.log("Body (JSON):", JSON.stringify(jsonBody, null, 2));
} catch (e) {
  const textBody = pm.response.text();
  const snippet = textBody.length > 1000 
    ? textBody.substring(0, 1000) + "... (обрезано)"
    : textBody;
  console.log("Body (text):", snippet);
}

console.log("===================");
```

---

## Быстрый пример

```javascript
// В Pre-request Script: вставьте скрипт "1) Вывод cURL"
// В Tests: вставьте скрипт "2) Дамп response"

// Результат: в console.log будет cURL команда и информация о response
// В Globals: {{dbg_curl}}, {{dbg_last_status}}, {{dbg_last_response_snippet}}
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
