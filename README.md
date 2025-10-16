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

Диаграмма вариантов использования и ее описание

### User-flow диаграммы

Описание переходов между части ПС для всех ролей из диаграммы ВИ (название ролей должны совпадать с тем, что указано на c4-модели и диаграмме вариантов использования)


---

## **Детали реализации**

### UML-диаграммы

Представить все UML-диаграммы , которые позволят более точно понять структуру и детали реализации ПС

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

