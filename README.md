# **ПРОГРАММНОЕ СРЕДСТВО АВТОМАТИЗАЦИИ ПРОЦЕССОВ ПО ТЕХНИЧЕСКОМУ ОБСЛУЖИВАНИЮ И РЕМОНТУ В СФЕРЕ ЖИЛИЩНО-КОММУНАЛЬНОГО ХОЗЯЙСТВА**

Проект представляет собой разработку программного средства, предназначенного для автоматизации процессов технического обслуживания и ремонта 
в сфере жилищно-коммунального хозяйства. Его основная цель — повысить эффективность работы управляющих организаций, 
улучшить качество предоставляемых услуг населению и сократить затраты за счёт цифровизации ключевых операций.
Система позволяет принимать и обрабатывать заявки от жильцов, формировать графики профилактических работ, 
назначать исполнителей и отслеживать выполнение задач. Также предусмотрены функции сбора статистики, формирования отчётности, 
уведомления пользователей о статусе заявок и возможность обратной связи. Благодаря мобильному доступу сотрудники могут работать с системой в реальном времени, 
находясь на выезде.

Ссылки на репозитории:
КЛИЕНТ: https://github.com/Uklaonuka/refactoringClient
СЕРВЕР: https://github.com/Uklaonuka/refactoringServer


## **Содержание**

1. [Архитектура](#Архитектура)
	1. [C4-модель](#C4-модель)
	2. [Схема данных](#Схема_данных)
2. [Функциональные возможности](#Функциональные_возможности)
	1. [Диаграмма вариантов использования(#Диаграмма_вариантов_использования)]
	2. [User-flow диаграммы](#User-flow_диаграммы)
3. [Детали реализации](#Детали_реализации)
	1. [UML-диаграммы](#UML-диаграммы)
	2. [Спецификация API](#Спецификация_API)
	3. [Безопасность](#Безопасность)
	4. [Оценка качества кода](#Оценка_качества_кода)
4. [Тестирование](#Тестирование)
	1. [Unit-тесты](#Unit-тесты)
	2. [Интеграционные тесты](#Интеграционные_тесты)
5. [Установка и  запуск](#installation)
	1. [Манифесты для сборки docker образов](#Манифесты_для_сборки_docker_образов)
	2. [Манифесты для развертывания k8s кластера](#Манифесты_для_развертывания_k8s_кластера)
6. [Лицензия](#Лицензия)
7. [Контакты](#Контакты)

---
## **Архитектура**

### C4-модель

В разрабатываемом программном средстве реализуется 4 роли пользователя: Сотрудник, Клиент, Руководитель, Технический специалист. Клиентская часть представлена единым web-приложением. Контекстная диаграмма представленна ниже.
<img width="944" height="471" alt="С4-1" src="https://github.com/user-attachments/assets/6f124b3f-8640-42ed-a760-9b537ecdde7e" />
На компонентном уровне програбное средство состоит из Базы данных и сервера, связанного с клиентской частью. Помиму того, задействованы вспомогательные сторонние сервисы: SMTP для отправки электронных сообщений и PowerBI для сбора и анализа данных. Компонентный уровень представлен ниже.
<img width="1189" height="1061" alt="С4-2" src="https://github.com/user-attachments/assets/8e7b0493-dbe8-4f65-a5d7-2cb7a5a0432d" />
Контейнерный уровень описывает разделение сервера на отдельные контейнеры: Контейнер основной логики, Контейнер для работы с файлами, Контейнер для работы с данными пользователей, Контейнер отправки сообщений. База данных также декомпозирована на базу клиентов, базу файлов, основную базу данных. Контейнерный уровень представлен ниже.
<img width="1442" height="929" alt="С4-3" src="https://github.com/user-attachments/assets/09969049-86e3-487a-94f5-90c3e5fd3600" />
Самый низкий уровнь в нотации C4 - уровень кода. Он отображает диаграмму классов, реализующихся в программном средстве. Уровень кода представлен ниже.
<img width="691" height="804" alt="С4-4" src="https://github.com/user-attachments/assets/16c0eb11-78ae-4e2f-a3f3-9caf531dbe4a" />


### Схема данных

Сущности, конфигурирующие в предметной области представлены ниже:
User – сущность пользователя программного средства.
Application – сущность заявки на оказание услуги и/или ремонта.
Service – сущность услуги.
Bill – сущность счета.
Appeal – сущность обращения (сообщения) пользователя.
Survey – сущность опроса пользователя.
Post – сущность поста (объявления).
Appointment – сущность приема (посещения) для записи пользователя на прием к сотруднику.
Object – сущность жилого объекта, обслуживающегося предприятием.
Document – сущность документа.
Employee – сущность сотрудника.

<img width="964" height="630" alt="image" src="https://github.com/user-attachments/assets/9cc30e2b-ab36-4748-b2c5-b2c420256825" />

В физической модели данных созданы таблицы в соответствии с сущностями, в которые внесены соответствующие поля. Ранее существовавшие связи в логической модели были отображены на конкретных полях таблиц. Такие поля приобрели тип внешнего ключа.
В таблице «Appointment», «Appeal», «Object», «Bill», «Application» поля user_id являются внешними ключами от поля id_user в таблице «User».
В таблице «Bill» поле service_id является внешним ключом от поля id_service в таблице «Service».
В таблице «Application» поле service_id является внешним ключом от поля id_service в таблице «Service», поле employee_id является внешним ключом от поля id_employee в таблице «Employee».
База данных соответствует третьей нормальной форме (3НФ), так как каждая таблица имеет первичный ключ, все неключевые атрибуты зависят только от этого ключа и не зависят транзитивно друг от друга.
Ниже приведен код генерации полученной базы данных.

CREATE SCHEMA IF NOT EXISTS mydb DEFAULT CHARACTER SET utf8;
USE mydb;
CREATE TABLE IF NOT EXISTS mydb.User (
  ID_user INT NOT NULL AUTO_INCREMENT,
  Identify VARCHAR(14) NOT NULL,
  First_name VARCHAR(255) NOT NULL,
  Last_name VARCHAR(255) NOT NULL,
  Surname VARCHAR(255) NOT NULL,
  Birthday DATE NOT NULL,
  Email VARCHAR(45) NOT NULL,
  Adress VARCHAR(255) NOT NULL,
  Contact VARCHAR(45) NOT NULL,
  Login VARCHAR(255) NOT NULL,
  Password VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_user))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Appeal (
  ID_appeal INT NOT NULL AUTO_INCREMENT,
  Date DATE NOT NULL,
  Text TEXT(900) NOT NULL,
  User_ID INT NOT NULL,
  Status TINYINT(1) NOT NULL,
  PRIMARY KEY (ID_appeal),
  INDEX user-appeal_idx (User_ID ASC) VISIBLE,
  CONSTRAINT user-appeal
    FOREIGN KEY (User_ID)
    REFERENCES mydb.User (ID_user)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Post (
  ID_post INT NOT NULL AUTO_INCREMENT,
  Date DATE NOT NULL,
  Title VARCHAR(255) NOT NULL,
  Text TEXT(1500) NOT NULL,
  Image VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_post))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Appointment (
  ID_appointment INT NOT NULL AUTO_INCREMENT,
  DateTime DATETIME NOT NULL,
  User_ID INT NOT NULL,
  Employee_ID INT NOT NULL,
  PRIMARY KEY (ID_appointment),
  INDEX appointment-user_idx (User_ID ASC) VISIBLE,
  CONSTRAINT appointment-user
    FOREIGN KEY (User_ID)
    REFERENCES mydb.User (ID_user)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Document (
  ID_document INT NOT NULL AUTO_INCREMENT,
  Date DATE NOT NULL,
  Title VARCHAR(255) NOT NULL,
  Discription VARCHAR(255) NOT NULL,
  Link VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_document))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Survey (
  ID_survey INT NOT NULL AUTO_INCREMENT,
  Date_close DATE NOT NULL,
  Title VARCHAR(255) NOT NULL,
  Text TEXT(800) NOT NULL,
  Link VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_survey))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Service (
  ID_service INT NOT NULL AUTO_INCREMENT,
  Title VARCHAR(255) NOT NULL,
  Deadline INT NOT NULL,
  Price DOUBLE NOT NULL,
  Department VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_service))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Employee (
  ID_employee INT NOT NULL AUTO_INCREMENT,
  First_name VARCHAR(255) NOT NULL,
  Last_name VARCHAR(225) NOT NULL,
  Surname VARCHAR(225) NOT NULL,
  Birthday DATE NOT NULL,
  Email VARCHAR(255) NOT NULL,
  Adress VARCHAR(255) NOT NULL,
  Contact VARCHAR(45) NOT NULL,
  Department VARCHAR(255) NOT NULL,
  Login VARCHAR(255) NOT NULL,
  Password VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_employee))
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Bill (
  ID_bill INT NOT NULL AUTO_INCREMENT,
  User_ID INT NOT NULL,
  Sevice_ID INT NOT NULL,
  Price DOUBLE NOT NULL,
  Status TINYINT(1) NOT NULL,
  Date DATE NOT NULL,
  Link VARCHAR(255) NOT NULL,
  PRIMARY KEY (ID_bill),
  INDEX Bill-user_idx (User_ID ASC) VISIBLE,
  INDEX Bill-service_idx (Sevice_ID ASC) VISIBLE,
  CONSTRAINT Bill-user
    FOREIGN KEY (User_ID)
    REFERENCES mydb.User (ID_user)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT Bill-service
    FOREIGN KEY (Sevice_ID)
    REFERENCES mydb.Service (ID_service)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Application (
  ID_application INT NOT NULL AUTO_INCREMENT,
  User_ID INT NOT NULL,
  Employee_ID INT NOT NULL,
  Service_ID INT NOT NULL,
  Bill_ID INT NOT NULL,
  Date DATE NOT NULL,
  Status TINYINT(1) NOT NULL,
  PRIMARY KEY (ID_application),
  INDEX Application-user_idx (User_ID ASC) VISIBLE,
  INDEX Application_idx (Service_ID ASC) VISIBLE,
  INDEX Application-employee_idx (Employee_ID ASC) VISIBLE,
  CONSTRAINT Application-user
    FOREIGN KEY (User_ID)
    REFERENCES mydb.User (ID_user)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT Application-service
    FOREIGN KEY (Service_ID)
    REFERENCES mydb.Service (ID_service)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT Application-employee
    FOREIGN KEY (Employee_ID)
    REFERENCES mydb.Employee (ID_employee)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

CREATE TABLE IF NOT EXISTS mydb.Object (
  Adres VARCHAR(255) NOT NULL,
  Floor INT NOT NULL,
  Floors INT NOT NULL,
  Rooms INT NOT NULL,
  Common_area DOUBLE NOT NULL,
  Living_area DOUBLE NOT NULL,
  Occupancy INT NOT NULL,
  Drainage TINYINT(1) NOT NULL,
  Water_supply TINYINT(1) NOT NULL,
  Gas_supply TINYINT(1) NOT NULL,
  Garbage TINYINT(1) NOT NULL,
  Water_heating TINYINT(1) NOT NULL,
  User_ID INT NOT NULL,
  PRIMARY KEY (Adres),
  INDEX Object-user_idx (User_ID ASC) VISIBLE,
  CONSTRAINT Object-user
    FOREIGN KEY (User_ID)
    REFERENCES mydb.User (ID_user)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

---

## **Функциональные возможности**

### Диаграмма вариантов использования

<img width="567" height="690" alt="image" src="https://github.com/user-attachments/assets/14ea5c4a-e65d-40e2-9f51-67b6d19c0bde" />

### User-flow диаграммы

В соответствии с темой, выбранной для практического зянятия и пользовательскими требованиями в программном средстве имеет место быть реализация 3 ролей пользователей:
– сотрудник;
– клиент;
– системный менеджер.
На рисунке представлена user-flow диаграмма, отображающая логику взаимодействия роли «Системный менеджер» с графическим интерфейсом.
<img width="921" height="725" alt="image" src="https://github.com/user-attachments/assets/ac31daad-d4c5-4c2c-aeaf-5a26f81f9e05" />

взаимодействия роли «Системный менеджер» с графическим интерфейсом

Для роли «Системный менеджер» ключевыми бизнес-процессами являются процесс управления заявками и процесс изменения тарифов. User-flow диаграммы для  данных бизнес-процессов представлены на рисунках соответственно.

<img width="615" height="393" alt="image" src="https://github.com/user-attachments/assets/cd3e6960-db36-4d25-a534-cabbabad1ec5" />
<img width="617" height="259" alt="image" src="https://github.com/user-attachments/assets/178b3021-4208-45a5-9869-6d1dfb010de5" />


На рисунке представлена user-flow диаграмма, отображающая логику взаимодействия роли «Клиент» с графическим интерфейсом.
<img width="974" height="752" alt="image" src="https://github.com/user-attachments/assets/15f3f30f-52e1-442a-a90e-1f1a3dcc3eed" />
 
взаимодействия роли «Клиент» с графическим интерфейсом

Для роли «Клиент» ключевыми бизнес-процессами являются процесс формирования заявки и процесс оплаты счета. User-flow диаграммы для  данных бизнес-процессов представлены на рисунках соответственно.
<img width="628" height="302" alt="image" src="https://github.com/user-attachments/assets/ac5ad7f0-101a-48a2-a843-4e13a6db26d3" />

<img width="649" height="527" alt="image" src="https://github.com/user-attachments/assets/57749f47-0fe7-47c6-98d6-3c9a8f987d83" />

На рисунке представлена user-flow диаграмма, отображающая логику взаимодействия роли «Сотрудник» с графическим интерфейсом.
<img width="925" height="675" alt="image" src="https://github.com/user-attachments/assets/fc2cb7b0-2c93-4faa-967f-e25108e03389" />

взаимодействия роли «Сотрудник» с графическим интерфейсом
Для роли «Сотрудник» ключевым бизнес-процессом является процесс закрытия заявки. User-flow диаграммы для  данного бизнес-процесса представлена на рисунке.

<img width="607" height="269" alt="image" src="https://github.com/user-attachments/assets/363bab97-746d-487d-bbcf-f5e9a6d50d44" />



---

## **Детали реализации**

### UML-диаграммы

Контекстная диаграмма представлена на рисунке.
<img width="614" height="284" alt="image" src="https://github.com/user-attachments/assets/409335fa-b49e-4349-890d-7977b161b30e" />

Контекстная диаграмма отображает взаимодействие пользователей (сотрудника, администратора и клиента) через программное средство.
Представленная на рисунке диаграмма классов является одним из инструментов, используемых для визуализации структуры программы.
<img width="897" height="705" alt="image" src="https://github.com/user-attachments/assets/5357a526-5254-4a2f-9520-ba0886e08fb9" />

Диаграмма классов представляет зависимость между классами в программном средстве. Такие классы, как «Appoinment», «Appeal», «Object», «Bill», «Application» имеют зависимость «Один ко многим» с классом «User». Классы «Service», «User», «Application», «Document», «Post» имеют возможность использовать интерфейсы «Menage» и «Search».
На рисунке представлена диаграмма пакетов. 
<img width="574" height="636" alt="image" src="https://github.com/user-attachments/assets/be69b4c1-d85f-49d7-a760-e437b874e352" />

Диаграмма пакетов представляет собой структуру распределения данных классов в виде пакетов. В пакете, который используется для хранения пользовательского интерфейса сожержаться дочерние пакеты, такие как «Views» и «Servlets». В пакете, который используется для хранения кода для функциональности программного средства содержаться такие пакеты, как «Entities», «Model». В пакете, который используется для хранения классов, отвечающих за подключение к базе данных, содержится дочерний пакет «DataBaseConnection».
Диаграммы размещения используются для моделирования статического представления системы с точки зрения размещения. В основном под этим понимается моделирование топологии аппаратных средств, на которых работает система. По существу, диаграммы размещения – это просто диаграммы классов, сосредоточенные на системных узлах. На рисунке представлена диаграмма пакетов.
<img width="943" height="618" alt="image" src="https://github.com/user-attachments/assets/cc740295-ee83-4707-8160-9678b6f40724" />

Диаграмма размещения описывает систему хранению программного средства от модели компьютера до описания файлов, в котрых хранится основная информация для полноценной работы программного средства. В нём описаны очновные компоненты «Service», «Application», «User», «Employee», «Bill», «Object», «Appeal», «Appointment». И оcновные схемы, которые хранятся в базе данных: «Service», «Application», «User», «Employee», «Bill», «Object», «Appeal», «Appointment».
Моделирование поведения объектов программной системы может быть представлено при помощи диаграмм, раскрывающих динамические аспекты: диаграммы деятельности, диаграмму состояний и др. На рисунке представлена деятельности, отображающая процесс формирования заявки на услугу.
<img width="705" height="833" alt="image" src="https://github.com/user-attachments/assets/10818d13-2dff-473d-90c7-d179f735dc33" />

Описание модели поведения сложных объектов программной системы для процесса формирования заявки на оказание услуги.
Клиент:
1 Проходит авторизацию.
2 Выбирает необходимую услугу.
3 Заполнят данные для оказания услуги.
4 Заказывает услугу.
5 Проводит оплату услуги.
6 Если оплата проходит успешно, то формируется заявка на оказание услуги, которая передается администратору для обработки.
Администратор:
1 Выбирает заявку и проверяет данные клиента.
2 Подтверждает заявку на исполнение.
3 Если заявка подтверждена, то назначается исполняющий работник и формируется задание.
Работник: 
1 Выбирает задание.
2 Если задание выполнено, то оставляет отметку о его выполнении. 
Данное описание позволяет отобразить последовательность действий клиента, администратора и работника в процессе формирования заявки на оказание услуги.
На рисунке представлена диаграмма состояний в сущности «Услуга» при заказе услуги клиентом.
<img width="806" height="606" alt="image" src="https://github.com/user-attachments/assets/8e7fa830-ee57-4e8a-9e44-7be73c5b7879" />

Описание модели поведения сложных объектов программной системы для диаграммы состояний.
1 Клиент переходит на страницу «Услуги».
2 Отображается перечень всех доступных услуг.
3 Клиент выбирает услугу и заполняет форму заказа.
4 Если клиент ввел данные неверно, то он получает сообщение об ошибке, иначе, происходит активация кнопки «Заказать».
5 Если клиент, нажал кнопку «Отменить», то он переходит на страницу «Услуги», иначе, он получает сообщение об успешном заказе (при этом происходит формирование счета на оказание услуги).
Диаграмма состояний – это полезный инструмент, который помогает наглядно показать различные состояния программного средства и отобразить его изменения.
Моделирование взаимодействия объектов программной системы может быть представлено с использованием диаграммы последовательности. На рисунке отображена диаграмма последовательности.
<img width="742" height="589" alt="image" src="https://github.com/user-attachments/assets/992464f2-5d31-4acc-9043-3050d774f384" />

Диаграмма последовательности отражает временные последовательности событий и обмен сообщениями между объектами.
Диаграмма компонентов представлена на рисунке.
<img width="854" height="751" alt="image" src="https://github.com/user-attachments/assets/3fb7f0b4-c40e-47d1-97f6-6c77c8a25916" />

С помощью диаграммы компонентов можно описать статическое представление дизайна системы.


### Спецификация API

Представить описание реализованных функциональных возможностей ПС с использованием Open API (можно представить либо полный файл спецификации, либо ссылку на него)

### Безопасность

Описать подходы, использованные для обеспечения безопасности, включая описание процессов аутентификации и авторизации с примерами кода из репозитория сервера

### Оценка качества кода

Используя показатели качества и метрики кода, оценить его качество

---

## **Тестирование**

### Unit-тесты

Представить код тестов для пяти методов и его пояснение

### Интеграционные тесты

Представить код тестов и его пояснение

---

## **Установка и  запуск**

### Манифесты для сборки docker образов

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

### Манифесты для развертывания k8s кластера

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

---

## **Лицензия**

Этот проект лицензирован по лицензии MIT - подробности представлены в файле [[License.md|LICENSE.md]]

---

## **Контакты**

Автор: Филон Андрей Юрьевич
Email: kirisyuk151001@gmail.com

