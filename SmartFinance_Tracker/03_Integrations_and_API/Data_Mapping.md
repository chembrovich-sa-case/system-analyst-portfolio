# Data Mapping (Маппинг данных интеграции)

В данном документе описано правило преобразования (маппинга) данных, получаемых от внешнего REST API курсов валют (например, ExchangeRate-API), для сохранения в локальную базу данных трекера.

**Направление:** Внешний API (Source) -> БД SmartFinance Tracker (Target)
**Эндпоинт источника:** `GET https://v6.exchangerate-api.com/v6/{API_KEY}/latest/RUB`
**Целевая таблица в БД:** `exchange_rates`

## Таблица маппинга

| Source Field (Поле в ответе API JSON) | Тип данных (Source) | Target Field (Поле в нашей БД) | Тип данных (Target) | Правило преобразования (Transformation Rule) | Обязательность |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `time_last_update_utc` | String (Date) | `fetch_date` | DATE | Конвертировать строку формата "Thu, 15 Oct 2026 00:00:00 +0000" в стандартный формат даты `YYYY-MM-DD`. | Да (NOT NULL) |
| `base_code` | String | `base_currency` | VARCHAR(3) | Прямое копирование (например, "RUB"). | Да (NOT NULL) |
| `conversion_rates` (ключ объекта) | String | `target_currency` | VARCHAR(3) | Извлечь ключ валюты из массива/объекта (например, "USD", "EUR"). | Да (NOT NULL) |
| `conversion_rates` (значение) | Float | `rate` | DECIMAL(10,4) | Прямое копирование числового значения курса (например, 0.0105). | Да (NOT NULL) |

## Пример ответа внешнего API (Payload)
```json
{
  "result": "success",
  "documentation": "[https://www.exchangerate-api.com/docs](https://www.exchangerate-api.com/docs)",
  "terms_of_use": "[https://www.exchangerate-api.com/terms](https://www.exchangerate-api.com/terms)",
  "time_last_update_unix": 1709251200,
  "time_last_update_utc": "Thu, 15 Oct 2026 00:00:00 +0000",
  "time_next_update_unix": 1709337600,
  "time_next_update_utc": "Fri, 16 Oct 2026 00:00:00 +0000",
  "base_code": "RUB",
  "conversion_rates": {
    "RUB": 1,
    "USD": 0.0105,
    "EUR": 0.0097,
    "KZT": 4.75
  }
}
