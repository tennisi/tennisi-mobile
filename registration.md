# Регистрация c маркетинговых лендингов

* просим пользователя ввести номер телефона
* по номеру телефона определяем регион:

```dart
String? getRegionFromPhoneNumber(String phone) => phone.startsWith('7')
    ? 'kz'
    : phone.startsWith('996')
        ? 'kg'
        : null;
```

* просим подтвердить что возраст пользователя соответствует местному законодательству:
    примеры текстов из мобильного клиента:

  * "CodeScreenI18n_ageConfirmKG": "Я подтверждаю, что мне больше 18 лет и соглашаюсь со всеми условиями и правилами букмекерской конторы",
  * "CodeScreenI18n_ageConfirmKZ": "Я подтверждаю, что мне больше 21 года и соглашаюсь со всеми условиями и правилами букмекерской конторы",

    текст правил можно получить из нашей CMS [KG](https://strapi.karamba.cloud/api/documents?filters%5Bbrands%5D%5Bname%5D%5B%24eqi%5D=chesnok&locale=ru&filters%5Bregions%5D%5Bname%5D%5B%24eqi%5D=kg&filters%5Bslug%5D%5B%24eqi%5D=rules) [KZ](https://strapi.karamba.cloud/api/documents?filters%5Bbrands%5D%5Bname%5D%5B%24eqi%5D=chesnok&locale=ru&filters%5Bregions%5D%5Bname%5D%5B%24eqi%5D=kz&filters%5Bslug%5D%5B%24eqi%5D=rules)

* в зависимости от региона все сетевые запросы направляем на [KG сервер](https://kg.chesnok.bet) или [KZ сервер](https://kz.chesnok.bet)
* далее делаем post запрос на отправку смс с кодом регистрации на endpoint [`/api/v2/registration/check-phone`](https://confluence.tennisi.it/display/BOOK/Api+v2.0) и передаем номер телефона

  пример запроса на нашем dev окружении

  ```bash
  curl -X POST https://kg.chesnok.dev/api/v2/registration/check-phone\
    -H 'Content-Type: application/json'\
    -d '{"phoneNumber":"996888007717"}'
  ```

  в ответ приходит объект

  ```json
  {
    "type": "string",
    "requisite": "string",
    "resendDate": "2023-09-20T07:28:31.9568798Z",
    "attemptsLeft": int,
    "attemptsTotal": int,
    "sessionToken": "string",
    "phoneToken": "string",
    "serverTime": "2023-09-20T07:27:31.9599168Z"
  }
  ```

  нас может заинтересовать поля `attemptsLeft`, `attemptsTotal`.

* после ввода пользователем кода, делаем post запрос на endpoint `/api/v2/registration/check-code`, передаем в него номер телефона и код из смс

  пример запроса на нашем dev окружении

  ```bash
  curl -X POST https://kg.chesnok.dev/api/v2/registration/check-code\
    -H 'Content-Type: application/json'\
    -d '{"phoneNumber":"996888007717", "code": "2591"}'
  ```

  в ответ приходит объект

  ```json
  {
    "sessionToken": "string",
    "phoneToken": "string",
  }
  ```

* после просим пользователя придумать придумать пароль и отсылаем делая post запрос на endpoint `/api/v2/registration/set-password`

  пример запроса на нашем dev окружении

  ```bash
  curl -X POST https://kg.chesnok.dev/api/v2/registration/set-password\
    -H 'Content-Type: application/json'\
    -d '{"sessionToken":"1153958553581518848","phoneToken":"1153958554424573952", "password": "qwerqwer"}'
  ```

* после отсылаем делая post запрос на endpoint `/api/v2/auth`

  пример запроса на нашем dev окружении

  ```bash
  curl -X POST https://kg.chesnok.dev/api/v2/auth\
    -H 'Content-Type: application/json'\
    -d '{"username":"996888007717", "password": "qwerqwer", "forceRedirect": true}'
  ```

  в ответ приходит 203 и объект

  ```json
  {
    "host": "string",
    "key": "string"
  }
  ```

* далее формируем url по шаблону `${host}?redirectKey=${key}` и открываем его в браузере
