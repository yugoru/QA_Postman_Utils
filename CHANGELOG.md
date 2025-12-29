# Changelog

Все значимые изменения в проекте документируются в этом файле.

Формат основан на [Keep a Changelog](https://keepachangelog.com/ru/1.0.0/),
и проект следует [Semantic Versioning](https://semver.org/lang/ru/).

## [1.0.0] - 2024-XX-XX

### Добавлено

#### 01_ru_identifiers.md
- Генерация ИНН ЮЛ (10 цифр)
- Генерация ИНН ФЛ (12 цифр)
- Генерация КПП (9 символов)
- Генерация ОГРН (13 цифр)
- Генерация ОГРНИП (15 цифр)
- Генерация СНИЛС (11 цифр, с форматированием)
- Генерация ОКВЭД2 (валидный формат)

#### 02_ru_payments_cards.md
- Генерация PAN карты по Luhn (с поддержкой MIR/Visa/Mastercard/UnionPay)
- Валидация PAN по Luhn
- Генерация CVV (3 или 4 цифры)
- Маскировка PAN для логов
- Генерация срока действия карты (MM/YY)
- НДС: начисление к базе (net → gross)
- НДС: выделение из суммы (gross → net)
- Округление денег (обычное и банковское)
- Шаблон "назначение платежа"

#### 03_ru_bank_accounts.md
- Генерация БИК (форматная)
- Валидация БИК
- Валидация расчётного счёта по БИК
- Валидация корсчёта по БИК
- Генерация "заглушечного" расчётного счёта
- Генерация "заглушечного" корсчёта

#### 04_ru_auto_geo.md
- Генерация госномера РФ (старый и новый формат)
- Валидация госномера РФ
- Справочник кодов регионов РФ (JSON)
- Проверка почтового индекса РФ
- Валидация VIN (ISO 3779)
- Валидация СТС (серия/номер)
- Валидация ПТС (серия/номер)

#### 05_dates_time.md
- Сегодняшняя дата в формате ISO
- Текущий timestamp в миллисекундах
- Генерация даты рождения по возрасту
- Генерация случайной даты в диапазоне
- Добавление дней к дате
- Форматирование даты в YYYY-MM-DD
- Период "последние N дней"

#### 06_strings_encoding.md
- Нормализация ё → е
- Удаление невидимых символов (NBSP, BOM, zero-width и т.д.)
- Trim + collapse spaces
- Очистка для XML/JSON (удаление control chars)

#### 07_postman_visualizers.md
- Visualizer: таблица из массива объектов
- Visualizer: base64 изображение (png/jpg)
- Visualizer: pretty JSON с подсветкой синтаксиса

#### 08_debug_export.md
- Вывод cURL текущего запроса
- Дамп response (status, headers, body)
- Полный дамп request + response

#### 09_notifications.md
- Отправка уведомления в Slack (webhook)
- Отправка уведомления в Telegram (Bot API)
- Пример: отправка при ошибке или статусе >= 400

#### 10_security_light.md
- XSS payload list (20+ payload)
- SQLi payload list (25+ payload)
- Naughty strings (RU/Unicode, 50+ строк)
- Генерация длинной строки
- Boundary strings (граничные случаи)

### Документация
- README.md с полным описанием проекта
- CONTRIBUTING.md с руководством по внесению вклада
- LICENSE (MIT)
- .gitignore
- CHANGELOG.md

---

[1.0.0]: https://github.com/yourusername/qa-postman-utils/releases/tag/v1.0.0

