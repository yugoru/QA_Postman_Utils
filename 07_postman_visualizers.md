# 07 — Postman Visualizers (таблицы / base64 / pretty JSON)

Все скрипты ниже вставляются в **Tests** и создают визуализацию ответа.

**Как использовать**: Вставьте скрипт в раздел **Tests** вашего запроса. Визуализация появится на вкладке **Visualize**.

---

## Переменные (Globals)

### Входные (необязательно)
- `vis_b64_field` — имя поля с base64 изображением, по умолчанию `"data"`

---

## 1) Visualizer: таблица из массива объектов

**Что делает**: Создаёт HTML-таблицу из массива объектов в JSON-ответе.

**Требования**: Response должен быть JSON с массивом объектов.

**Входные переменные**: не требуются

```javascript
const template = `
<style>
  table { border-collapse: collapse; width: 100%; margin: 20px 0; }
  th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
  th { background-color: #4CAF50; color: white; }
  tr:nth-child(even) { background-color: #f2f2f2; }
  tr:hover { background-color: #f5f5f5; }
</style>
<table>
  <thead>
    <tr>
      {{#each headers}}
      <th>{{this}}</th>
      {{/each}}
    </tr>
  </thead>
  <tbody>
    {{#each rows}}
    <tr>
      {{#each this}}
      <td>{{this}}</td>
      {{/each}}
    </tr>
    {{/each}}
  </tbody>
</table>
`;

try {
  const jsonData = pm.response.json();
  
  if (!Array.isArray(jsonData) || jsonData.length === 0) {
    throw new Error("Response должен быть непустым массивом объектов");
  }
  
  // Получаем все уникальные ключи из объектов
  const allKeys = new Set();
  jsonData.forEach(obj => {
    Object.keys(obj).forEach(key => allKeys.add(key));
  });
  const headers = Array.from(allKeys);
  
  // Формируем строки таблицы
  const rows = jsonData.map(obj => {
    return headers.map(header => {
      const value = obj[header];
      if (value === null || value === undefined) return "";
      if (typeof value === "object") return JSON.stringify(value);
      return String(value);
    });
  });
  
  const data = { headers, rows };
  
  pm.visualizer.set(template, data);
  console.log("✅ Таблица создана:", rows.length, "строк");
} catch (error) {
  console.error("❌ Ошибка создания таблицы:", error.message);
}
```

---

## 2) Visualizer: base64 изображение

**Что делает**: Отображает изображение из base64 строки в JSON-ответе.

**Требования**: Response должен быть JSON с полем, содержащим base64 строку.

**Входные переменные**:
- `vis_b64_field` (опционально) — имя поля с base64, по умолчанию `"data"`

```javascript
const template = `
<style>
  .image-container { text-align: center; padding: 20px; }
  img { max-width: 100%; height: auto; border: 1px solid #ddd; border-radius: 4px; }
</style>
<div class="image-container">
  <img src="data:image/{{format}};base64,{{data}}" alt="Base64 Image" />
</div>
`;

try {
  const jsonData = pm.response.json();
  const fieldName = pm.globals.get("vis_b64_field") || "data";
  
  const base64Data = jsonData[fieldName];
  
  if (!base64Data || typeof base64Data !== "string") {
    throw new Error(`Поле "${fieldName}" не найдено или не является строкой`);
  }
  
  // Определяем формат изображения по началу base64 строки
  let format = "png";
  if (base64Data.startsWith("/9j/") || base64Data.startsWith("iVBORw0KGgo")) {
    format = base64Data.startsWith("/9j/") ? "jpeg" : "png";
  } else if (base64Data.startsWith("R0lGOD")) {
    format = "gif";
  } else if (base64Data.startsWith("UklGR")) {
    format = "webp";
  }
  
  const data = {
    data: base64Data,
    format: format
  };
  
  pm.visualizer.set(template, data);
  console.log("✅ Изображение отображено, формат:", format);
} catch (error) {
  console.error("❌ Ошибка отображения изображения:", error.message);
}
```

---

## 3) Visualizer: pretty JSON

**Что делает**: Отображает JSON-ответ в форматированном виде с подсветкой синтаксиса.

**Требования**: Response должен быть валидным JSON.

**Входные переменные**: не требуются

```javascript
const template = `
<style>
  pre { 
    background-color: #f4f4f4; 
    border: 1px solid #ddd; 
    border-radius: 4px; 
    padding: 15px; 
    overflow-x: auto;
    font-family: 'Courier New', monospace;
    font-size: 14px;
    line-height: 1.5;
  }
  .json-key { color: #881391; }
  .json-string { color: #1A1AA6; }
  .json-number { color: #1C00CF; }
  .json-boolean { color: #0B7500; }
  .json-null { color: #808080; }
</style>
<pre>{{{json}}}</pre>
`;

function syntaxHighlight(json) {
  if (typeof json !== "string") {
    json = JSON.stringify(json, null, 2);
  }
  
  json = json
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g, (match) => {
      let cls = "json-number";
      if (/^"/.test(match)) {
        if (/:$/.test(match)) {
          cls = "json-key";
        } else {
          cls = "json-string";
        }
      } else if (/true|false/.test(match)) {
        cls = "json-boolean";
      } else if (/null/.test(match)) {
        cls = "json-null";
      }
      return `<span class="${cls}">${match}</span>`;
    });
  
  return json;
}

try {
  const jsonData = pm.response.json();
  const highlighted = syntaxHighlight(jsonData);
  
  const data = { json: highlighted };
  
  pm.visualizer.set(template, data);
  console.log("✅ Pretty JSON создан");
} catch (error) {
  console.error("❌ Ошибка создания pretty JSON:", error.message);
}
```

---

## Быстрый пример

```javascript
// Вставьте скрипт "1) Visualizer: таблица из массива объектов" в Tests
// Убедитесь, что response содержит массив объектов JSON
// Перейдите на вкладку Visualize для просмотра таблицы
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
