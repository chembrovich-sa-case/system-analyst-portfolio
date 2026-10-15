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
