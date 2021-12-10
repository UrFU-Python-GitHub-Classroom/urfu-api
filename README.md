#  API УрФУ

API УрФУ (https://urfu.ru/api/) поддерживает несколько открытых методов:

## Расписание группы
* schedule/groups/suggest/?query=`<group>`
* schedule/groups/lessons/`<id>` - расписание на ближайшие две недели
* schedule/groups/lessons/`<id>`/`<YYYYMMDD>` - расписание на две недели от даты
* schedule/groups/calendar/`<id>` - загрузить календарь в формате `.ics`
* schedule/groups/calendar/`<id>`/`<YYYYMMDD>` - загрузить календарь на две недели от даты

## Расписание преподавателя
* schedule/teacher/suggest/?query=`<fio>`
* schedule/teacher/lessons/`<id>` 
* schedule/teacher/lessons/`<id>`/`<YYYYMMDD>`
* schedule/teacher/calendar/`<id>`
* schedule/teacher/calendar/`<id>`/`<YYYYMMDD>`

# Задача

На языке Python реализовать чат-бота для Telegram и развернуть на HEROKU

### Библиотеки, которые могут понадобиться:
* `pyTelegramBotAPI` - Telegram API для Python
* `BeautifulSoup` - парсинг HTML и XML

### Дополнительные файлы для HEROKU (должны быть в корне репозитория)
* [Procfile](Procfile) - файл запуска приложения
* [runtime.txt](runtime.txt) - окружение выполнения кода
* [requirements.txt](requirements.txt) - зависимости окружения Python

# Пример бота

```python
import requests
import json

from urllib.parse import quote

from flask import Flask, request

import telebot
from telebot import types

BOT_TOKEN = os.getenv('BOT_TOKEN')
APP_NAME = os.getenv('APP_NAME')
bot = telebot.TeleBot(BOT_TOKEN)

# Обработчики сообщений от пользователей
@bot.message_handler(content_types=['text'])
def handler(message):
   """Обрабатывает все сообщения пользователя с текстом"""
   request_url = 'https://urfu.ru/api/schedule/groups/suggest/?query=' + quote(''.join(message.text))
   response = requests.get(request_url);
   suggestions = response.json()['suggestions']
   if suggestions:
       result = ""
       for group in suggestions:
           result += f"{group['value']}\n"
       bot.send_message(message.from_user.id, text)


if __name__ == '__main__':
    
    # Чтобы запускать удаленно и локально один и тот же код, можно использовать пустую переменную среды HEROKU
    # Если она есть, код будет считать, что он на сервере
    if "HEROKU" in list(os.environ.keys()):
        server = Flask(__name__)

        @server.route(f"/{BOT_TOKEN}", methods=['POST'])
        def getMessage():
            bot.process_new_updates([telebot.types.Update.de_json(request.stream.read().decode("utf-8"))])
            return "!", 200

        @server.route("/")
        def webhook():
            bot.remove_webhook()
            bot.set_webhook(
                url=f"https://{APP_NAME}.herokuapp.com/{BOT_TOKEN}")
            return "?", 200

        server.run(host="0.0.0.0", port=int(os.environ.get('PORT', 80)))

    else:
        # если переменной окружения HEROKU нет, значит это запуск с машины разработчика
        # Удаляем вебхук на всякий случай, и запускаем с обычным поллингом
        bot.remove_webhook()
        bot.polling()
```
