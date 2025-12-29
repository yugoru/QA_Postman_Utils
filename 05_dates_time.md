# 05 — Даты и время (ISO / timestamp / интервалы)

Все скрипты ниже вставляются в **Pre-request Script** и пишут результат в **Globals**.

**Как установить входные переменные**:
- **Рекомендуется**: прямо в скрипте через `pm.globals.set()` перед основным скриптом
- **Альтернатива**: вручную через интерфейс Postman (меню Globals)

---

## Переменные (Globals)

### Входные (необязательно)
- `dt_age_years` — возраст в годах для генерации даты рождения
- `dt_from_iso` — начальная дата в формате ISO (YYYY-MM-DD)
- `dt_to_iso` — конечная дата в формате ISO (YYYY-MM-DD)
- `dt_base_iso` — базовая дата в формате ISO (YYYY-MM-DD)
- `dt_add_days` — количество дней для добавления
- `dt_in` — дата для форматирования
- `dt_last_n` — количество дней назад для периода

### Выходные
- `dt_today_iso` — сегодняшняя дата в формате ISO (YYYY-MM-DD)
- `dt_now_ms` — текущий timestamp в миллисекундах
- `dt_birthdate_iso` — дата рождения в формате ISO
- `dt_rand_iso` — случайная дата в формате ISO
- `dt_result_iso` — результат добавления дней в формате ISO
- `dt_ymd` — дата в формате YYYY-MM-DD
- `dt_range_from_iso` — начало периода в формате ISO
- `dt_range_to_iso` — конец периода в формате ISO

---

## 1) Сегодняшняя дата в формате ISO

**Что делает**: Получает текущую дату в формате ISO (YYYY-MM-DD).

**Входные переменные**: не требуются

**Выходная переменная**: `dt_today_iso` (YYYY-MM-DD)

```javascript
const today = new Date();
const iso = today.toISOString().split('T')[0];

pm.globals.set("dt_today_iso", iso);
console.log("dt_today_iso =", iso);
```

---

## 2) Текущий timestamp в миллисекундах

**Что делает**: Получает текущий timestamp в миллисекундах (Unix timestamp × 1000).

**Входные переменные**: не требуются

**Выходная переменная**: `dt_now_ms` (число)

```javascript
const now = Date.now();

pm.globals.set("dt_now_ms", now);
console.log("dt_now_ms =", now);
```

---

## 3) Генерация даты рождения по возрасту

**Что делает**: Генерирует дату рождения на основе указанного возраста.

**Входные переменные**:
- `dt_age_years` — возраст в годах

**Выходная переменная**: `dt_birthdate_iso` (YYYY-MM-DD)

```javascript
function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const age = Number(pm.globals.get("dt_age_years") || 30);
const today = new Date();
const birthYear = today.getFullYear() - age;
const birthMonth = randInt(1, 12);
const daysInMonth = new Date(birthYear, birthMonth, 0).getDate();
const birthDay = randInt(1, daysInMonth);

const birthdate = new Date(birthYear, birthMonth - 1, birthDay);
const iso = birthdate.toISOString().split('T')[0];

pm.globals.set("dt_birthdate_iso", iso);
console.log("dt_birthdate_iso =", iso);
```

---

## 4) Генерация случайной даты в диапазоне

**Что делает**: Генерирует случайную дату между двумя указанными датами.

**Входные переменные**:
- `dt_from_iso` — начальная дата в формате ISO (YYYY-MM-DD)
- `dt_to_iso` — конечная дата в формате ISO (YYYY-MM-DD)

**Выходная переменная**: `dt_rand_iso` (YYYY-MM-DD)

```javascript
function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const fromStr = pm.globals.get("dt_from_iso") || "2020-01-01";
const toStr = pm.globals.get("dt_to_iso") || "2024-12-31";

const from = new Date(fromStr);
const to = new Date(toStr);

if (isNaN(from.getTime()) || isNaN(to.getTime())) {
  throw new Error("Неверный формат даты. Используйте YYYY-MM-DD");
}

if (from > to) {
  throw new Error("Начальная дата должна быть раньше конечной");
}

const fromTime = from.getTime();
const toTime = to.getTime();
const randomTime = fromTime + randInt(0, toTime - fromTime);
const randomDate = new Date(randomTime);

const iso = randomDate.toISOString().split('T')[0];

pm.globals.set("dt_rand_iso", iso);
console.log("dt_rand_iso =", iso);
```

---

## 5) Добавление дней к дате

**Что делает**: Добавляет указанное количество дней к базовой дате.

**Входные переменные**:
- `dt_base_iso` — базовая дата в формате ISO (YYYY-MM-DD)
- `dt_add_days` — количество дней для добавления (может быть отрицательным)

**Выходная переменная**: `dt_result_iso` (YYYY-MM-DD)

```javascript
const baseStr = pm.globals.get("dt_base_iso") || new Date().toISOString().split('T')[0];
const addDays = Number(pm.globals.get("dt_add_days") || 0);

const base = new Date(baseStr);
if (isNaN(base.getTime())) {
  throw new Error("Неверный формат базовой даты. Используйте YYYY-MM-DD");
}

base.setDate(base.getDate() + addDays);
const iso = base.toISOString().split('T')[0];

pm.globals.set("dt_result_iso", iso);
console.log("dt_result_iso =", iso);
```

---

## 6) Форматирование даты в YYYY-MM-DD

**Что делает**: Преобразует входную дату в формат YYYY-MM-DD.

**Входные переменные**:
- `dt_in` — дата для форматирования (может быть ISO строкой, timestamp или Date объектом)

**Выходная переменная**: `dt_ymd` (YYYY-MM-DD)

```javascript
function formatYMD(input) {
  if (!input) {
    const today = new Date();
    return today.toISOString().split('T')[0];
  }
  
  let date;
  
  if (typeof input === 'number') {
    // Timestamp
    date = new Date(input);
  } else if (typeof input === 'string') {
    // ISO строка или другой формат
    date = new Date(input);
  } else {
    date = new Date();
  }
  
  if (isNaN(date.getTime())) {
    throw new Error("Неверный формат даты");
  }
  
  return date.toISOString().split('T')[0];
}

const dtIn = pm.globals.get("dt_in") || new Date().toISOString();
const ymd = formatYMD(dtIn);

pm.globals.set("dt_ymd", ymd);
console.log("dt_ymd =", ymd);
```

---

## 7) Период "последние N дней"

**Что делает**: Генерирует диапазон дат от N дней назад до сегодня.

**Входные переменные**:
- `dt_last_n` — количество дней назад

**Выходные переменные**:
- `dt_range_from_iso` — начальная дата периода (YYYY-MM-DD)
- `dt_range_to_iso` — конечная дата периода (сегодня, YYYY-MM-DD)

```javascript
const lastN = Number(pm.globals.get("dt_last_n") || 7);

const today = new Date();
const fromDate = new Date(today);
fromDate.setDate(today.getDate() - lastN);

const fromIso = fromDate.toISOString().split('T')[0];
const toIso = today.toISOString().split('T')[0];

pm.globals.set("dt_range_from_iso", fromIso);
pm.globals.set("dt_range_to_iso", toIso);

console.log("dt_range_from_iso =", fromIso);
console.log("dt_range_to_iso =", toIso);
```

---

## Быстрый пример

```javascript
// Вставьте скрипт "1) Сегодняшняя дата в формате ISO"
// Вставьте скрипт "2) Текущий timestamp в миллисекундах"

// Результат: {{dt_today_iso}}, {{dt_now_ms}}
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
