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


## Методы для работы с заявками

### Создать фиатную заявку

`POST /create`

Метод создания заявки для оплаты.

Вам возвращается код ответа 400 и текст ошибки, если:
- При ошибке валидации модели;
- Если вы используете внешний ключ повторно

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| OrderId |string  |  Да|  Внешний ключ заявки|
| Amount |decimal|  Да|  Сумма заявки|
| CallbackUrl|string|  Да|  URL оповещения о результата|
| ReturnUrl|string|  Нет|  URL возврата в случае успеха|

Пример запроса:
```<json>   
{
  "orderId": "11111111232132",
  "amount": 10000,
  "callbackUrl": "https://site.com/pay/result",
  "returnUrl": "https://site.com/pay/success"
}
```
Ответ (статус код 200):
```<json>   
{
    "success": true,
    "payUrl": "https://site.com/payment/a18bb2a8-b359-412b-9dc8-704b366c7850"
    "hash": "a18bb2a8-b359-412b-9dc8-704b366c7850"
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
### Получить мои фиатные заявки

`GET /orders`

Получить мои заявки. В параметрах можно передать страницу пагинация, по умолчанию page=1.

Вам возвращается код ответа 400 и текст ошибки, если:
- Ошибка валидации модели;

**Параметры:** Штамп времени в строке запроса

|Параметр  | Тип |Обязательный|Примечание|
|--|--|--|--|
| CreatedAt|DateTime  |  Нет|  Дата создания заявки|
| ExcludeExpired|bool  |  Да|  Исключить истекшие|
| Amount|decimal  |  Нет|  Сумма заявки|
| Page|int  |  Нет|  Страница пагинации|

Пример запроса:
excludeExpired=false&page=1&timestamp=19.04.2023+09%3a47%3a20
Ответ (статус код 200):
```
{
  "success": true,
  "result": {
    "orders": [
      {
        "id": 2,
        "hash": "d41ff6fd-d5ec-473a-8485-64ae881b8ce7",
        "amount": 5000.0000000000000000000000000,
        "expiredAt": "2023-08-04T07:09:02.986453",
        "createdAt": "2023-08-04T06:13:06.768686",
        "bank": "KapitalBank",
        "cardDetail": null,
        "confirmationDate": null,
        "result": null
      },
      {
        "id": 1,
        "hash": "e8b901bf-9ad1-45dc-9a9f-741241a586bc",
        "amount": 5000.0000000000000000000000000,
        "expiredAt": "2023-08-03T14:28:39.512664",
        "createdAt": "2023-08-03T14:17:48.89544",
        "bank": "Azeri",
        "cardDetail": "2202234554345",
        "confirmationDate": "2023-08-03T14:25:39.512664",
        "result": "Success"
      }
    ],
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

POST-запрос с телом {"Hash": "хеш заявки", "Status", "Статус заявки", "ExternalId": "Внешний ключ магазина, заданный при создании заявки", "Amount": 100 }

Запрос подписывается ключом api и секретом, по аналогии авторизации для вызова методов апи

Для проверки корректности информации, обязательно проверяйте хеш модели с хешем из заголовка "SIGNATURE"

Варианты статуса заявки:

Success = "Успешно",
Cancel = "Отменено",

#### Обратный вызов не означает, что заявка прошла успешно. Вы должны сравнить статус заявки.

Пример кода (C#) метод контроллера для принятия и обработки уведомлений

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
