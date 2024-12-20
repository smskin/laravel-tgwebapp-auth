# Модуль аутентификации пользователей Telegram WebApp

[Документация Telegram WebApp](https://core.telegram.org/bots/webapps)

Кейс: Кейс: при разработке API для Telegram WebApp необходимо проверять, что пользователь, отправивший запрос в API, является тем, за кого он себя выдает (то есть запрос действительно пришел из Telegram WebApp).

## Принцип работы

1. JS-скрипт Telegram WebApp забирает из API объект WebAppUser и отправляет его в каждом запросе к API в заголовке запроса (название заголовка настраивается).
2. Guard получает запрос и извлекает из него объект WebAppUser.
3. Guard проверяет подпись данных с помощью BOT_TOKEN.
4. Guard ищет пользователя в базе данных:
   1. Если пользователь найден, то он авторизуется.
   2. Если пользователя нет:
      1. Если автоматическое создание пользователей разрешено, то пользователь будет создан, и будет выполнена авторизация.
      2. Если автоматическое создание пользователей запрещено, то вернется ошибка 403.

## Конфигурация

В файле `config/auth.php` необходимо зарегистрировать guard `tgwebapp`.

Примерное содержимое файла после регистрации guard:
```text
...
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'tgwebapp' => [
        'driver' => 'tgwebapp', // название guard
        'token' => env('TELEGRAM_BOT_TOKEN'), // токен бота
        'autoCreation' => true, // флаг, разрешающий автоматическое создание пользователей
        'userDataHeaderName' => 'X-TELEGRAM-USER-DATA' // название заголовка, из которого guard получает объект WebAppUser,
        'userModel' => \App\Models\User::class, // класс user модели
    ]
],
...
```

## Использование 

Подключите guard в файле роутинга `routes/web.php`.

```text
...
Route::middleware('auth:tgwebapp')->group(function(){
    Route::get('/me', function(){
        return 'Hello!';
    });
});
...
```

GET-запрос к /me пройдет аутентификацию через Telegram WebApp guard.