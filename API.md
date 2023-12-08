# bnn-pay.com документация модуля оплаты

> ОБРАТИТЕ ВНИМАНИЕ!
> 
> Ключи показаные в руководстве **НЕ ПРЕДОСТАВЛЯЮТ ДОСТУПА К ФУНКЦИОНАЛУ**, но позволят Вам проверить правильность работы подписи в вашем приложении.
>
> Для работы с API необходимо быть зарегистрированым на https://bnn-pay.com/
>

## Общая информация

* Базовый адрес: https://bnn-pay.com/api/
* Все поля, связанные со временем указыватся с нулевым смещением (UTC+0) в формате *YYYY-MM-DD'T'hh:mm:ss*)<br>**Пример:** 2023-08-20T13:51:00 (Московское время: 2023-08-20T16:51:00)
* API-ключи **чувствительны к регистру**.

## HTTP-коды возврата
|Код возврата| Описание |
|--|--|
| 200 | Операция выполнена успешно, возвращается объект JSON |
| 4XX | Используются для некорректно сформированных запросов; проблема на стороне отправителя |
| 401 | Используется при ошибке авторизации |
| 408 | Используется если превышен интервал запроса |
| 5XX | Используются для внутренних ошибок, проблема на стороне bnn-pay. **Важно НЕ рассматривать это как неудачную операцию; состояние выполнения НЕИЗВЕСТНО и могло быть выполнено успешно** |

## Авторизация

* **Обязательно:** Для каждого запроса ВНЕ ЗАВИСИМОСТИ ОТ ТИПА добавляется штамп времени (текущее время в формате UTC), передается в параметре запроса timestamp.
> GET https://bnn-pay.com/api/orders?timestamp=2023-08-20T13:51:00
  
Для авторизации необходимо поместить следущие параметры в заголовок запроса:
|Заголовок| Содержание |
|--|--|
| UID | **Персональный ключ (UID)** |
| SIGNATURE | **MD5-хеш (SIGNATURE)** |

* **Заголовки необходимо формировать для каждого запроса к API**
* **MD5-хеш формируется в зависимости от типа запроса (GET/POST)**
* Для рассчета **MD5-хеша (SIGNATURE)** потребуется **персональный ключ (UID)** и **приватный ключ (PRIVATE_KEY)**. Их можно получить в <a href="https://bnn-pay.com/manage/account">личном кабинете</a>.
* **ВНИМАНИЕ! PRIVATE_KEY ИСПОЛЬЗУЕТСЯ ТОЛЬКО ДЛЯ ПОДПИСИ ЗАПРОСА И В ЯВНОМ ВИДЕ НЕ ПЕРЕДАЕТСЯ!**
<br>

### GET-запрос
|Параметр| Описание | Пример|
|--|--|--|
|UID|Персональный ключ|<code>e7742caf-5e74-498c-8f4f-d4ae0a6f2bf3</code>|
|PRIVATE_KEY|Приватный ключ|<code>ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==</code>|
|PARAMS|Является конкантенация всех параметров в формате "param1=a&param2=b"|<code>timestamp=2023-08-20T13:51:00&page=2</code>|

Подписью является MD5-HASH("{UID}:{PRIVATE_KEY}:{PARAMS}")

Пример тела подписи для GET-запроса:

    e7742caf-5e74-498c-8f4f-d4ae0a6f2bf3:ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==:timestamp=2023-08-20T13:51:00&page=2

Полученая подпись md5

    709d3ded3e39a5d260e7887d06bd1e70

### POST-запрос
|Параметр| Описание | Пример|
|--|--|--|
|UID|Персональный ключ|<code>e7742caf-5e74-498c-8f4f-d4ae0a6f2bf3</code>|
|PRIVATE_KEY|Приватный ключ|<code>ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==</code>|
|PAYLOAD|Тело запроса|<code>{"OrderId": "11111111232132","Amount": 10000,"CallbackUrl": "https://site.com/pay/result","ReturnUrl":"https://site.com/pay/success"}</code>|

Подписью является MD5-HASH("{UID}:{PRIVATE_KEY}:{PAYLOAD}")

Пример тела подписи для POST-запроса:

    e7742caf-5e74-498c-8f4f-d4ae0a6f2bf3:ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==:{"OrderId": "11111111232132","Amount": 10000,"CallbackUrl": "https://site.com/pay/result","ReturnUrl":"https://site.com/pay/success"}

Полученая подпись md5

    d52f5a417e8eb3ce7eef991cb2bfb0ac

Пример кода (C#) подписи полезной нагрузки

    private string SignResponse(string uid, string secret, string content)
    {
        var sourceHash = $"{uid}:{secret}:{content}";

        using var md5 = MD5.Create();

        var inputBytes = Encoding.ASCII.GetBytes(sourceHash);
        var hashBytes = md5.ComputeHash(inputBytes);

        var hash = Convert.ToHexString(hashBytes);

        return hash;
    }

## Методы для работы с балансом

### Получить баланс

`GET /balance`

Получить баланс

В случае успеха, возвращается баланс.

**Параметры:** Штамп времени в строке запроса

Пример запроса:
timestamp=19.04.2023+09%3a47%3a20

Ответ (статус код 200):
```
{
  "success": true,
  "result": 0.956
}
```

## Методы для работы с заявками приема средств

### Получить доступные банки

`GET /banks`

Получить список доступных банков.

В случае успеха, возвращается список банков (Id, название, минимальный лимит одной заявки, максимальный лимит одной заявки). Если у банка не заданы лимиты, то минимальный и максимальный лимит будут null.

**Параметры:** Штамп времени в строке запроса

Пример запроса:
timestamp=19.04.2023+09%3a47%3a20

Ответ (статус код 200):
```
{
  "success": true,
  "result": {
    [
        {
            "Id": 1,
            "Name": "KapitalBank",
            "MinimumValue": 5.0,
            "MaximumValue": 1000.0
        },
        {
            "Id": 2,
            "Name": "AzeriCard",
            "MinimumValue": null,
            "MaximumValue": null
        }
    ]
  }
}
```

### Создать фиатную заявку приема средств

`POST /order/create`

Метод создания заявки для оплаты. Опционально можно указать Id банка, чтобы создать заявку с выбранным банком. 

Желательно указывать информацию о клиенте в поле "ClientInfo".

В случае успеха, возвращается хеш заявки и url онлайн-оплаты.

Если вы указали банк, то в случае успеха возвращается хеш, url и реквизиты.

Если вы указали банк, но все реквизиты заняты, то вы получите код 400 и сообщение "The selected bank does not have free details."

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы используете внешний ключ повторно;
- Если вы указали Id несуществующего банка;
- Если вы указали банк, но все реквизиты заняты
- Если вы не указали URL коллбека и в профиле он также не указан
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| OrderId |string  |  Да|  Внешний ключ заявки|
| Amount |decimal|  Да|  Сумма заявки|
| CallbackUrl|string|  Нет|  URL оповещения о результата|
| ReturnUrl|string|  Нет|  URL возврата в случае успеха|
| ClientInfo|string|  Нет|  Информация о клиенте|
| BankId|int|  Нет |  Id выбранного банка|

Пример запроса:
### Создание заявки без принудителього выбора банка
```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "callbackUrl": "https://site.com/pay/result",
  "returnUrl": "https://site.com/pay/success",
  "clientInfo": "User777"
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "hash": "b78903c0-ed46-4d94-9d53-f09a3f876f51",
    "payUrl": "https://bnn-pay.com/payment/b78903c0-ed46-4d94-9d53-f09a3f876f51"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 401,
        "descriptionCode": 401,
        "message": "Unauthorized",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

### Создание заявки с принудительным выбором банка 

```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "callbackUrl": "https://site.com/pay/result",
  "returnUrl": "https://site.com/pay/success",
  "clientInfo": "User777",
  "bankId": 1
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "hash": "b78903c0-ed46-4d94-9d53-f09a3f876f51",
    "payUrl": "https://bnn-pay.com/payment/b78903c0-ed46-4d94-9d53-f09a3f876f51",
    "cardDetail": "4127209302903924"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
      "code": 400,
      "descriptionCode": 400,
      "requestErrors": {
        "bankId": [
          "Банк не найден"
        ]
      },
      "time": "2023-08-12T07:42:51.3157655Z"
    }
}
```

### Создать фиатную заявку приема средств с любым банком

`POST /order/create/any/bank`

Метод создания заявки для оплаты с любым банком. Если нет доступных реквизитов, то заявка сразу отменяется.

Желательно указывать информацию о клиенте в поле "ClientInfo".

В случае успеха возвращается хеш, url, id и название банка, реквизиты.

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы используете внешний ключ повторно;
- Если все реквизиты заняты;
- Если вы не указали URL коллбека и в профиле он также не указан
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| OrderId |string  |  Да|  Внешний ключ заявки|
| Amount |decimal|  Да|  Сумма заявки|
| CallbackUrl|string|  Нет|  URL оповещения о результата|
| ReturnUrl|string|  Нет|  URL возврата в случае успеха|
| ClientInfo|string|  Нет|  Информация о клиенте|

Пример запроса:

```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "callbackUrl": "https://site.com/pay/result",
  "returnUrl": "https://site.com/pay/success",
  "clientInfo": "User777"
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "hash": "b78903c0-ed46-4d94-9d53-f09a3f876f51",
    "payUrl": "https://bnn-pay.com/payment/b78903c0-ed46-4d94-9d53-f09a3f876f51",
    "bankId": 1,
    "BankName": "ATBBank",
    "cardDetail": "4127209302903924"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 401,
        "descriptionCode": 401,
        "message": "Unauthorized",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

### Создать фиатную заявку приема средств с платежным чеком

`POST /order/create/check`

Метод создания заявки для оплаты с платежным чеком. 

Если клиент выполнил перевод средств без заявки, вы можете создать её с платежным чеком. После подтверждения\отклонения вы получите коллбек.

Желательно указывать информацию о клиенте в поле "ClientInfo".

В случае успеха возвращается хеш заявки.

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы используете внешний ключ повторно;
- Если вы отправили некорректный чек;
- Если вы не указали URL коллбека и в профиле он также не указан
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| OrderId |string  |  Да|  Внешний ключ заявки|
| Amount |decimal|  Да|  Сумма заявки|
| CallbackUrl|string|  Нет|  URL оповещения о результата|
| Photo|byte[]|  Да|  Фото чека|
| ClientInfo|string|  Нет|  Информация о клиенте|

Пример запроса:

```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "callbackUrl": "https://site.com/pay/result",
  "clientInfo": "User777",
  "Photo": "/9j/4AAQSkZJRgABAQ...." //Сериализованный массив байт
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "hash": "b78903c0-ed46-4d94-9d53-f09a3f876f51"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 401,
        "descriptionCode": 401,
        "message": "Unauthorized",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

### Получить фиатную заявку приема средств по хешу

`GET /order`

Получить заявку по Hash.

В случае успеха, возвращается заявка.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации хеша (пустой или отсутсвует);

Вам возвращается код ответа 404 и текст ошибки, если:
- Хеш транзакции не найден;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| hash|string  |  Да|  Хеш ордера|

Пример запроса:
hash=d41ff6fd-d5ec-473a-8485-64ae881b8ce7&timestamp=19.04.2023+09%3a47%3a20
Ответ (статус код 200):
```
{
  "success": true,
  "result": {
        "id": 2,
        "hash": "d41ff6fd-d5ec-473a-8485-64ae881b8ce7",
        "amount": 5000.00,
        "resultAmount": 4850.00,
        "aznUsdtPrice": 1.7,
        "settlement": 2852.94,
        "fee": 150.00,
        "expiredAt": "2023-08-04T07:09:02.986453",
        "createdAt": "2023-08-04T06:13:06.768686",
        "confirmationDate": null,
        "result": "Success",
        "clientInfo": "User777"
      }
}
```

### Получить мои фиатные заявки приема средств

`GET /orders`

Получить мои заявки. В параметрах можно передать страницу пагинация, по умолчанию page=1.

В случае успеха, возвращается список заявок, текущая страница и общее количество страниц.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации модели;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| Hash|string  |  Нет|  Отфильтровать по хешу заявки|
| CreatedAt|DateTime  |  Нет|  Дата создания заявки|
| ExcludeExpired|bool  |  Да|  Скрыть завершенные|
| Amount|decimal  |  Нет|  Сумма заявки|
| Status|string  |  Нет|  Статус заявки|
| Page|int  |  Нет|  Страница пагинации|

Статусы заявки:

Pending - ожидает оплаты

Success - успешно завершено

Cancel - отмена заявки


Пример запроса:
excludeExpired=false&page=1&timestamp=19.04.2023+09%3a47%3a20
Ответ (статус код 200):
```
{
  "success": true,
  "result": {
    "items": [
      {
        "id": 2,
        "hash": "d41ff6fd-d5ec-473a-8485-64ae881b8ce7",
        "amount": 5000.00,
        "resultAmount": 0.00,
        "aznUsdtPrice": 0.00,
        "settlement": 0.00,
        "fee": 0.00,
        "expiredAt": "2023-08-04T07:09:02.986453",
        "createdAt": "2023-08-04T06:13:06.768686",
        "confirmationDate": null,
        "result": "Pending",
        "clientInfo": "User777"
      },
      {
        "id": 1,
        "hash": "e8b901bf-9ad1-45dc-9a9f-741241a586bc",
        "amount": 5000.00,
        "resultAmount": 4850.00,
        "aznUsdtPrice": 1.7,
        "settlement": 2852.94,
        "fee": 150.00,
        "expiredAt": "2023-08-03T14:28:39.512664",
        "createdAt": "2023-08-03T14:17:48.89544",
        "confirmationDate": "2023-08-03T14:25:39.512664",
        "result": "Success",
        "clientInfo": null
      }
    ],
    "currentPage": 1,
    "totalPages": 1
  }
}
```

### Получить платежную информацию по заявке приема средств

`GET /order/payinfo`

Получить банковские реквизиты заявки. В параметрах необходимо указать хеш заявки.

В случае успеха, возвращается модель с названием банка, реквизитами (номер карты или телефона), дата отмены заявки.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации модели;
- Вы не являетесь владельцем;
- Заявка закрыта;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| Hash|string  |  Да|  Хеш заявки|


Пример запроса:
hash=eb43fea6-80e2-4d31-a1d1-cc55fb6e327d&timestamp=19.04.2023+09%3a47%3a20

Ответ, если у заявки есть реквизиты (статус код 200):
```
{
  "success": true,
  "result": {
    "isActive": true,
        "cardDetail" : {
            "bank": "KapitalBank",
            "card"": "1111111111",
            "cancellationDate": "2023-08-12T07:00:07.293722"
        }
  }
}
```

Ответ, если у заявки нет реквизитов (статус код 200):
```
{
  "success": true,
  "result": {
    "isActive": true,
    "cardDetail" : null
  }
}
```

Ошибка:
```<json>   
{
  "Success": false,
  "Error": {
    "Code": 400,
    "DescriptionCode": 400,
    "Message": "Order expired.",
    "Time": "2023-08-12T10:09:45.2424597Z"
  }
}
```

### Подтвердить оплату фиатной заявки приема средств

`POST /order/confirm`

Метод подтверждения оплаты клиентом.
В случае успеха, возвращается "OK".

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если заявка не существует;
- Если заявка закрыта;
- Если у заявки нет банковских реквизитов;
- Если уже подтверждена оплата клиентом.
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| Hash |string  |  Да|  Хеш заявки|

Пример запроса:

```<json>   
{
  "Hash": "09c36af9-6d54-4c12-a416-e9f13c942730"
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "result": "OK"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 400,
        "descriptionCode": 400,
        "message": "Order expired.",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

### Создать апелляцию на фиатную заявку приема средств

`POST /order/appeal`

Метод создания апелляции на заявку оплаты. После подтверждения или отклонения, вы получите коллбек, так же как и с заявкой.

Чек передается в виде байтового массива, размер не более 3МБ.

В случае успеха, возвращается "OK". 

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы не являетесь владельцем заявки;
- Если заявка еще не закрыта;
- Если заявка уже в апелляции;
- Если изображение чека не корректно
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| Hash |string  |  Да|  Хеш заявки|
| Amount |decimal|  Да|  Сумма заявки|
| Photo|byte[]|  Да|  Фото чека|

Пример запроса:

```<json>   
{
  "Hash":"70CA27C3-4EE1-4D13-8A86-3EDC70FA9683",
  "Amount":10.0,
  "Photo": "/9j/4AAQSkZJRgABAQ...." //Сериализованный массив байт
}
```

Ответ (статус код 200):
```<json>   
{
    "success": true,
    "result": "OK"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 400,
        "descriptionCode": 400,
        "message": "Incorrect image.",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

## Методы работы с заявками вывода средств (выплата)

### Создать фиатную заявку выплаты средств на карту клиента

`POST /order/payout/create`

Метод создания заявки для вывода средств на карту. Обязательно необходимо указать банк и реквизиты.

В случае успеха, возвращается хеш заявки.

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы используете внешний ключ повторно;
- Если у вас недостаточно баланса;
- Если вы не указали URL коллбека и в профиле он также не указан
  
**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| OrderId |string  |  Да|  Внешний ключ заявки|
| Amount |decimal|  Да|  Сумма заявки|
| Bank|string|  Да|  Название банка|
| CardDetail|string|  Да|  Реквизиты|
| CallbackUrl|string|  Да|  URL оповещения о результата|

Пример запроса:
```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "Bank": "ATBBank",
  "CardDetail": "2222333344445555",
  "callbackUrl": "https://site.com/pay/result"
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "hash": "b78903c0-ed46-4d94-9d53-f09a3f876f51"
}
```

Ошибка:
```<json>   
{
    "success": false,
    "error": {
        "code": 401,
        "descriptionCode": 401,
        "message": "Unauthorized",
        "time": "2023-08-11T11:25:31.6590684Z"
    }
}
```

### Получить фиатную заявку выплаты средств по хешу

`GET /order/payout`

Получить заявку по Hash.

В случае успеха, возвращается заявка.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации хеша (пустой или отсутсвует);
- Хеш транзакции не найден;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| hash|string  |  Да|  Хеш ордера|

Пример запроса:
hash=d41ff6fd-d5ec-473a-8485-64ae881b8ce7&timestamp=19.04.2023+09%3a47%3a20

Ответ (статус код 200):
```
{
  "success": true,
  "result": {
        "Id": 4,
        "Hash": "395485b1-1c3f-4c5d-bc05-1255518fb435",
        "Bank": "M10",
        "CardDetail": "899995554444",
        "Amount": 100.0,
        "ResultAmount": 101.0,
        "AznUsdtPrice": 1.72,
        "Settlement": 58.721,
        "Fee": 1.0,
        "ExpiredAt": "2023-10-30T07:32:48.635846",
        "CreatedAt": "2023-10-30T07:32:27.73036",
        "Result": "Success"
      }
}
```

### Получить мои фиатные заявки выплаты средств

`GET /orders`

Получить мои заявки. В параметрах можно передать страницу пагинация, по умолчанию page=1.

В случае успеха, возвращается список заявок, текущая страница и общее количество страниц.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации модели;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| CreatedAt|DateTime  |  Нет|  Дата создания заявки|
| Amount|decimal  |  Нет|  Сумма заявки|
| Status|string  |  Нет|  Статус заявки|
| Page|int  |  Нет|  Страница пагинации|

Статусы заявки:

Pending - ожидает оплаты

Success - успешно завершено

Cancel - отмена заявки


Пример запроса:
page=1&timestamp=19.04.2023+09%3a47%3a20

Ответ (статус код 200):
```
{
  "success": true,
  "result": {
    "items":[
        {
          "Id":4,
          "Hash":"395485b1-1c3f-4c5d-bc05-1255518fb435",
          "Bank":"M10",
          "CardDetail":"899995554444",
          "Amount":100.0,
          "ResultAmount":101.0,
          "AznUsdtPrice":1.72,
          "Settlement":58.721,
          "Fee":1.0,
          "ExpiredAt":"2023-10-30T07:32:48.635846",
          "CreatedAt":"2023-10-30T07:32:27.73036",
          "Result":"Success"},
        {
          "Id":3,
          "Hash":"7c1bf04d-84e1-4dc0-b749-45c95a9eff27",
          "Bank":"M10",
          "CardDetail":"899995554444",
          "Amount":100.0,
          "ResultAmount":10.31,
          "AznUsdtPrice":1.72,
          "Settlement":5.994,
          "Fee":0.31,
          "ExpiredAt":"2023-10-30T06:40:07.295543",
          "CreatedAt":"2023-10-30T06:39:47.126989",
          "Result":"Success"
        },
        {
          "Id":2,
          "Hash":"9610d8ae-3be3-48ec-a058-c76771cc453f",
          "Bank":"Azeri",
          "CardDetail":"22023545645644534556",
          "Amount":523.0,
          "ResultAmount":0.0,
          "AznUsdtPrice":0.0,
          "Settlement":0.0,
          "Fee":0.0,
          "ExpiredAt":"2023-10-30T06:39:06.950911",
          "CreatedAt":"2023-10-30T06:38:45.92542",
          "Result":"Cancel"
        }
    "currentPage": 1,
    "totalPages": 1
  }
}
```

## Уведомления

Мы работаем с системой уведомлений, которая уведомляет ваш сервис об изменении статуса транзакции.

Существует 10 попыток уведомлений, которые повторяются через определенный интервал времени.

Если вы получили статус, то обязательно верните код 200, чтобы система больше не отправляла уведомления по заявке.

### Как работает обратный звонок?

Прежде всего, при создании заявки вы получите идентификатор HASH, в котором указан идентификатор в нашей системе.

Когда статус транзакции будет изменен, вы получите обратные вызовы на адрес указанный при создании заявки:

POST-запрос с телом {"Hash": "хеш заявки", "Status", "Статус заявки", "ExternalId": "Внешний ключ магазина, заданный при создании заявки", "Amount": Фиатная сумма с учетом комиссии в AZN, "AznUsdtPrice": Актуальный курс USDT/AZN, "Settlement": Сконвертированная сумма в USDT с учетом комиссии, "Fee": Размер комиссии в AZN, "ReasonCancel": "Возможная причина отмены" }

ReasonCancel - необязательный строковый параметр причины отмены. В случае отмены заявки PayOut сервис указывает причину отказа, например "Карта заблокирована"

Запрос подписывается ключом api и секретом, по аналогии авторизации для вызова методов апи

Для проверки корректности информации, обязательно проверяйте хеш модели с хешем из заголовка "SIGNATURE"

Варианты статуса заявки:

Success = "Успешно",
Cancel = "Отменено",

#### Обратный вызов не означает, что заявка прошла успешно. Вы должны сравнить статус заявки.

Пример кода (C#) метод контроллера для принятия и обработки уведомлений

      /* Пример модели коллбека */
      public class OrderStatusNotification
      {
          public string Hash { get; set; }
          public string Status { get; set; }
          public string ExternalId { get; set; }
          public decimal Amount { get; set; }
          public decimal AznUsdtPrice { get; set; }
          public decimal Settlement { get; set; }
          public decimal Fee { get; set; }
          public string? ReasonCancel { get; set; }
      
          public OrderStatusNotification(
          string hash, 
          string status, 
          string externalId, 
          decimal amount, 
          decimal aznUsdtPrice, 
          decimal settlement, 
          decimal fee,
          string? reasonCancel)
          {
              Hash = hash;
              Status = status;
              ExternalId = externalId;
              Amount = amount;
              AznUsdtPrice = aznUsdtPrice;
              Settlement = settlement;
              Fee = fee;
              ReasonCancel = reasonCancel;
          }
      }
      
      /* Расширение позволяющая получать полезную нагрузку из HttpRequest */
      public static class HttpContextExtension
      {
              public static async Task<string> GetRawBodyAsync(this HttpRequest request, Encoding encoding = null)
              {
                  if (!request.Body.CanSeek)
                  {
                      // We only do this if the stream isn't *already* seekable,
                      // as EnableBuffering will create a new stream instance
                      // each time it's called
                      request.EnableBuffering();
                  }
      
                  request.Body.Position = 0;
      
                  var reader = new StreamReader(request.Body, encoding ?? Encoding.UTF8);
      
                  var body = await reader.ReadToEndAsync().ConfigureAwait(false);
      
                  request.Body.Position = 0;
      
                  return body;
              }
        }

        /* Фрагмент для контроллера фраемворка ASP.NET CORE */
        
        const string paySecret = "ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==";
        const string payUid = "f638ecdc-d7ef-40dc-a8c1-8ae42b16f43c";

        [AllowAnonymous]
        [HttpPost]
        public async Task<IActionResult> Index()
        {
            var hash = HttpContext.Request.Headers["SIGNATURE"].FirstOrDefault();  // Получаем подпись из заголовка (md5 хеш)

            if (string.IsNullOrEmpty(hash))
                return NotFound();

            var content = await Request.GetRawBodyAsync(); // Получаем полезную нагрузку в виде строки

            if (string.IsNullOrEmpty(content))
                return NotFound();

            var sourceHash = $"{payUid}:{paySecret}:{content}"; // Формируем строку для подписи исходя из данных payUid, paySecret и полученной полезной нагрузки
            using var md5 = MD5.Create();
            var inputBytes = Encoding.ASCII.GetBytes(sourceHash); // Получаем байты строки по таблице ASCII
            var hashBytes = md5.ComputeHash(inputBytes);  // Высчитываем md5 хеш
            var userHash = Convert.ToHexString(hashBytes); // Полученный байтовый хеш транслируем в текст

            if (string.Equals(userHash, hash, StringComparison.CurrentCultureIgnoreCase)) // Сравниваем полученный в заголовке md5 хеш и рассчитанный самостоятельно (Внимание! В случае возникновения проблем проверяйте регистр полученного и расчитанного хеша)
            {
                //Все ок, подписи совпадают
            }
            else
            {
                // Ошибка, подписи различаются
            }

            //Обязательно возвращаем код 200
            return Ok();
        }
    }

### Уведомление о изменении статуса банка

Мы отправляем колббек со статусом банка при его изменении (например включен или отключен) по адресу указанному в личном кабинете, раздел профиль, так же где вы указываете адрес коллбека и возврата.

POST-запрос с телом {"Id": "Id банка", "Name", "Название банка", "IsEnable": Если true, то банк активен, если false, то отключен }

Запрос подписывается ключом api и секретом, по аналогии авторизации для вызова методов апи

Для проверки корректности информации, обязательно проверяйте хеш модели с хешем из заголовка "SIGNATURE"

Пример модели коллбека, примеры метода обработки коллбека, можете посмотреть выше

      /* Пример модели коллбека */
      public class BankStatusNotification
      {
          public int Id { get; set; }
          public string Name { get; set; }
          public bool IsEnable { get; set; }
      
          public BankStatusNotification(int id, string name, bool isEnable)
          {
              Id = id;
              Name = name;
              IsEnable = isEnable;
          }
      }
