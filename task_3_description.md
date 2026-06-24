# Задание 3.1: REST API для регистрации пользователя

---

## 1. Общее описание эндпоинта

- HTTP метод: POST
- URL: /api/v1/auth/register
- Content-Type: application/json
- Авторизация: не требуется

---

## 2. Входные параметры (тело запроса)

Поле          | Тип    | Обязательно? | Ограничения
--------------|--------|--------------|-------------------------------------------
firstName     | string | Да           | 1-50 символов, только буквы
lastName      | string | Да           | 1-50 символов, только буквы
username      | string | Да           | 3-30 символов, буквы/цифры/подчеркивание
password      | string | Да           | Минимум 8 символов. Должен содержать: 1 цифру, 1 заглавную букву, 1 строчную букву, 1 спецсимвол, 1 не-буквенно-цифровой символ
recaptchaToken| string | Да           | Токен от Google reCAPTCHA после прохождения проверки

---

## 3. Выходные параметры при успехе (HTTP 201 Created)

Поле     | Тип    | Обязательно? | Описание
---------|--------|--------------|-------------------------------------------
userId   | string | Да           | Уникальный ID созданного пользователя (UUID)
username | string | Да           | Имя пользователя (совпадает с отправленным)
message  | string | Да           | Текст подтверждения

---

## 4. Выходные параметры при ошибке

Поле        | Тип    | Обязательно? | Описание
------------|--------|--------------|-------------------------------------------
errorCode   | string | Да           | Код ошибки в системе (VALIDATION_ERROR, USER_EXISTS и т.д.)
message     | string | Да           | Текст ошибки для пользователя
fieldErrors | array  | Нет          | Список полей с ошибками (только для 400)

---

## 5. Коды ответов и ошибки

Код | Кто виноват      | Когда возникает                              | Текст сообщения
----|------------------|---------------------------------------------|--------------------------
201 | — (успех)        | Пользователь успешно создан                 | User registered successfully
400 | Клиент (фронт)   | Невалидные данные (пустые поля, пароль не по правилам) | Passwords must have at least one non alphanumeric character...
400 | Клиент (фронт)   | reCAPTCHA не пройдена                       | Please verify reCaptcha to register!
409 | Клиент (фронт)   | Пользователь с таким username уже есть      | User exists!
500 | Сервер (бэк)     | Проблема с базой данных, сервер недоступен | Internal server error. Please try again later.

---

## 6. Примеры

### 6.1. Успешный запрос

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

---

### 6.2. Ошибка — пользователь уже существует

Запрос (тот же, что выше)

Ответ (409 Conflict):

{
  "errorCode": "USER_EXISTS",
  "message": "User exists!"
}

---

### 6.3. Ошибка — не пройдена reCAPTCHA

Ответ (400 Bad Request):

{
  "errorCode": "RECAPTCHA_FAILED",
  "message": "Please verify reCaptcha to register!"
}

---

### 6.4. Ошибка — пароль не соответствует требованиям

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

---

## 7. Примечания для разработчика

1. reCAPTCHA проверяется ДО валидации остальных полей.
2. Если username уже занят — возвращаем 409, не проверяем остальные поля.
3. Пароль храним в БД только в захешированном виде (bcrypt).
4. После успешной регистрации можно сразу выдать access token (опционально).
