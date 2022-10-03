Структура URI
=============

//:entity\[//:id\]\[/?params\]

-   **entity** - название сущности, например таблицы в бд. Примеры:
    users, dictionary
-   **id** - первичный ключ объекта. Если первичный ключ составной, то
    части указываются через слэш. Примеры: /users/10,
    /dictionary/ru/apptitle
-   **params** - дополнительные параметры

Методы HTTP
===========

1.  **GET** //:entity//:id **- getById**

    В случае `успеха` сервер возвращает [200 OK с полями объекта в
    формате JSON в теле ответа]{.underline} В случае, если объект с
    таким id `не существует`, сервер возвращает [404 Not
    Found]{.underline}

2.  **GET** /:entity\[?param=1...&param2=...\] **- спичосный get**

    В случае `успеха` сервер возвращает [200 OK с массивом в формате
    JSON в теле ответа]{.underline} Если по заданным параметрам объектов
    `не существует` то все равно возвращается [200 OK с пустым массивом
    \[\] в теле ответа]{.underline}

3.  **HEAD** //:entity\[//:id\] **- запрос заголовков**

    Полный аналог GET с таким же URI, но не возвращает тело ответа, а
    только HTTP-заголовки Реализация поддержки HEAD запросов
    **обязательна** Активно используется браузерами в качестве
    автоматических pre-flight запросов перед выполнением потенциально
    опасных, по их мнению, операций. Может использоваться для проверки
    существования объекта без его передачи(например, для больших
    объектов или медиа файлов)

4.  **POST** /:entity **- создает новый объект типа :entity**

    В случае `успеха` сервер должен возвращать [201 Created с пустым
    телом, но с дополнительным заголовком]{.underline} - Location:
    //:entity//:new~id~ Возвращать тело ответа чаще всего не требуется.

    Если по заданным параметрам объект `сущесвует` то возвращаем [409
    conflict с пустым телом]{.underline}

5.  **PUT** //:entity//:id **- изменняет объект целиком**

    В запросе должны содержаться **все** поля изменяемого объекта в
    формате JSON В случае `успеха` должен возвращать [204 No Data с
    пустым телом]{.underline}

6.  **PATCH** //:entity//:id **- изменяет отдельные поля объекта**

    В запросе должны быть перечислены только поля, подлежашие изменениюю
    В случае `успеха` вощвращает [200 OK с телом, аналогичным запросу
    getById, со всеми полями измененного объекта]{.underline}

7.  **DELETE** //:entity//:id **- удаляет объект, если он существует.**

    В случае `успеха` возвращает [204 No Data с пустым
    телом]{.underline}

8.  **OPTIONS** //:entity\[//:id\]

    Сервер должен ответить [200 ОК с дополнительным
    заголовком]{.underline} - Allow: GET, POST, ...

    Некешируемый необязательный метод

Обработка ошибок
================

Возвращаемы ошибки передаются с сервера на клиент как ответы со статусом
4xx(ошибка клиента) или 5xx(ошибка сервера). При этом описание ошибки,
если оно есть, приводится в теле формата text/plain, без всякого JSON

Наиболее часто используемые коды
--------------------------------

-   **400 Bad Request**

    Универсальный код ошибки если серверу не понятен запрос от клиента

-   **403 Forbidden**

    Возвращается если операция запрещена для текущего пользователя

-   **404 Not Found**

    Возвращается если в запросе был указан несуществующий entity или id
    несущетсвующего объекта

    Списочные методы get не должны возвращать этот код при верном entity

    Если запрос вообще не удалось разобрать, следует возвращать 418

-   **415 Unsupported Media Type**

    Возвращается при загрузке файлов на сервер, если фактический формат
    переданного файла не поддерживается. Так же может возвращаться, если
    не удалось распарсить JSON запроса, или сам запрос пришел не в
    формате JSON

-   **418 I\'m a Teapot**

    Возвращается для неизвестных серверу запросов, которые не удалось
    даже разобрать. Обычно это указывает на ошибку в клиентe, типа
    ошибки при формировании URI,либо что версия протокола клиента и
    сервера не совпадают

    Этот ответ удобно использовать, чтобы отличать запросы на
    неизвестные URI (т.е явные баги клиента) от ответов 404, у которых
    просто нет данных (элемент не найден). В отличие от 404, код 418 не
    бросается никаким промежуточным софтом. Альтернатива - использовать
    для обозначения ситуаций \"элемент не найден\" 410 Gone, но это не
    совсем корректно, т.к. предпологается, что ресурс когда-то
    существовал. Да и выделить баги клиента из потока 404 будет сложнее

-   **419 Authentication Timeout**

    Отправляется, если клиенту нужно пройти повторную
    авторизацию(например протухли куки, или CSRF токены). При этом на
    клиенте могут быть несохраненные данные, которые будут потеряны,
    если просто выкинуть клиента на страницу авторизации

-   **422 Unprocessable Entity**

    Запрос корректно разобран, но содержание запроса не прошло серверную
    валидацию.

    Например, в теле запроса были указаны неизвестные серверу поля, или
    не были указаны обязательные, или с содержимым полей что-то не так

    Обычно это означает ошибку в веденных пользователем данных, но может
    так же быть вызвано ошибкой на клиенте или несовпадением версий

-   **500 Internal Server Error**

    Возвращается если на сервере вылетело наобработанное исключение или
    произошла любая другая ошибка времени исполнения

    Все что может сделать клиент в этом случае - это уведомить
    пользователя и сделать console.error(err) для более продвинутых
    товарищей (админов, разработчиков и тестировщиков)

-   **501 Not Implemented**

    Возвращается, если текущий метод не применим(не реализован) к
    объекту запроса.
