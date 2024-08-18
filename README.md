#### Опис

Телеграм-бот для організації зустрічей Random Coffee.

#### Налаштування бота

Через @BotFather

Бот повинен вміти приймати 3 команди:

```text
start -  Головне меню / Профіль
info - Довідка
feedback - Залишити відгук
```

+1 команда для бота, що використовується в розробці:

```
status - Стан діалогу
```

#### Управління

##### Запуск робота на сервері (режим WEBHOOK)

```bash
./start_production.sh
```

##### Запуск робота локально (режим POLLING) з доступом до інтерфейсу адміністратора

```bash
./start_development.sh
```

#### CRON на сервері

```txt
# KYIV LOCAL TIME IS +3h RELATIVE TO SYSTEM TIME
# m h  dom mon dow   command
0 9 * * THU ~/random_coffee_platform/scripts/send_invitations_prod.sh
0 9 * * MON ~/random_coffee_platform/scripts/connect_participants_prod.sh
0 15 * * FRI ~/random_coffee_platform/scripts/collect_feedback_prod.sh
```

#### База даних

##### Відкрити базу даних та пошаритися по ній

SQLite. 1 file = 1 database.

Підключитись до бази:

```bash
sqlite3 random_coffee_platform/db.sqlite3
```

Показати таблиці

```sqlite-psql
.tables
```

Переглянути кількість користувачів у базі:

```sqlite-psql
SELECT COUNT(*) FROM connector_user;
```

Вийти з бази: **Ctrl+D**.

#### Нотатки

1. Пошарити контакт, не знаючи його телефон, не можна - API вимагатиме вказівки обох полів.
1. Телефони для фейкових телеграм-акаунтів можна взяти тут: https://grizzlysms.com/
1. Django-обв'язка багато в чому перейнята у https://github.com/jlmadurga/django-telegram-bot
1. Щоб щось вивести в лог, використовувати logger.info() / logger.error()
1. Усі стани бота пишуться в стилі CamelCase і закінчуються на State, перераховані в chatbot.py
1. Щоб створити користувача вручну, потрібно обов'язково створити обидва об'єкти User та UserState
1. Забаненим користувач (enabled=False в User) більше не надсилаються запрошення
1. Створення SSL сертифіката для отримання повідомлень по webhook'ам: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04

#### Побажання

1. Мотивація зустрічі - має бути multiple choice поле
1. Додати вибір теми зустрічі - прикольно @MRRandomCoffeeBot, але там швидко утвориться бардак
1. У профілі потрібно додати кнопку "Відмовитись від зустрічі на цьому тижні"
1. Показувати іконку "друкує" в боті після отримання повідомлення від користувача (https://github.com/python-telegram-bot/python-telegram-bot/wiki/Code-snippets, Send chat action)
1. Надсилати не тільки телефон, а й телеграмний username у складі опису зустрічі (щоб клінкути і перейти)
1. Додати список дозволених/заборонених користувачів
1. Винести налаштування системи в адмінку Django
1. Додати функцію "Розсилка повідомлення новин"
1. Переставити місцями рядки у профілі, щоб номери рядків відповідали номерам запитань
1. Локалізація – щоб можна було вибирати мову інтерфейсу.
1. Текстове поле "коментар" для кожного користувача

#### Проблемки

1. Рефакторинг: chatbot.py все ще використовує змінні з vars.py, але не всі;
1. Тести: код пройшов тільки ручне тестування, unit тести не написані
1. Все, що позначено у коді мітками TODO

#### Налаштування NGINX

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
        proxy_pass http://localhost:8081;
        proxy_read_timeout 600s;
        proxy_connect_timeout 600s;
    }
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     /home/admin/cert.pem;
    ssl_certificate_key /home/admin/pkey.key;

    location /<BOT_TOKEN> {
        proxy_pass http://127.0.0.1:8081;
    }

    location /<BOT_TOKEN> {
        proxy_pass http://127.0.0.1:8081;
    }

}
```
