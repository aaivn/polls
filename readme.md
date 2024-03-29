### Инструкция

#### ТЗ
Необходимо создать REST-сервис для управления опросами

Схема данных:
```
Опрос
--------------------
Наименование опроса
Дата начала
Дата окончания
Активность (да/нет)
```

```
Вопрос опроса
--------------------
Ссылка на опрос
Текст вопроса
Порядок отображения (целое число, по которому будет производиться сортировка)
```

REST-сервис должен предоставлять следующие методы:
- получить все опросы (Опционально можно передать фильтр по наименованию, дате, активности. Набор фильтров произвольный. Обязательно указание сортировки: по наименованию или по дате начала опроса. Должна поддерживаться пагинация.) Метод должен возвращать поля по опросу + все вопросы, которые в нём содержатся;
- создание опроса (метод должен принимать все данные по опросу и все вопросы сразу);
- редактирование опроса (метод должен принимать все данные по опросу и все вопросы сразу);
- удаление опроса.

Сервис должен предоставлять документацию с использованием Swagger.
Сервис при первом запуске должен самостоятельно создавать необходимые объекты в БД с помощью Liquibase.
Необходимо использовать PostgreSQL.

Для реализации необходимо использовать Java 8, Spring Boot, Hibernate, PostgreSQL, Liquibase.

Один опрос может иметь много вопросов, поле  "Порядок отображения" используется для сортировки списка вопросов, для последующего вывода их пользователю

#### Запуск приложения
- должен быть установлен docker ([см. документацию docker](docs.docker.com));
- запуск приложения (команды выполнять в корне проекта)
    - развернуть базу: 
    ```
    docker-compose -f ./docker-compose.yml up --build -d backend
    ```
    - указать host докера, задав переменную окружения, например:
    ```
    //Linux
    export APP_HOST=192.168.99.100
    или 
    export APP_HOST=localhost
    
    //Windows
    set APP_HOST=192.168.99.100
    или
    set APP_HOST=localhost
    ```
    - собрать проект, убедиться в отсутствии ошибок:
    ```
    ./gradlew clean build
    ``` 
    - запустить rest-сервис:
    ```
    docker-compose -f ./docker-compose.yml up --build -d frontend
    ```
    - проверить сервис, например(для 192.168.99.100):
        - request
        ```
        http://192.168.99.100:8080/api/polls
        ```
        - response
        ```
        {"errorMessage":"Polls not found"}
        ``` 

#### Примеры вызова
Документация находится по ссылке `{HOST}/swagger-ui.html`, где HOST - ссылка на хост, где развернут сервис.

##### 1. POST /api/polls
###### request
```
curl -L -X POST 'http://192.168.99.100:8080/api/polls' -H 'Content-Type: application/json' --data-raw '{
    "id": "1",
    "name": "Опрос1",
    "startDate": "2020-01-01",
    "endDate": "2020-12-31",
    "activity": "true",
    "questions": [
        {
            "id": "1",
            "content": "Вопрос222?",
            "orderNum": "2"
        },
        {
            "id": "2",
            "content": "Вопрос111?",
            "orderNum": "1"
        }
    ]
}'
```
###### response
```
true
```

##### 2. POST /api/polls
###### request
```
curl -L -X POST 'http://192.168.99.100:8080/api/polls' -H 'Content-Type: application/json' --data-raw '{
	"id": 2,
	"name": "Опрос1",
	"startDate": "2020-01-01",
	"endDate": "2020-12-31",
	"activity": true,
	"questions": []
}'
```
###### response
```
true
```
##### 3. PUT /api/polls
###### request
```
curl -L -X POST 'http://192.168.99.100:8080/api/polls' -H 'Content-Type: application/json' --data-raw '{
    "id": "1",
    "name": "Опрос1",
    "startDate": "2020-01-01",
    "endDate": "2020-12-31",
    "activity": "true",
    "questions": [
        {
            "id": "1",
            "content": "Вопрос222?",
            "orderNum": "2"
        },
        {
            "id": "2",
            "content": "Вопрос111?",
            "orderNum": "1"
        }
    ]
}'
```
###### response
```
true
```
##### 4. GET /api/polls
###### request
```
curl --location --request GET 'http://192.168.99.100:8080/api/polls'
```
###### response
```
[{
	"id": 1,
	"name": "Опрос1",
	"startDate": "2020-01-01T00:00:00.000+0000",
	"endDate": "2020-12-31T00:00:00.000+0000",
	"activity": true,
	"questions": [{
		"id": 2,
		"content": "Вопрос111?",
		"orderNum": 1
	}, {
		"id": 1,
		"content": "Вопрос222?",
		"orderNum": 2
	}, {
		"id": 3,
		"content": "q3",
		"orderNum": 3
	}]
},
 {
     "id": 2,
     "name": "Опрос1",
     "startDate": "2020-01-01T00:00:00.000+0000",
     "endDate": "2020-12-31T00:00:00.000+0000",
     "activity": true,
     "questions": []
 }]
```
##### 5. Filtered GET /api/polls
###### request
```
curl -L -X GET 'http://192.168.99.100:8080/api/polls?name.like=%D0%9E%D0%BF%D1%80%D0%BE%D1%81%25&startDate.eq=2020-01-01T00:00:00&endDate.lt=2022-01-01T00:00:00&size=100&sortBy.asc=name&page=0&size=1'
```
###### response
```
[
    {
        "id": 1,
        "name": "Опрос1",
        "startDate": "2020-01-01T00:00:00.000+0000",
        "endDate": "2020-12-31T00:00:00.000+0000",
        "activity": true,
        "questions": [
            {
                "id": 2,
                "content": "Вопрос111?",
                "orderNum": 1
            },
            {
                "id": 1,
                "content": "Вопрос222?",
                "orderNum": 2
            }
        ]
    },
    {
        "id": 2,
        "name": "Опрос1",
        "startDate": "2020-01-01T00:00:00.000+0000",
        "endDate": "2020-12-31T00:00:00.000+0000",
        "activity": true,
        "questions": []
    }
]
```
##### 6. GET /api/polls/1
###### request
```
curl -L -X GET 'http://192.168.99.100:8080/api/polls/1'
```
###### response
```
{
    "id": 1,
    "name": "Опрос1",
    "startDate": "2020-01-01T00:00:00.000+0000",
    "endDate": "2020-12-31T00:00:00.000+0000",
    "activity": true,
    "questions": [
        {
            "id": 2,
            "content": "Вопрос111?",
            "orderNum": 1
        },
        {
            "id": 1,
            "content": "Вопрос222?",
            "orderNum": 2
        }
    ]
}
```
##### 6. DELETE /api/polls/1
###### request
```
curl -L -X DELETE 'http://192.168.99.100:8080/api/polls/1'
```
###### response
```
true
```
