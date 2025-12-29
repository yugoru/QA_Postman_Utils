# 10 — Безопасность (XSS / SQLi / naughty strings)

Все скрипты ниже вставляются в **Pre-request Script** и генерируют тестовые payload для проверки безопасности.

**ВАЖНО**: Эти скрипты предназначены ТОЛЬКО для тестирования собственных приложений. Не используйте для эксплуатации!

---

## Переменные (Globals)

### Входные (необязательно)
- `sec_len` — длина для генерации длинной строки

### Выходные
- `sec_xss_list` — массив XSS payload
- `sec_xss_pick` — один случайный XSS payload
- `sec_sqli_list` — массив SQLi payload
- `sec_sqli_pick` — один случайный SQLi payload
- `sec_naughty_list` — массив "naughty strings" (проблемные строки)
- `sec_long_string` — длинная строка заданной длины

---

## 1) XSS payload list

**Что делает**: Генерирует список тестовых XSS payload и сохраняет в Globals.

**Выходные переменные**:
- `sec_xss_list` — массив XSS payload (JSON-строка)
- `sec_xss_pick` — один случайный payload из списка

```javascript
const xssPayloads = [
  "<script>alert('XSS')</script>",
  "<img src=x onerror=alert('XSS')>",
  "<svg onload=alert('XSS')>",
  "javascript:alert('XSS')",
  "<body onload=alert('XSS')>",
  "<iframe src=javascript:alert('XSS')>",
  "<input onfocus=alert('XSS') autofocus>",
  "<select onfocus=alert('XSS') autofocus>",
  "<textarea onfocus=alert('XSS') autofocus>",
  "<keygen onfocus=alert('XSS') autofocus>",
  "<video><source onerror=alert('XSS')>",
  "<audio src=x onerror=alert('XSS')>",
  "<details open ontoggle=alert('XSS')>",
  "<marquee onstart=alert('XSS')>",
  "'\"><script>alert('XSS')</script>",
  "<script>alert(String.fromCharCode(88,83,83))</script>",
  "<img src=x onerror=\"alert('XSS')\">",
  "<svg/onload=alert('XSS')>",
  "<body/onload=alert('XSS')>",
  "<iframe/src=javascript:alert('XSS')>"
];

function pick(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

const xssListJson = JSON.stringify(xssPayloads);
const xssPick = pick(xssPayloads);

pm.globals.set("sec_xss_list", xssListJson);
pm.globals.set("sec_xss_pick", xssPick);

console.log("sec_xss_list сохранён (", xssPayloads.length, "payload)");
console.log("sec_xss_pick =", xssPick);
```

---

## 2) SQLi payload list

**Что делает**: Генерирует список тестовых SQL injection payload и сохраняет в Globals.

**Выходные переменные**:
- `sec_sqli_list` — массив SQLi payload (JSON-строка)
- `sec_sqli_pick` — один случайный payload из списка

```javascript
const sqliPayloads = [
  "' OR '1'='1",
  "' OR '1'='1' --",
  "' OR '1'='1' /*",
  "admin'--",
  "admin'/*",
  "' UNION SELECT NULL--",
  "' UNION SELECT NULL,NULL--",
  "' UNION SELECT NULL,NULL,NULL--",
  "1' OR '1'='1",
  "1' OR '1'='1' --",
  "1' OR '1'='1' /*",
  "' OR 1=1--",
  "' OR 1=1/*",
  "') OR ('1'='1",
  "') OR ('1'='1'--",
  "') OR ('1'='1'/*",
  "1') OR ('1'='1",
  "1') OR ('1'='1'--",
  "1') OR ('1'='1'/*",
  "'; DROP TABLE users--",
  "'; DROP TABLE users/*",
  "' UNION SELECT username,password FROM users--",
  "' UNION SELECT * FROM users--",
  "1' AND 1=1--",
  "1' AND 1=2--",
  "1' AND '1'='1",
  "1' AND '1'='2"
];

function pick(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

const sqliListJson = JSON.stringify(sqliPayloads);
const sqliPick = pick(sqliPayloads);

pm.globals.set("sec_sqli_list", sqliListJson);
pm.globals.set("sec_sqli_pick", sqliPick);

console.log("sec_sqli_list сохранён (", sqliPayloads.length, "payload)");
console.log("sec_sqli_pick =", sqliPick);
```

---

## 3) Naughty strings (RU/Unicode)

**Что делает**: Генерирует список "naughty strings" — проблемных строк, которые могут вызвать ошибки в приложениях.

**Выходная переменная**: `sec_naughty_list` — массив naughty strings (JSON-строка)

```javascript
const naughtyStrings = [
  // Пустые и пробельные
  "",
  " ",
  "  ",
  "\t",
  "\n",
  "\r",
  "\r\n",
  
  // Специальные символы
  "!@#$%^&*()",
  "`~-_=+[]{}|\\;:'\",.<>/?",
  
  // Unicode
  "🚀",
  "🎉",
  "Привет мир",
  "Здравствуй, мир!",
  "Тест 123",
  
  // Эмодзи микс
  "🚀🎉💻🔥",
  "Привет 🎉 мир",
  
  // Кириллица + латиница
  "Hello Привет",
  "Test Тест 123",
  "АБВГД abcdefg 123",
  
  // Длинные строки
  "A".repeat(1000),
  "Привет".repeat(100),
  
  // Граничные случаи
  "0",
  "-1",
  "null",
  "undefined",
  "true",
  "false",
  "NaN",
  "Infinity",
  "-Infinity",
  
  // SQL-подобные
  "SELECT * FROM users",
  "DROP TABLE users",
  
  // HTML/JS-подобные
  "<script>",
  "</script>",
  "<img>",
  "javascript:",
  
  // Пути
  "../../../etc/passwd",
  "C:\\Windows\\System32",
  "/etc/passwd",
  
  // URL-подобные
  "http://example.com",
  "https://example.com/test?param=value",
  
  // JSON-подобные
  '{"key":"value"}',
  '[1,2,3]',
  'null',
  'true',
  'false',
  
  // XML-подобные
  "<?xml version='1.0'?>",
  "<root><child>value</child></root>",
  
  // Строки с кавычками
  "'single quote'",
  '"double quote"',
  "`backtick`",
  
  // Строки с переносами
  "Line 1\nLine 2",
  "Line 1\rLine 2",
  "Line 1\r\nLine 2",
  
  // Строки с табуляцией
  "Col1\tCol2\tCol3",
  
  // Строки с нулевым байтом
  "test\0null",
  
  // Строки с пробелами в начале/конце
  "  leading spaces",
  "trailing spaces  ",
  "  both sides  "
];

const naughtyListJson = JSON.stringify(naughtyStrings);

pm.globals.set("sec_naughty_list", naughtyListJson);

console.log("sec_naughty_list сохранён (", naughtyStrings.length, "строк)");
```

---

## 4) Генерация длинной строки

**Что делает**: Генерирует длинную строку заданной длины для тестирования обработки больших данных.

**Входные переменные**:
- `sec_len` — длина строки (по умолчанию 10000)

**Выходная переменная**: `sec_long_string` — длинная строка

```javascript
const length = Number(pm.globals.get("sec_len") || 10000);

// Генерируем строку из повторяющегося паттерна
const pattern = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
let longString = "";

while (longString.length < length) {
  longString += pattern;
}

// Обрезаем до нужной длины
longString = longString.substring(0, length);

pm.globals.set("sec_long_string", longString);

console.log("sec_long_string создана, длина:", longString.length);
```

---

## 5) Boundary strings (граничные случаи)

**Что делает**: Генерирует специальные граничные строки для тестирования.

**Выходные переменные**: отдельные переменные для каждого типа

```javascript
pm.globals.set("sec_boundary_empty", "");
pm.globals.set("sec_boundary_space", " ");
pm.globals.set("sec_boundary_tab", "\t");
pm.globals.set("sec_boundary_newline", "\n");
pm.globals.set("sec_boundary_long", "A".repeat(100000));
pm.globals.set("sec_boundary_emoji_mix", "🚀🎉💻🔥 Привет мир Hello 123");
pm.globals.set("sec_boundary_cyrillic_latin", "Привет Hello Тест Test 123");

console.log("Boundary strings созданы:");
console.log("  sec_boundary_empty = (пустая строка)");
console.log("  sec_boundary_space = (один пробел)");
console.log("  sec_boundary_tab = (табуляция)");
console.log("  sec_boundary_newline = (новая строка)");
console.log("  sec_boundary_long = (100000 символов)");
console.log("  sec_boundary_emoji_mix = (эмодзи микс)");
console.log("  sec_boundary_cyrillic_latin = (кириллица + латиница)");
```

---

## Быстрый пример

```javascript
// Вставьте скрипт "1) XSS payload list"
// Вставьте скрипт "2) SQLi payload list"

// Использование:
// pm.globals.get("sec_xss_pick") — случайный XSS payload
// pm.globals.get("sec_sqli_pick") — случайный SQLi payload
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
