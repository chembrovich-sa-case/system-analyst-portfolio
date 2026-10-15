# Диаграммы последовательности (Sequence Diagrams)

В данном разделе представлено описание взаимодействия компонентов системы с помощью кода PlantUML.

## UC-1: Добавление транзакции (Расхода)

```plantuml
@startuml
actor "Пользователь" as User
participant "Client (App)" as App
participant "Backend API" as API
database "PostgreSQL" as DB

User -> App: Вводит данные расхода и жмет "Сохранить"
activate App
App -> API: POST /transactions (сумма, счет, категория)
activate API

API -> API: Валидация данных
alt Данные некорректны
    API --> App: 400 Bad Request
    App --> User: Подсветка полей с ошибкой
else Данные корректны
    API -> DB: INSERT INTO transactions
    activate DB
    DB --> API: OK
    deactivate DB
    
    API -> DB: UPDATE accounts SET balance = balance - сумма
    activate DB
    DB --> API: OK
    deactivate DB
    
    API --> App: 201 Created
    App --> User: Обновляет экран, показывает новый баланс
end
deactivate API
deactivate App
@enduml

@startuml
actor "Пользователь" as User
participant "App" as App
participant "Backend" as API
database "PostgreSQL" as DB

User -> App: Заполняет форму перевода (из USD в RUB)
App -> API: GET /rates/cross-course?from=USD&to=RUB
API -> DB: Запрос текущего кросс-курса
DB --> API: Курс = 90.5
API --> App: Возвращает курс
App --> User: Показывает сумму зачисления в RUB

User -> App: Подтверждает перевод
App -> API: POST /transfers (from_acc, to_acc, amount)
activate API
API -> DB: BEGIN
API -> DB: UPDATE счета списания (минус USD)
API -> DB: UPDATE счета зачисления (плюс RUB по курсу)
API -> DB: COMMIT
API --> App: 200 OK
deactivate API
App --> User: Сообщение об успехе перевода
@enduml

@startuml
actor "Пользователь" as User
participant "App" as App
participant "Backend" as API
database "PostgreSQL" as DB
participant "External Rate API" as ExtAPI

User -> App: Открывает дашборд
activate App
App -> API: GET /balance/summary
activate API

API -> DB: Проверка свежих курсов валют на сегодня
activate DB
DB --> API: Курсов за сегодня нет
deactivate DB

API -> ExtAPI: GET /latest?base=RUB
activate ExtAPI
alt Внешний сервис доступен
    ExtAPI --> API: JSON с курсами (200 OK)
    API -> DB: Сохранение свежих курсов в кэш
else Внешний сервис недоступен (Таймаут)
    ExtAPI --> API: 503 / Timeout
    API -> DB: Извлечение вчерашних курсов (Fallback)
end
deactivate ExtAPI

API -> DB: Запрос остатков по всем счетам пользователя
activate DB
DB --> API: Данные по счетам
deactivate DB

API -> API: Конвертация валют в базовую (RUB) и расчет
API --> App: Общий баланс и список операций
deactivate API
App --> User: Отображение баланса на экране
deactivate App
@enduml
