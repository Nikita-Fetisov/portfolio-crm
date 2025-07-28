<style>
/* Разрыв страницы перед каждым H1 */
h2 {
    page-break-before: always;
}

/* Заголовок и контент на одной странице */
h3, h4 {
    page-break-after: avoid;
}
</style>

## 1. Введение
### 1.1. Назначение документа
Документ описывает требования к CRM системе для контрля и управления клиентской базой и финансами фитнес-клуба.
### 1.2. Область применения
Система будет использоваться 
- Менеджерами отдела продаж для ведения и актуализации клиентской базы, ведения истории продаж абониментов и дополнительных услуг, рассылки уведомлений клиентам.
- ГД для формирования отчетности по загруженности клуба, продажам.
### 1.3. Цель проекта
Создать единую систему для взаимотношения с клиентами и ведения отчетности
### 1.4. Глоссарий 
### Глоссарий

#### 1. Основные термины
| Термин             | Определение                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------- |
| CRM система        | Программное обеспечение для управления взаимодействием с клиентами, учета продаж и аналитики  |
| Клиент             | Физическое лицо, пользующееся услугами фитнес-клуба (действительный или потенциальный клиент) |
| Абонимент          | Пакет услуг фитнес-клуба, приобретаемый клиентом на определенный срок                         |
| Транзакция         | Операция оплаты услуги (наличными или банковской картой)                                      |
| Пропускная система | Система контроля доступа в клуб (браслеты, карты, биометрия)                                  |

#### 2. Роли пользователей
| Термин                    | Определение                                                                        |
| ------------------------- | ---------------------------------------------------------------------------------- |
| Менеджер                  | Сотрудник, отвечающий за продажи, работу с клиентами и ведение базы данных         |
| Управляющий директор (ГД) | Руководитель клуба, получающий аналитику по продажам, посещаемости и загруженности |
| Администратор             | Пользователь с правами регистрации новых сотрудников в системе                     |

#### 3. Технические термины
| Термин             | Определение                                                                            |
| ------------------ | -------------------------------------------------------------------------------------- |
| API                | Интерфейс для взаимодействия CRM с внешними системами (банк, пропускная система и др.) |
| Webhook            | Механизм автоматической отправки данных между системами в реальном времени             |
| SMTP               | Протокол для отправки email-уведомлений                                                |
| ОФД (Онлайн-касса) | Система передачи фискальных данных в налоговую службу                                  |
| Хеширование        | Защитное преобразование паролей в необратимый формат (например, MD5)                   |

#### 4. Уведомления и коммуникация
| Термин       | Определение                                                      |
| ------------ | ---------------------------------------------------------------- |
| SMS-сервис   | Внешний сервис для массовой рассылки SMS (например, SMS Aero)    |
| Telegram-bot | Автоматизированный бот для отправки уведомлений через Telegram   |
| Рассылка     | Массовая отправка сообщений клиентам (акции, напоминания и т.д.) |

#### 5. База данных и аналитика
| Термин                            | Определение                                                                 |
| --------------------------------- | --------------------------------------------------------------------------- |
| Карточка клиента                  | Профиль клиента в CRM с персональными данными, историей посещений и покупок |
| Дашборд                           | Визуализированная сводка ключевых метрик для руководства                    |
| ERD (Entity-Relationship Diagram) | Диаграмма сущностей и связей в базе данных                                  |

#### 6. Статусы
| Термин             | Определение                                                |
| ------------------ | ---------------------------------------------------------- |
| Активный клиент    | Клиент с действующим абониментом                           |
| Неактивный клиент  | Клиент без действующего абонимента (но сохраненный в базе) |
| Сотрудник на смене | Тренер или менеджер, находящийся в рабочее время в клубе   |

#### 7. Безопасность
| Термин                      | Определение                                         |
| --------------------------- | --------------------------------------------------- |
| ART (Average Response Time) | Среднее время отклика системы (требование: ≤ 1.5 с) |
| RPS (Requests Per Second)   | Количество запросов в секунду (требование: 50 RPS)  |

## 2. Общее описание системы
### 2.1. Контекст системы
Система взаимодействует с:
- менеджером по средству приложения
- ГД по средству приложения
- с мессенджера для рассылки уведомлений (Telegram-bot)
- c SMS сервисами для рассылки уведомлений (SMS-Aero)
- с почтой для рассылки уведомлений (SMTP + App Password)
- с банком через WEBhook (Сбербанк Business Online API)
- с вебсайтом через API CRM системы для предоставления информации о предоставляемых услугах
- с пропускной системой (браслеты, карты) через Ready4Sky + http/API 

### 2.2. Диаграмма контекста

```mermaid

flowchart TD
    subgraph Персонал
        MD((Управляющий директор))
        Manager((Менеджер))
    end

    subgraph Внешние системы
        Bank((Банк))
        Access_system((Пропускная систем))
    end

    subgraph Каналы уведомлений
        SMS((СМС сервис))
        Telegram((Telegram-bot))
        Email((Электронная почта))
    end

    System[[CRM система]]

    Database[[База данных]]
    WEB_site[[Вебсайт клуба]]

    Manager -->|Запрос данных клиента| System
    System -->|Отчет| Manager
    MD -->|Аналитика деятельности| System
    System -->|Дашборд| MD

    System -->|Хранение| Database
    Database -->|Загрузка| System
    System -->|Обновление контента| WEB_site

    System -->|Рассылка| SMS
    System -->|Рассылка| Telegram
    System -->|Рассылка| Email

    Bank <-->|Платежи| System
    Access_system <-->|Логи доступа| System


```
### 2.2. Бизнес требования
- обеспечить 10% от притока новых посетителей путем привлечения неактивных клиентов (email/SMS рассылки и персональные предложеня) в течение 3 месяцев после внедрения
- сократить штат менеджеров отдела продаж за 6 месяцев без снижения выручки
### 2.3. Ограничения
- оплата услуг исключительно оффлайн
- веб-сайт клуба уже в работе

## 3. Функциональные требования
UC-01: Регистрация новых работников  
Актор: ГД
Основной поток:
1. Система предоставляет форму для создания учетной записи работника (ФИО, логин, пароль, подтверждение пароля)
2. Администратор регистрирует нового пользователя
3. Система создает учетную запись, хэширует пароль
  
Альтернативный поток:  
- А1: Пароль не соответсвует требованиям  
  1. На шаге 2 система проверяет данные
  2. Система сообщает об ошибке и предоставляет требования для пароля во всплывающем окне
  3. Администратор повторно вводит пароль согласно требованиям
  4. Система создает учетную запись, хэширует пароль 
- А2: Пароли не совпадают  
  1. На шаге 2 система проверяет совпадение паролей
  2. Система сообщает об ошибке, подсвечивает поля с паролями
  3. Администратор повторно вводит пароль согласно требованиям
  4. Система создает учетную запись, хэширует пароль 
- А3: Логин уже занят
  1. На шаге 2 система проверяет данные, сравнивает введеный логин с уже существующими
  2. Система сообщает об ошибке, подсвечивает поле для логина
  3. Администратор повторно вводит иной логин
  4. Система создает учетную запись, хэширует пароль 

UC-02: Аутентификация пользователя  
Актор: Менеджер, ГД
Основной поток:  
1. Система предоставляет форму для аутентификации (логин, пароль)
2. Пользователь вводин данные
3. Система авторизует пользователя

Алтернативный поток:  
- А1: Неверный логин/пароль
  1. На шаге 2 система проверяет введенные данные
  2. Система сообщает об ошибке аутентификации
  3. Пользователь вводин данные повторно
  4. Система авторизует пользователя

UC-03: Внесение информации о новом клиент в базу данных  
Актор: Менеджер  
Основной поток: 
1. Система предоставляет раздел "Клиенты" главного меню (боковая панель)
2. Пользователь переходит в раздел "Клиенты"
3. Система предоставляет список клиентов с личными данными
4. Пользователь нажимает кнопку "Добавить клиента"
5. Система предоставляет форму для заполнения - "Карточка клиента"
6. Пользователь вносит основную информацию о клиенте (ФИО, телефон, email, соцсети) и поддтверждает создание "карточки"
7. Система сообщает об успешном внесении информации о клиенте в базу данных

Альтернативный поток:
 - А1: Клиент уже внесен в базу данных
   1. На шаге 5 система проверяет внесенные данные (Совпадение ФИО, контактных данных)
   2. Система сообщает о дублирующихся данных и просит подтвердить внесение информации 
   3. Пользователь отменяет заполнение формы
   4. Система перенесит пользователя на главное меню
 - А2: Совпадают ФИО разных клиентов
   1. На шаге 5 система проверяет внесенные данные (Совпадение ФИО, контактных данных)
   2. Система сообщает о дублирующихся данных и просит подтвердить внесение информации 
   3. Пользователь подтверждает внесение информации
   4. Система сообщает об успешном внесении информации о клиенте в базу данных

UC-04: Поиск карточки клиента  
Актор: Менеджер  
Основной поток:
1. Пользователь переходить в раздел "Клиенты" боковой панели главного меню
2. Система предоставляет поисковую строку 
3. Пользователь вводит ФИО, id или контактную информацию
4. Система предоставляет все существующие "карточки" с совпадающими данными
5. Пользоваетель переходит на необходиму "карточку"

Альтернативный поток:
 - А1: Совпадений не найдено
   1. На шаге 3 система производит поиск по указанным данным
   2. Система сообщает об ошибке поиска

UC-05: Продажа услуги  
Актор: Менеджер, клиент  
Основной поток:
1. Система предоставляет вкладку "Оплата" на боковой панели главного меню
2. Пользователь нажимает на кнопку "Новая транзакция" в разделе "Оплата"
3. Система предоставляет форму для заполнения
4. Пользователь выбирает клиента из базы данных с помощью поисковой строки 
5. Пользователь выбирает услугу и способ оплаты через терминал
6. Система связывается с терминалом
7. Клиент оплаичивает услугу через терминал
8. Терминал передает данные банку через банковский шлюз
9. Банк возвращает информацию и подтверждает оплату услуги
10. Система сохраняет чек оплаты в разделе "Продажи"
11. Система дублирует информацию о приобретенной услуге в "карточке" клиента и активирует абонимент
12. Система дублирует информацию о приобретенной услуге в разделе "Аналитика" боковой панели главного меню

Альтернативный поток: 
 - А1: Оплата наличными
   1. На шаге 5 пользователь выбирает оплату наличными
   2. Система передает информацию в онлайн кассу
   3. Пользователь вносит оплату
   4. Онлайн касса передает данные в ОФД
   5. Система сохраняет чек оплаты в разделе "Продажи"
   6. Система дублирует информацию о приобретенной услуге в "карточке" клиента и активирует абонимент
   7. Система дублирует информацию о приобретенной услуге в разделе "Аналитика" боковой панели главного меню

UC-06: Формирование графика работы тренеров  
Актор: Менеджер  
Основной поток:
1. Система предоставляет вкладку "Расписание" на боковой панели главного меню
2. Система формирует графический интерфейс в формате календаря
3. Пользователь вносит данные о рабочих часах тренерского состава для каждой даты месяца (id специалиста, рабочие часы)
4. Система дублирует информацию на сайте клуба
5. Система дублирует информацию о рабочих часах каждого спецалиста в разделе "Аналитика"

UC-07: Мониторинг деятельности клуба  
Актор: Менеджер, ГД  
Основной поток:
1. Система предоставляет раздел "Аналитика" боковой панели главного меню
2. Пользователь переходит в раздел "Аналитика"
3. Система предоставляет меню раздела с вкладками "Клиенты", "Продажи", "Сотрудники"
4. Пользователь выбирает интересующий раздел
5. Система выводит статистику в графическом или табличном формате
6. Пользователь запрашивает экспорт статистики в требуемы формат (pdf, xlsx)
7. Система экспортирует данные в указанном формате и загружает по указанной директории

UC-08: Формирование рассылок и уведомлений  
Актор: Менеджер  
Основной поток:  
1. Система предоставляет раздел "Маркетинг" на боковой панели главного меню
2. Пользователь переходит в раздел
3. Система предоставляет форму для заполнения
4. Пользователь определяет клиента для уведомления с помощью поисковой строки или группу клиентов по ключевому параметру (статус активности, тип оказываемой услуги, дата рождения и т.д.)
5. Пользователь определяет текст рассылки
6. Пользователь определяет способ отправки сообщений (email, sms, Telegram)
7. Система связывается с выбранной системой и передает текст сообщения, контактные данные пользователей
8. Система уведомляет пользователя об успешной отправки уведомлений

UC-09: Формирование перечня услуг  
Актор: Менеджер  
Основной поток:
1. Система предоставляет раздел "Услуги" боковой панели главного меню
2. Пользователь переходит в упомянутый раздел
3. Система предоставляет список оказываемых услуг + описание (стоимость оказания услуг, специалисты и т.д.) с возможностью поиска по названию
4. Пользователь нажимает кнопку добавить услугу
5. Система предоставляет форму для заполнения 
6. Пользователь заносит описание услуги и подтверждает запись информации об услуге в базу данных
7. Система сохраняет информацию об услуге
8. Система уведомляет об успешном сохранении данных

UC-10: Формирование списка сотрудников клуба  
Актор: ГД 
Основной поток:  
1. Система предоставляет раздел "Сотрудники" на боковой панели главного меню 
2. Пользователь переходит в упомянутый раздел
3. Система предоставляет список сотрудников с личными данными (ФИО, должность, специализация и т.д.)
4. Пользователь нажимает на кнопку "Добавить сотрудника"
5. Система предоставляет форму для заполнения 
6. Пользователь вносит необходимую информацию о сотруднике
7. Пользователь подтверждает запись данных
8. Система сохраняет информацию в базе данных
9. Система сообщает об успешной записи данных

Альтернативный поток:
 - А1: Сотрудник уже внесен в базу данных
   1. На шаге 7 система проверяет внесенные данные (полное совпадение комбинации ФИО, контактных данных)
   2. Система сообщает о дублирующихся данных и просит подтвердить внесение информации 
   3. Пользователь отменяет заполнение формы
   4. Система перенесит пользователя на главное меню

## 4. Нефункциональные требования
### 4.1 Производительность
NFR-1.1: ART ≤ 1.5 с  
NFR-1.2: Пропускная способность 50 RPS 
### 4.2 Совместимость
NFR-2.1: Кросс платформенная совместимость (Windows, mac OS)
### 4.3 Usability
NFR-3.1: Время выполнения базовых задач новыми пользователями ≤ 5 мин  
NFR-3.2: Количество кликов для ключевых операций ≤ 4 
NFR-3.3: Ввод данных с клавиатуры ≤ 20% от всех действий  
### 4.4 Безопасность
NFR-4.1: Парольная сложность: 8+ chars, (1 заглавная, 1 цифра минимум) 
NFR-4.2: Хеширование паролей и данных пользователей (bcrypt)

## 5. Концептуальная диаграмма БД (ERD)
```mermaid
erDiagram
    direction LR

    SUBSCRIPTION }o--|| PURCHASE : ""
    SUBSCRIPTION }o--|| CUSTOMER_PROFILE : ""
    SUBSCRIPTION }o--|| SERVICE : ""

    CUSTOMER_PROFILE ||--|| CLIENT : ""
    SERVICE }o--|| SCHEDULE : ""
    SERVICE ||--o{ EMPLOYEE : ""

    CLIENT }o--o{ MAILING : ""
    CLIENT ||--o{ VISITING : ""
    MAILING }o--|| METHOD : ""
```

## 6. Логическая диаграмма БД
```mermaid
classDiagram
    direction LR

    class SUBSCRIPTION{
        subscription_id: PK, integer
        purchase_id: FK, integer
        customer_profile_id: FK, integer
        service_id: FK, integer
        first_day: date
        last_day: date
        created_at: timestamp
    }

    class PURCHASE{
        purchase_id: PK, integer
        payment_date: date
        sum_price: decimal(10,2)
        created_at: timestamp
    }

    class CUSTOMER_PROFILE{
        customer_profile: PK, integer
        client_id: FK, integer
        created_at: timestamp
        updated_at: timestamp
    }

    class SERVICE{
        service_id: PK, integer
        employee_id: FK, integer
        service_name: varchar(50)
        description: text
        recomindateions: text
        service_price: decimal(10,2)
        duration_minutes: integer
        max_participants: integer
        created_at: timestamp
        updated_at: timestamp
    }

    class SCHEDULE {
        schedule_id: PK, integer
        service_id: FK, integer
        date: date
        day_off: boolean
        created_at: timestamp
        updated_at: timestamp
    }

    class EMPLOYEE{
        employee_id: PK, integer
        employee_status: boolean
        employee_phone_number: varchar(20)
        employee_email: varchar(50)
        employee_social_media: varchar(50)
        employee_first_name: varchar(50)
        employee_last_name: varchar(50)
        employee_patronymic: varchar(50)
        employee_specialization: varchar(50)
        session_price: decimal(10,2)
        employee_login_hash: varchar(255)
        employee_password_hash: varchar(255)
        created_at: timestamp
        updated_at: timestamp
    }

    class EMPLOYEE_STATUS{
        employee_status_id: PK, integer
        employee_status: varchar(50)
    }

    class CLIENT{
        client_id: PK, integer
        client_status_id: FK, integer
        client_phone_number: varchar(20)
        client_email: varchar(50)
        client_social_media: varchar(50)
        client_first_name: varchar(50)
        client_last_name: varchar(50)
        client_patronymic: varchar(50)
        birth_date: date
        preferences: varchar(50)
        Health_restrictions: text
        created_at: timestamp
        updated_at: timestamp
    }

    class CLIENT_STATUS{
        client_status_id: PK, integer
        client_status: varchar(50)
    }

    class VISITING{
        visiting_id: PK,integer
        client_id: FK, integer
        chek_in: timestamp
        chek_out: timestamp
    }

    class CLIENT_MAILING{
        client_mailing: PK, integer
        mailing_id: FK, integer
        client_id: FK, integer
        created_at: timestamp
        updated_at: timestamp
    }

    class MAILING{
        mailing_id: PK, integer
        method_id: FK, integer
        message_text: text
        created_at: timestamp
        updated_at: timestamp
    }

    class METHOD{
        method_id: PK, integer
        method_name: varchar(50)
        created_at: timestamp
        updated_at: timestamp
    }

    SUBSCRIPTION "N" -- "1" PURCHASE : ""
    SUBSCRIPTION "N" -- "1" CUSTOMER_PROFILE : ""
    SUBSCRIPTION "N" -- "1" SERVICE : ""

    CUSTOMER_PROFILE "1" -- "1" CLIENT : ""
    SERVICE "N" -- "1" SCHEDULE : ""
    SERVICE "1" -- "N" EMPLOYEE : ""

    CLIENT "1" -- "N" CLIENT_MAILING : ""
    CLIENT "1" -- "N" VISITING : ""
    CLIENT_MAILING "N" -- "1" MAILING : ""
    MAILING "N" -- "1" METHOD : ""

    CLIENT_STATUS "1" -- "N" CLIENT: ""
    EMPLOYEE_STATUS "1" -- "N" EMPLOYEE: ""
```

## 7. UML
### 7.1. Диаграмма классов

```plantuml
@startuml
left to right direction

    class User{
        +id: UUID
        +phoneNumber: String
        +email: String
        +socialMedia: String
        +firstName: String
        +lastName: String
        +patronymic: String
        -loginHash: String
        -passwordHash: String
        -logIn(password: String, login: String)::Void
    }

    class ManagerMenu{
        +getClient():: Void
        -getStatistics():: Void
        +getSchedule():: Void
        +getService():: Void
        +getMailing():: Void
        +getPayment():: Void
    }

    class MdMenu{
        -getStatistics():: Void
        +getSchedule():: Void
        +getEmployee():: Void
    }

    class Client{
        +id: UUID
        -clientStatus: Enum
        +phoneNumber: String
        +email: String
        +socialMedia: String
        +firstName: String
        +lastName: String
        +patronymic: String
        +birthDate: Date
        +preferences: String
        +healthRestrictions: String
        -clientsFilter(preferences: String = null, clientStatus: Enum = null, healthRestrictions: String):: List~Client~
        -searchClients(query: String)::List~Client~
        -addClient(clientData: ClientDTO):: UUID
        -updateClient(id: UUID, updateInfo: ClientDTO):: void
    }

    enum ClientStatus{
        Active
        NotActive
    }

    class ClientDTO{
        +phoneNumber: String
        +email: String = null
        +socialMedia: String = null
        +firstName: String
        +lastName: String
        +patronymic: String
        +birthDate: Date = null
        +preferences: String = null
        +healthRestrictions: String = null
    }

    class Statistics{
        -getClientStat(id: UUID):: List~StatisticItem~
        -getEmployeeStat(id: UUID):: List~StatisticItem~
        -getServiceStat(id: UUID):: List~StatisticItem~
        -downloadStats(periodStart: LocalDate, periodDate: LocalDate, reportType: StatsReportType): ByteArray
    }

    enum StatsReportType{
        PDF
        EXCEL
        CSV
    }

    class Schedule{
        +serviceId: UUID
        +date: Date
        +day_off: Boolean
        -updateSchedule(serviceId: UIID, date: Date, dayOff: Enum):: void
    }

    class Service{
        +serviceId: UUID
        +employeeId: UUID
        +serviceName: String
        +description: String
        +recommendations: String
        +servicePrice: Integer
        +durationMinutes: Integer
        +maxParticipants: Integer
        -getService(query: String):: List~Service~
        -addService(serviceData: ServiceDTO):: UUID
        -updateService(serviceId: UUID, serviceData: ServiceDTO):: void
    }

    class ServiceDTO{
        +employeeId: UUID 
        +serviceName: String
        +description: String
        +recommendations: String = null
        +servicePrice: Double
        +durationMinutes: Integer = null
        +maxParticipants: Integer = null
    }

    class Mailing{
        +mailingId: UUID
        +methodId: UUID
        -messageText: text
        +createdAt: Date
        -createMailing(clients: List~Client~, text: String, methodId: Integer):: UUID
    }

    class Purchase{
        +purchaseId: Integer
        +paymentDate: Date
        +sumPrice: Double
        +method: Enum
        -getPaymentInfo(dateFrom: Date = null, dateTo: Date = null, client: UUID = null, service: String = null):: List~Purchase~
        -payment( method: Enum, sumPrice: Double, client: UUID, paymentDate: Date):: UUID
    }

    enum Method{
        byCard
        inCash
    }

    class Employee{
        +employeeId: UUID
        -employeeStatus: Enum
        +employeePhoneNumber: String
        +employeeEmail: String
        +employeeSocialMedia: String
        +employeeFirstName: String
        +employeeLastName: String
        +employeePatronymic: String
        +employeeSpecialization: String
        +sessionPrice: Double
        -employeePassword_hash: String
        -searchEmployee(query: String)::List~Employee~
        -addEmployee(EmployeeData: EmployeeDTO):: UUID
        -updateEmployee(id: UUID, updateInfo: EmployeeDTO):: void
    }

    enum EmployeeStatus{
        onDuty
        offDuty
    }

    class EmployeeDTO{
        +phoneNumber: String
        +email: String = null
        +socialMedia: String = null
        +firstName: String
        +lastName: String
        +patronymic: String
        +employeeSpecialization: String = null
        +sessionPrice: Double
        -employeePasswordHash: String
    }

    User "N" --> "1" ManagerMenu
    User "1" --> "1" MdMenu

    ManagerMenu o--> Client
    ManagerMenu o--> Statistics
    ManagerMenu o--> Schedule
    ManagerMenu o--> Service
    ManagerMenu o--> Mailing
    ManagerMenu o--> Purchase

    MdMenu o--> Statistics
    MdMenu o--> Schedule
    MdMenu o--> Employee

    Client ..> Statistics
    Employee ..> Statistics
    Service ..> Statistics
    Mailing ..> Statistics
    Purchase ..> Statistics

    EmployeeDTO <.. Employee
    ServiceDTO <.. Service
    ClientDTO <.. Client
    PaymentService <.. Purchase

    EmployeeStatus "1" <-- "N" Employee
    Method "1" <-- "N" Purchase
    StatsReportType "1" <-- "N" Statistics
    ClientStatus "1" <-- "N" Client

@enduml
```

### 7.2. Диаграмма последовательности (Sequence Diagram)
   
```plantuml
@startuml

title UC-02 А1: Ошибка аутентификации (Неверные данные)

actor Пользователь as User
participant "Система" as System
participant "База данных" as Database

User -> System: Вводит логин и пароль
System -> Database: Проверяет соответствие логина и хэша пароля
Database --> System: Данные неверны
System --> User: "Ошибка: неверный логин или пароль"

alt Повторная попытка
    User -> System: Вводит данные повторно
    System -> Database: Проверяет снова
    Database --> System: Данные верны
    System --> User: "Аутентификация успешна"
end
@enduml
```

```plantuml
@startuml

title UC-03 A1: Внесение клиента в базу данных

actor Пользователь as USER

participant Система as SYSTEM
participant База_данных as DB

USER -> SYSTEM: Пользователь нажимает кнопку "Добавить клиента"
SYSTEM --> USER: Система предоставляет форму для заполнения - "Карточка клиента"
USER -> SYSTEM: Пользователь вносит основную информацию о клиенте и поддтверждает создание "карточки"
SYSTEM -> DB: Проверяет данные на совпадение
DB --> SYSTEM: Клиент уже числится в БД
SYSTEM --> USER: Ошибка создания "карточки клиента"

alt Повторная попытка
    USER -> SYSTEM: Пользователь повторно вводит данные
    SYSTEM -> DB: Проверяет данные на совпадение
    DB --> SYSTEM: Данные не дублируются
    SYSTEM -> DB: Запись данных
    DB --> SYSTEM: Данные внесены
    SYSTEM --> USER: "Карточка клиент" создана успешно
end
@enduml
```

```plantuml
@startuml

title UC-05: Оплата услуги (Безналичный расчет)

actor Менеджер as User
actor Клиент as Client

participant Система as System
participant Платежный_терминал as Terminal
participant Банковский_шлюз as Gateway
participant ОФД as OFD
participant База_данных as DB

User -> System: Выбирает услугу и клиента
System --> User: Показывает сумму к оплате
User -> System: Инициирует оплату
System -> Terminal: Передает сумму
Terminal --> Client: Запрашивает оплату (карта)
Client -> Terminal: Подтверждает платеж
Terminal -> Gateway: Отправляет запрос в банк
Gateway --> Terminal: Подтверждение оплаты
Terminal -> OFD: Фискализирует чек
Terminal --> System: Успех оплаты
System -> DB: Сохраняет транзакцию
System --> User: "Оплата завершена"

@enduml
```

### 7.3. USE-CASE DIAGRAM

```plantuml

@startuml CRM_Registration_UseCase

left to right direction  
actor "Администратор" as Admin

usecase "Регистрация нового работника" as UC1
usecase "Пароль не соответствует требованиям" as A1
usecase "Пароли не совпадают" as A2
usecase "Логин уже занят" as A3

Admin --> UC1
UC1 <|-- A1
UC1 <|-- A2
UC1 <|-- A3

note right of UC1
  **Основной поток:**
  1. Система показывает форму.
  2. Администратор вводит данные.
  3. Система создает учетную запись.
end note

note right of A1
  **Альтернативный поток A1:**
  1. Пароль слишком простой.
  2. Система показывает ошибку.
  3. Администратор исправляет.
end note

note right of A2
  **Альтернативный поток A2:**
  1. Пароли не совпадают.
  2. Система подсвечивает поля.
  3. Администратор вводит заново.
end note

note right of A3
  **Альтернативный поток A3:**
  1. Логин занят.
  2. Система сообщает об ошибке.
  3. Администратор выбирает другой.
end note

@enduml

```

### 7.4. Диаграмма активности

```plantuml

title Формирование списка сотрудников клуба

:Пользователь нажимает на кнопку "Добавить сотрудника";
:Система предоставляет форму для заполнения;
:Пользователь вносит необходимую информацию о сотруднике;
:Система проверяет данные на совпадение;
if (Клиент зарегистрирован?) then (Нет)
    :Пользователь подтверждает запись данных;
    :Система сохраняет информацию в базе данных;
    :Система сообщает об успешной записи данных;
    :Система переносит пользователя на главное меню;
    else (Да)
    :Система сообщает о дублирующихся данных;
    :Пользователь отменяет заполение формы;
    :Система переносит пользователя на главное меню;
end if
stop

```

## 8. Open-API

```yaml

openapi: 3.0.0

info:
    title: CRM API
    version: 1.0.0
    description:

paths:
    /clients:
        get:
            summary: "Получить список клиентов с пагинацией на главной странице раздела"
            tags: [Clients]
            parameters:
              - name: page
                in: query
                schema:
                    type: integer
                    minimum: 1
                    default: 1
              - name: limit
                in: query
                schema:
                    type: integer
                    maximum: 100
                    default: 20
            responses:
                200:
                    description: "Список клиентов с метаданными"
                    content:
                        application/json:
                            schema:
                                type: object
                                properties:
                                    data:
                                        type: array
                                        items:
                                            $ref:
                                    meta:
                                        type: object
                                        properties:
                                            totalItems:
                                                type: integer
                                            totalPages:
                                                type: integer
    /clients/search:
        get:
            summary: "Поиск клиентов"
            tags: [Clients]
            parameters:
              - name: query
                in: query
                required: true
                schema:
                    type: string
                    minLength: 3
                    descriptiion: "Поиск по ФИО, контактным данным, абонементу"
              - name: exactMatch
                in: query
                schema:
                    type: boolean
                    default: false
            responses:
                200:
                    description: "Список клиентов"
                    content:
                        application/json:
                            schema:
                                type: array
                                items:
                                    $ref:
    /clients/register:
        post:
            summary: "Регистрация клиента"
            tags: [Clients]
            requestBody:
                content:
                    application/json:
                        schema:
                            $ref:
            responses:
                201:
                    description: "Клиент создан"
                    content:
                        application/json:
                            schema:
                                $ref:
    /clients/{clientsId}:
        patch:
            summary: "Обновить данные клиента"
            tags: [Clients]
            parameters:
              - name: clientId
                in: path
                required: true
                schema:
                    type: string
                    format: uiid
            requestBody:
                required: true
                content:
                    application/json:
                        schema:
                            type: object
                            properties:
                                email:
                                    type: string
                                    format: email
                            required: [email]
            responses:
                200:
                    description: "email обновлен"
                    content:
                        application/json:
                            schema:
                                $ref:
    clients/{clientId}:
        delete:
            summary: "Удалить данные о клиенте"
            tags: [Client]               
            parameters:
              - name: clientId
                in: path
                required: true
                schema:
                    type: string
                    format: uiid
            responses:
                204:
                    description: "Данные о клиенте удалены"
                404:
                    description: "Клиент не найден"
    /employee:
        get:
            summary: "Получить список сотрудников на главной странице раздела"
            tags: [Employee]
            responses:
                200:
                    description: "Список клиентов"
                    content:
                        applecation/json:
                            schema:
                                type: array
                                items:
                                    $ref:
    /employee/register:
        post:
            summary: "Регистрация сотрудника"
            tags: [Employee]
            requestBody:
                content:
                    application/json:
                        schema:
                            $ref:
            responses:
                201:
                    description: "Учетная запись создана"
                    content:
                        application/json:
                            schema:
                                $ref:
    /employee/{employeeId}:
        patch:
            summmary: "Обновить данные клиента"
            tags: [Employee]
            parameters:
              - name: clientId
                in: path
                required: true
                schema:
                    type: string
                    format: uiid 
            requestBody:
                content:
                    application/json:
                        schema:
                            type: object
                                properties:
                                    email:
                                        type: string
                                        format: email
                                    required: [email]
            responses:
                200:
                    description: "email сотрудника обновлен"
                    content:
                        application/json:
                            schema:
                                $ref:
    /employee/{employeeId}:
        delete:
            summary:
            tags: [Employee]
            parameters:
              - name: employeeId
                in: path
                required: true
                schema:
                    type: string
                    format: uiid
            responses:
                204:
                    description: "Данные о сотруднике удалены"
                404:
                    description: "Сотрудник не найден"
    /mailing/send-sms:
        post:
            summary: "Создать рассылку SMS"
            tags: [Mailing]
            requestBody:
                content:
                    application/json:
                        schema:
                            type: object
                                properties:
                                    phone:
                                        type: array
                                        pattern: '^\+7\d{10}$'
                                    required: [phone]
            responses:
                200:
                    description: "Расслыка отправлена"
                    content:
                        application/json:
                            schema:
                                type: object
                                    properties:
                                        success:
                                            type: boolean
```
