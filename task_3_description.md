1. Общее описание
| Параметр         | Значение                |
| :--------------- | :---------------------- |
| **HTTP метод**   | `POST`                  |
| **URL**          | `/api/v1/auth/register` |
| **Content-Type** | `application/json`      |
| **Авторизация**  | Не требуется            |

2. Входные параметры (тело запроса)
| Наименование     | Тип    | Обязательность | Ограничения                                                                                                                     |
| :--------------- | :----- | :------------- | :------------------------------------------------------------------------------------------------------------------------------ |
| `firstName`      | string | Да             | 1–50 символов, только буквы (A–Z, а–я)                                                                                          |
| `lastName`       | string | Да             | 1–50 символов, только буквы (A–Z, а–я)                                                                                          |
| `username`       | string | Да             | 3–30 символов, буквы, цифры, подчёркивание `_`                                                                                  |
| `password`       | string | Да             | Минимум 8 символов. Должен содержать: 1 цифру, 1 заглавную букву, 1 строчную букву, 1 спецсимвол, 1 не-буквенно-цифровой символ |
| `recaptchaToken` | string | Да             | Токен, полученный от Google reCAPTCHA после прохождения проверки                                                                |

3. Выходные параметры при успехе (HTTP 201 Created)
| Наименование | Тип    | Обязательность | Ограничения                                                     |
| :----------- | :----- | :------------- | :-------------------------------------------------------------- |
| `userId`     | string | Да             | UUID или число, уникальный идентификатор                        |
| `username`   | string | Да             | Совпадает с отправленным                                        |
| `message`    | string | Да             | Текст подтверждения, например: `"User registered successfully"` |

4. Выходные параметры при ошибке
| Наименование  | Тип    | Обязательность | Ограничения                                                                                      |
| :------------ | :----- | :------------- | :----------------------------------------------------------------------------------------------- |
| `errorCode`   | string | Да             | Код ошибки внутри системы, например: `"VALIDATION_ERROR"`, `"USER_EXISTS"`, `"RECAPTCHA_FAILED"` |
| `message`     | string | Да             | Человекочитаемое описание ошибки                                                                 |
| `fieldErrors` | array  | Нет            | Список конкретных полей с ошибками (для 400 Bad Request)                                         |

5. Коды ответов и ошибки
| HTTP код | Тип               | Когда возникает                                                         | Текст сообщения (пример)                                           |
| :------- | :---------------- | :---------------------------------------------------------------------- | :----------------------------------------------------------------- |
| **201**  | Успех             | Пользователь создан                                                     | `"User registered successfully"`                                   |
| **400**  | Клиентская ошибка | Невалидные данные (пустые поля, пароль не по правилам, неверный формат) | `"Passwords must have at least one non alphanumeric character..."` |
| **400**  | Клиентская ошибка | reCAPTCHA не пройдена                                                   | `"Please verify reCaptcha to register!"`                           |
| **409**  | Клиентская ошибка | Пользователь с таким `username` уже есть                                | `"User exists!"`                                                   |
| **500**  | Серверная ошибка  | Проблема с БД, недоступен сервис и т.д.                                 | `"Internal server error. Please try again later."`                 |

6. Примеры
6.1. Успешный запрос
POST /api/v1/auth/register
Content-Type: application/json

{
  "firstName": "Ivan",
  "lastName": "Ivanov",
  "username": "ivan",
  "password": "Ivanov123!",
  "recaptchaToken": "03AGdBq25Q_..."
}

Ответ (201 Created):
{
  "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "username": "ivan",
  "message": "User registered successfully"
}

6.2. Ошибка — пользователь уже существует
Запрос (тот же, что выше)
Ответ (409 Conflict):
{
  "errorCode": "USER_EXISTS",
  "message": "User exists!"
}

6.3. Ошибка — не пройдена reCAPTCHA
Ответ (400 Bad Request):
{
  "errorCode": "RECAPTCHA_FAILED",
  "message": "Please verify reCaptcha to register!"
}

6.4. Ошибка — пароль не соответствует требованиям
Ответ (400 Bad Request):
{
  "errorCode": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "fieldErrors": [
    {
      "field": "password",
      "message": "Passwords must have at least one non alphanumeric character, one digit ('0'-'9'), one uppercase ('A'-'Z'), one lowercase ('a'-'z'), one special character and Password must be eight characters or longer."
    }
  ]
}
