# Задание 3.2: Пошаговый алгоритм создания пользователя (бэкенд)

## Вход
POST /auth/register с телом запроса (JSON)

---

## Шаг 1. Получить и распарсить тело запроса
- Прочитать тело запроса
- Попытаться распарсить как JSON
- Если невалидный JSON → вернуть **422 Unprocessable Entity** с текстом «Invalid JSON»

---

## Шаг 2. Проверить, что все обязательные поля заполнены
- Проверить наличие: firstName, lastName, username, password, recaptchaToken
- Если какое-то поле отсутствует или пустое → вернуть **400 Bad Request** с текстом «Invalid input data»

---

## Шаг 3. Проверить формат каждого поля

| Поле | Что проверяем | Если не прошло |
|:---|:---|:---|
| firstName | Не пустое | 400 |
| lastName | Не пустое | 400 |
| username | Не пустое | 400 |
| password | Надёжный (мин. 8 симв., 1 цифра, 1 заглавная, 1 строчная, 1 спецсимвол) | 400 |

Если несколько полей невалидны → собрать все ошибки в `fieldErrors` и вернуть одним 400

---

## Шаг 4. Проверить reCAPTCHA
- Отправить `recaptchaToken` в сервис Google reCAPTCHA
- Если проверка не пройдена → вернуть **400** с ошибкой «Please verify reCaptcha to register!»

---

## Шаг 5. Проверить, не занят ли username
- Сделать запрос в БД: `SELECT * FROM users WHERE username = ?`
- Если пользователь найден → вернуть **409 Conflict** с текстом «User exists!»

---

## Шаг 6. Хешировать пароль
- Применить bcrypt к `password`
- Получить `passwordHash`

---

## Шаг 7. Создать пользователя в БД
- Вставить запись: firstName, lastName, username, passwordHash
- Если ошибка БД → вернуть **500 Internal Server Error** с текстом «Internal server error. Please try again later.»
- Получить сгенерированный `userId` (UUID)

---


## Шаг 8. Вернуть успешный ответ
- HTTP **201 Created**
- Тело:

    {
      "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "username": "ivan",
      "message": "User Register Successfully"
    }

---

## Блок-схема

    [Получить POST-запрос /auth/register]
            ↓
    [Распарсить JSON тело]
            ↓
    [JSON корректен?] ──Нет──→ [422 Invalid JSON]
            ↓ Да
    [Все обязательные поля заполнены?] ──Нет──→ [400 Отсутствуют поля]
            ↓ Да
    [Пароль надёжен?] ──Нет──→ [400 Слабый пароль]
            ↓ Да
    [Проверить reCAPTCHA] ──Нет──→ [400 Please verify reCaptcha to register!]
            ↓ Да
    [Запрос в БД: занят ли username?] ──Да──→ [409 User exists!]
            ↓ Нет
    [Хешировать пароль (bcrypt)]
            ↓
    [Создать запись пользователя]
            ↓
    [Ошибка записи в БД?] ──Да──→ [500 Internal Server Error]
            ↓ Нет
    [Отправить email подтверждения]
            ↓
    [201 Created + userId, username, message]

---
