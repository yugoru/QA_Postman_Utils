# 02 — Платежи и банковские карты (PAN / BIN / MIR / Luhn / CVV / Expiry) + НДС

Все скрипты ниже вставляются в **Pre-request Script** и пишут результат в **Globals**.

**Как установить входные переменные**:
- **Рекомендуется**: прямо в скрипте через `pm.globals.set("card_brand", "mir")` перед основным скриптом
- **Альтернатива**: вручную через интерфейс Postman (меню Globals)

---

## Переменные (Globals)

### Входные (необязательно)
- `card_brand` — бренд карты: `mir` | `visa` | `mastercard` | `unionpay` | `custom` (по умолчанию `mir`)
  - **Как использовать**: установите через `pm.globals.set("card_brand", "mir")` в скрипте или вручную в Globals
  - **Примеры**: 
    - Для МИР: `pm.globals.set("card_brand", "mir")`
    - Для Visa: `pm.globals.set("card_brand", "visa")`
    - Для Mastercard: `pm.globals.set("card_brand", "mastercard")`
- `card_bin` — BIN (минимум 6 цифр). Если задан — используется всегда, игнорируя `card_brand`
- `card_len` — длина PAN (12..19), по умолчанию 16
- `card_exp_years_ahead` — максимум лет вперёд для expiry, по умолчанию 5
- `pay_amount_net` — база (без НДС) для начисления НДС
- `pay_amount_gross` — сумма с НДС для выделения НДС
- `pay_nds_rate` — ставка НДС (например `0.2` для 20%), по умолчанию `0.2`
- `pay_round_value` — значение для округления
- `card_pan_in` — PAN для валидации (если не задан, используется `card_pan`)
- `pay_invoice` — номер счёта для назначения платежа
- `pay_contract` — номер договора для назначения платежа
- `pay_act` — номер акта для назначения платежа
- `pay_text` — произвольный текст для назначения платежа
- `pay_vat_rate` — ставка НДС для назначения платежа (например `20%`)

### Выходные
- `card_pan` — номер карты (валидный Luhn)
- `card_brand_used` — какой бренд реально использовали
- `card_bin_used` — какой BIN реально использовали
- `card_cvv` — CVV (3 цифры, 000-999)
- `card_cvv_len` — длина CVV (3 или 4), по умолчанию 3
- `card_exp_mm` — месяц (MM, 01-12)
- `card_exp_yy` — год (YY, последние 2 цифры)
- `card_pan_masked` — маскированный PAN (формат `**** **** **** 1234`)
- `card_pan_luhn_ok` — результат валидации Luhn (true/false)
- `card_pan_luhn_reason` — причина валидации Luhn
- `pay_nds_net` — база (без НДС)
- `pay_nds_amount` — сумма НДС
- `pay_nds_gross` — сумма с НДС
- `pay_round2` — округление до 2 знаков (обычное)
- `pay_bankers_round2` — округление до 2 знаков (банковское)
- `pay_purpose` — назначение платежа (сформированное)

---

## Примеры использования

**Важно**: Переменные можно установить двумя способами:
1. **Прямо в скрипте** (рекомендуется) — добавьте строки `pm.globals.set()` перед основным скриптом
2. **В интерфейсе Postman** — через меню Globals вручную

### Генерация карты МИР

Добавьте в начало Pre-request Script (перед основным скриптом генерации PAN):
```javascript
// Установка параметров для МИР
pm.globals.set("card_brand", "mir");
pm.globals.set("card_len", 16); // опционально, по умолчанию 16
```

### Генерация карты Visa

Добавьте в начало Pre-request Script:
```javascript
// Установка параметров для Visa
pm.globals.set("card_brand", "visa");
pm.globals.set("card_len", 16); // опционально
```

### Генерация карты Mastercard

Добавьте в начало Pre-request Script:
```javascript
// Установка параметров для Mastercard
pm.globals.set("card_brand", "mastercard");
pm.globals.set("card_len", 16); // опционально
```

### Генерация карты с конкретным BIN

Добавьте в начало Pre-request Script:
```javascript
// Установка конкретного BIN (минимум 6 цифр)
pm.globals.set("card_bin", "2200123456");
pm.globals.set("card_len", 16); // опционально
```

**Примечание**: Если задан `card_bin`, он имеет приоритет над `card_brand`.


## 1) Сгенерировать PAN карты (brand/BIN/длина) и сохранить в Globals

**Что делает**: Генерирует валидный номер карты (PAN) с правильной контрольной суммой по алгоритму Luhn.

**Приоритет источника BIN**:
1. Если задан `card_bin` в Globals — используется он (минимум 6 цифр)
2. Если задан `card_brand` — используется BIN из пресета для этого бренда
3. Если ничего не задано — генерируется случайный валидный Luhn без привязки к BIN

**Примеры**:
- Для МИР: установите `card_brand = mir` → будет использован BIN из пресета (220012, 220220 или 220400)
- Для Visa: установите `card_brand = visa` → будет использован BIN из пресета (400000, 411111 или 427600)
- Свой BIN: установите `card_bin = 1234567890` → будет использован именно этот BIN

```javascript
/**
 * Генерация PAN по алгоритму Luhn.
 * Приоритет источника BIN:
 * 1) pm.globals.get("card_bin") — если задан (минимум 6 цифр)
 * 2) пресеты по бренду (MIR/Visa/MC/UnionPay)
 * 3) fallback: случайный валидный Luhn без BIN
 */

function digitsOnly(v) {
  return String(v ?? "").replace(/\D+/g, "");
}

function padLeft(n, len, ch) {
  const s = String(n);
  return s.length >= len ? s : (String(ch || "0").repeat(len - s.length) + s);
}

function randInt(min, max) {
  // inclusive
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function pick(arr) {
  return arr[randInt(0, arr.length - 1)];
}

function luhnCheckDigit(numberWithoutCheck) {
  const s = digitsOnly(numberWithoutCheck);
  let sum = 0;

  // удваиваем каждую вторую цифру, начиная с правой в теле номера
  let doubleIt = true;
  for (let i = s.length - 1; i >= 0; i--) {
    let n = Number(s[i]);
    if (doubleIt) {
      n = n * 2;
      if (n > 9) n = n - 9;
    }
    sum += n;
    doubleIt = !doubleIt;
  }
  const mod = sum % 10;
  return String((10 - mod) % 10);
}

function generatePanByBin(bin, length) {
  const b = digitsOnly(bin);
  const len = Number(length);

  if (b.length < 6) throw new Error("BIN должен быть минимум 6 цифр");
  if (!(len >= 12 && len <= 19)) throw new Error("card_len должен быть в диапазоне 12..19");

  const bodyLen = len - 1;
  let body = b;

  while (body.length < bodyLen) {
    body += String(randInt(0, 9));
  }
  body = body.slice(0, bodyLen);

  const cd = luhnCheckDigit(body);
  return body + cd;
}

function generateRandomLuhn(length) {
  const len = Number(length);
  if (!(len >= 12 && len <= 19)) throw new Error("card_len должен быть в диапазоне 12..19");

  let body = "";
  while (body.length < (len - 1)) body += String(randInt(0, 9));
  body = body.slice(0, len - 1);

  return body + luhnCheckDigit(body);
}

// BIN пресеты (тестовые). Можно расширять под свои диапазоны.
// ВАЖНО: все BIN должны быть минимум 6 цифр!
const BIN_PRESETS = {
  mir: ["220012", "220220", "220400"],     // MIR: тестовые BIN (минимум 6 цифр)
  visa: ["400000", "411111", "427600"],    // Visa: типичные тестовые BIN
  mastercard: ["510000", "520000", "530000"], // Mastercard: тестовые BIN
  unionpay: ["620000"]                     // UnionPay: тестовый BIN
};

const brand = String(pm.globals.get("card_brand") || "mir").toLowerCase();
const forcedBin = digitsOnly(pm.globals.get("card_bin"));
const len = Number(pm.globals.get("card_len") || 16);

let binUsed = "";
let brandUsed = brand;
let pan = "";

// 1) Явный BIN из Globals — приоритет
if (forcedBin) {
  binUsed = forcedBin;
  brandUsed = (brand === "custom") ? "custom" : brand;
  pan = generatePanByBin(binUsed, len);
} else {
  // 2) BIN из пресета по бренду
  const preset = BIN_PRESETS[brand];
  if (preset && preset.length > 0) {
    binUsed = pick(preset);
    pan = generatePanByBin(binUsed, len);
  } else {
    // 3) fallback — просто валидный Luhn без BIN
    brandUsed = "custom";
    binUsed = "";
    pan = generateRandomLuhn(len);
  }
}

pm.globals.set("card_pan", pan);
pm.globals.set("card_brand_used", brandUsed);
pm.globals.set("card_bin_used", binUsed);

console.log("card_pan =", pan);
console.log("card_brand_used =", brandUsed);
console.log("card_bin_used =", binUsed);
```

---

## 2) Валидация PAN по Luhn

**Что делает**: Проверяет валидность номера карты по алгоритму Luhn.

**Входные переменные**:
- `card_pan_in` (опционально) — PAN для проверки. Если не задан, используется `card_pan`

**Выходные переменные**:
- `card_pan_luhn_ok` — результат валидации (true/false)
- `card_pan_luhn_reason` — причина (строка с описанием)

```javascript
function digitsOnly(v) {
  return String(v ?? "").replace(/\D+/g, "");
}

function luhnValidate(pan) {
  const s = digitsOnly(pan);
  if (s.length < 12 || s.length > 19) {
    return { ok: false, reason: "PAN должен быть от 12 до 19 цифр" };
  }

  let sum = 0;
  let doubleIt = false;
  
  for (let i = s.length - 1; i >= 0; i--) {
    let n = Number(s[i]);
    if (doubleIt) {
      n = n * 2;
      if (n > 9) n = n - 9;
    }
    sum += n;
    doubleIt = !doubleIt;
  }

  const isValid = (sum % 10) === 0;
  return {
    ok: isValid,
    reason: isValid ? "Валидный PAN по Luhn" : "Невалидный PAN (контрольная сумма не совпадает)"
  };
}

const panToCheck = pm.globals.get("card_pan_in") || pm.globals.get("card_pan") || "";
const result = luhnValidate(panToCheck);

pm.globals.set("card_pan_luhn_ok", result.ok);
pm.globals.set("card_pan_luhn_reason", result.reason);

console.log("card_pan_luhn_ok =", result.ok);
console.log("card_pan_luhn_reason =", result.reason);
```

---

## 3) Сгенерировать CVV и сохранить в Globals

**Что делает**: Генерирует случайный CVV код (3 или 4 цифры) и сохраняет в `card_cvv`.

**Входные переменные**:
- `card_cvv_len` (опционально) — длина CVV (3 или 4), по умолчанию 3

**Выходная переменная**: `card_cvv` (3 или 4 цифры)

```javascript
const cvvLen = Number(pm.globals.get("card_cvv_len") || 3);
const maxValue = cvvLen === 4 ? 10000 : 1000;
const cvv = String(Math.floor(Math.random() * maxValue)).padStart(cvvLen, "0");

pm.globals.set("card_cvv", cvv);
console.log("card_cvv =", cvv);
```

---

## 4) Маскировка PAN для логов

**Что делает**: Маскирует номер карты, оставляя видимыми только последние 4 цифры.

**Входные переменные**:
- `card_pan` — номер карты для маскировки

**Выходная переменная**: `card_pan_masked` (формат `**** **** **** 1234`)

```javascript
function maskPan(pan) {
  const s = String(pan ?? "").replace(/\D+/g, "");
  if (s.length < 4) return "****";
  
  const last4 = s.slice(-4);
  const groups = [];
  for (let i = 0; i < s.length - 4; i += 4) {
    groups.push("****");
  }
  groups.push(last4);
  return groups.join(" ");
}

const pan = pm.globals.get("card_pan") || "";
const masked = maskPan(pan);

pm.globals.set("card_pan_masked", masked);
console.log("card_pan_masked =", masked);
```

---

## 5) Сгенерировать срок действия карты (MM/YY) и сохранить в Globals

**Что делает**: Генерирует случайный срок действия карты (месяц и год) в будущем.

**Входные переменные**:
- `card_exp_years_ahead` (опционально) — максимум лет вперёд от текущей даты, по умолчанию 5

**Выходные переменные**:
- `card_exp_mm` — месяц (01-12)
- `card_exp_yy` — год (последние 2 цифры, например 25 для 2025)

**Пример**: Если сейчас 2024 год и `card_exp_years_ahead = 5`, то год будет от 2024 до 2029 включительно.

```javascript
const yearsAhead = Number(pm.globals.get("card_exp_years_ahead") || 5);
const now = new Date();

const mm = String(Math.floor(Math.random() * 12) + 1).padStart(2, "0");
const futureYear = now.getFullYear() + Math.floor(Math.random() * (yearsAhead + 1));
const yy = String(futureYear).slice(-2);

pm.globals.set("card_exp_mm", mm);
pm.globals.set("card_exp_yy", yy);

console.log("card_exp_mm =", mm);
console.log("card_exp_yy =", yy);
```

---

## 6) НДС: начислить к базе (net → gross) и сохранить в Globals

**Что делает**: Рассчитывает НДС от базы (суммы без НДС) и получает итоговую сумму с НДС.

**Формула**: 
- НДС = база × ставка
- Сумма с НДС = база + НДС

**Входные переменные**:
- `pay_amount_net` — база (сумма без НДС), например `1000`
- `pay_nds_rate` (опционально) — ставка НДС, по умолчанию `0.2` (20%)

**Выходные переменные**:
- `pay_nds_net` — база (без НДС)
- `pay_nds_amount` — сумма НДС
- `pay_nds_gross` — сумма с НДС

**Пример**: 
- База = 1000, ставка = 0.2 (20%)
- НДС = 200
- Итого = 1200

```javascript
const net = Number(pm.globals.get("pay_amount_net") || 0);
const r = Number(pm.globals.get("pay_nds_rate") || 0.2);

const nds = +(net * r).toFixed(2);
const gross = +(net + nds).toFixed(2);

pm.globals.set("pay_nds_net", net);
pm.globals.set("pay_nds_amount", nds);
pm.globals.set("pay_nds_gross", gross);

console.log("pay_nds_net =", net);
console.log("pay_nds_amount =", nds);
console.log("pay_nds_gross =", gross);
```

---

## 7) НДС: выделить из суммы (gross → net) и сохранить в Globals

**Что делает**: Выделяет НДС из суммы с НДС и рассчитывает базу (сумму без НДС).

**Формула**: 
- База = Сумма с НДС ÷ (1 + ставка)
- НДС = Сумма с НДС - База

**Входные переменные**:
- `pay_amount_gross` — сумма с НДС, например `1200`
- `pay_nds_rate` (опционально) — ставка НДС, по умолчанию `0.2` (20%)

**Выходные переменные**:
- `pay_nds_net` — база (без НДС)
- `pay_nds_amount` — сумма НДС
- `pay_nds_gross` — сумма с НДС

**Пример**: 
- Сумма с НДС = 1200, ставка = 0.2 (20%)
- База = 1200 ÷ 1.2 = 1000
- НДС = 1200 - 1000 = 200

```javascript
const gross = Number(pm.globals.get("pay_amount_gross") || 0);
const r = Number(pm.globals.get("pay_nds_rate") || 0.2);

const net = +(gross / (1 + r)).toFixed(2);
const nds = +(gross - net).toFixed(2);

pm.globals.set("pay_nds_net", net);
pm.globals.set("pay_nds_amount", nds);
pm.globals.set("pay_nds_gross", gross);

console.log("pay_nds_net =", net);
console.log("pay_nds_amount =", nds);
console.log("pay_nds_gross =", gross);
```

---

## 8) Округление денег (обычное и банковское)

**Что делает**: Округляет денежное значение до 2 знаков после запятой обычным способом и банковским округлением.

**Банковское округление** (round half to even): если число ровно посередине (например, 2.5), округляется к ближайшему чётному числу.

**Входные переменные**:
- `pay_round_value` — значение для округления

**Выходные переменные**:
- `pay_round2` — округление обычным способом (до 2 знаков)
- `pay_bankers_round2` — банковское округление (до 2 знаков)

```javascript
function bankersRound(value, decimals = 2) {
  const factor = Math.pow(10, decimals);
  const rounded = Math.round(value * factor) / factor;
  const diff = Math.abs(value * factor - Math.round(value * factor));
  
  // Если разница ровно 0.5 (посередине)
  if (Math.abs(diff - 0.5) < 0.0001) {
    const floor = Math.floor(value * factor);
    // Округляем к ближайшему чётному
    return (floor % 2 === 0 ? floor : floor + 1) / factor;
  }
  
  return rounded;
}

const value = Number(pm.globals.get("pay_round_value") || 0);
const round2 = +(value.toFixed(2));
const bankersRound2 = +bankersRound(value, 2).toFixed(2);

pm.globals.set("pay_round2", round2);
pm.globals.set("pay_bankers_round2", bankersRound2);

console.log("pay_round2 =", round2);
console.log("pay_bankers_round2 =", bankersRound2);
```

---

## 9) Шаблон "назначение платежа"

**Что делает**: Формирует текст назначения платежа на основе входных данных (счёт, договор, акт, текст, НДС).

**Варианты использования**:
- Без НДС: используйте `pay_invoice`, `pay_contract`, `pay_act`, `pay_text`
- С НДС: используйте `pay_invoice`, `pay_contract`, `pay_act`, `pay_vat_rate`

**Входные переменные**:
- `pay_invoice` (опционально) — номер счёта
- `pay_contract` (опционально) — номер договора
- `pay_act` (опционально) — номер акта
- `pay_text` (опционально) — произвольный текст
- `pay_vat_rate` (опционально) — ставка НДС (например `20%`)

**Выходная переменная**: `pay_purpose` (сформированное назначение платежа)

```javascript
function buildPaymentPurpose(invoice, contract, act, text, vatRate) {
  const parts = [];
  
  if (invoice) parts.push(`Счёт №${invoice}`);
  if (contract) parts.push(`Договор №${contract}`);
  if (act) parts.push(`Акт №${act}`);
  if (text) parts.push(text);
  
  let purpose = parts.join(", ");
  
  if (vatRate) {
    purpose += `. НДС ${vatRate}`;
  }
  
  return purpose || "Оплата по договору";
}

const invoice = pm.globals.get("pay_invoice") || "";
const contract = pm.globals.get("pay_contract") || "";
const act = pm.globals.get("pay_act") || "";
const text = pm.globals.get("pay_text") || "";
const vatRate = pm.globals.get("pay_vat_rate") || "";

const purpose = buildPaymentPurpose(invoice, contract, act, text, vatRate);

pm.globals.set("pay_purpose", purpose);
console.log("pay_purpose =", purpose);
```

---

## Быстрый пример

```javascript
// Установка параметров
pm.globals.set("card_brand", "mir");
pm.globals.set("card_len", 16);

// Вставьте скрипт "1) Сгенерировать PAN карты"
// Вставьте скрипт "3) Сгенерировать CVV"
// Вставьте скрипт "5) Сгенерировать срок действия"

// Результат: {{card_pan}}, {{card_cvv}}, {{card_exp_mm}}, {{card_exp_yy}}
```

---

## При поддержке

Этот проект создан при поддержке [школы Эрмита](https://ermita.one/) — онлайн IT-школы для тестировщиков.

📚 [Курсы для тестировщиков](https://ermita.one/courses/) | 💬 [Телеграм-канал](https://t.me/ermita_one) | 🎮 [Тренажеры для QA](http://qahacking.ru/)
