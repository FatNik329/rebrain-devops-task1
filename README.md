# TelegramBot

Как то раз в голове созрела идея, написать собственного бота для **Telegram**. Чтобы, он подключался к домашнему устройству **MikroTik** и осуществлял простенькие запросы на маршрутизаторе, например: *вывод статистики интерфейса*, *сделать бэкап конфига*, *перезапустить VPN соединение и т.д. и .т.п. 

В последствии к функционалу бота добавились ещё функции для вывода данных с домашних серверов на ОС **Debian** и **Raspberry Pi**, а также контроль работоспособности важных сервисов на этих серверах. 


## Условия использования

Для того, чтобы проект заработал, необходимо:
 1. Бот **Telegram** созданный через **[BotFather](https://telegram.org/botFather)**;
 2. **Token** и **API** созданного бота;
 3. Сервер работающий 24/7 (ОС **Debian**, **Linux**), где будет работать бот; 
 4. Установленный на сервере **Bash**;
 5. Установлена на сервере утилита **Curl**;
 6. Установленный на сервере язык программирования **[Python](https://www.python.org)** (желательно 3-й версии);
 7. Неглубокие знания **Bash** и **Python**.

![Debian](https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white)
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Telegram](https://img.shields.io/badge/Telegram-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)
![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)

 ---
 
 Также в дополнение к **Python** необходимо установить следующие модули:
  - **[RouterOS-API](https://pypi.org/project/RouterOS-api/#description "https://pypi.org/project/RouterOS-api/#description")** (для работы с **API  MikroTik**);
  - **[pyTelegramBotAPI](https://pypi.org/project/pyTelegramBotAPI/ "https://pypi.org/project/pyTelegramBotAPI/")** (собственно, для написания самого бота **Telegram**);
  - **[subprocess](https://python-scripts.com/subprocess "https://python-scripts.com/subprocess")** (дает возможность запускать процессы программ из **Python**. С помощью него, вы можете запускать приложения из **Python** и передавать им аргументы при помощи модуля **subprocess**)

---
>Также, не забудьте проверить, что на вашем устройстве MikroTik включен сервис API и в Firewall прописано разрешающее правило для IP адреса вашего сервера, где работает бот. 

## Установка
По сути для того, чтобы проект заработал, необходимо, чтобы были соблюдены все пункты из раздела "**Условия использования**". Достаточно просто закинуть весь проект на тот сервер, что был подготовлен для использования данного проекта.

Вкратце из чего состоит проект: 

| Каталог | Описание |
| ------ | ------ |
| telegrambot | Содержит в себе самого бота, который взаимодействует с ботом в Telegram. |
| mikrotik | Функционал для взаимодействия с устройством MikroTik. |
| debian | Функционал для взаимодействия с сервером Debian. |
| raspberry | Функционал для взаимодействия с мини сервером Raspberry Pi. |

### Каталог telegrambot
Самое главное в каталоге - это файл **TelegramBot.py**, в котором прописывается **API** и **Token** вашего бота, также описан весь функционал для реализации функционала бота.

Измените в нём значение на **API** и **Token** вашего **Telegram** бота:
```python
bot = telebot.TeleBot("xxxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxx", threaded=False)
```
---
За создание клавиатуры с кнопками, которые содержат в себе необходимый функционал бота, отвечают строки:
```python
keyboard1 = telebot.types.ReplyKeyboardMarkup(True, True)
keyboard1.row('Check connect MikroTik', 'BackupMikrotik', 'Condition MikroTik', 'Check vers ROS', 'Restart IGMP', 'Restart L2TP VPN')
```
---
За вызов клавиатуры в чате вашего бота **Telegram** отвечают следующие строки:
```python
#Кнопки для MikroTik
@bot.message_handler(commands=['mikrotik'])
def start_message(message):
    bot.send_message(message.chat.id, 'Это меню для "MikroTik RB4011" \U0001F916', reply_markup=keyboard1)
```

---
Для вызова необходимой функции, при работе с каким либо из устройств, отвечает оператор **if-elif-else**, в котором прописан путь до файлов-скриптов с необходимым функционалом. Обращаем внимание, что для вызова функции используется название кнопки клавиатуры:
```python
if message.text.lower() == 'check connect mikrotik':
        subprocess.Popen(['python3', '/python/telegrambothome/mikrotik/ScriptTestBot.py'])
    elif message.text.lower() == 'backupmikrotik':
        subprocess.Popen(['python3', '/python/telegrambothome/mikrotik/ScriptBackupConf.py'])
        bot.send_message(message.chat.id, 'Сделано, бро, команда отправлена!')
```
---
Также бот позволяет узнать **ID** стикера используемого в чатах **Telegram**, что может быть иногда полезно ~~да ничего подобного~~. Отвечает за функцию следующие строки:
```python
@bot.message_handler(content_types=['sticker'])
def sticker_id(message):
    bot.send_message(message.chat.id, f'Вот тебе ID стикера, бро: {message.sticker.file_id}'
```
На этой ноте с рассмотрением файла **TelegramBot.py**, да и каталога **telegrambot** в целом можно закончить.

### Каталог mikrotik
Содержит в себе файлы-скрипты написанные на языке **Python** в которых описан функционал для работы с устройством:
```python
connection = routeros_api.RouterOsApiPool('192.168.0.1', username='Admin', password='Admin', plaintext_login=True, use_ssl=True, ssl_verify=False, ssl_verify_hostname=True)
api = connection.get_api()
```
Что нужно изменить:
 - **192.168.0.1** - IP вашего устройства;
 - **username** - логин от вашего устройства;
 - **password** - пароль от вашего устройства.

### Каталог debian и raspberry
Содержит в себе файлы-скрипты написанные на языке **Bash** в которых описан функционал для работы с устройством:
```bash
nohup curl -s -X POST https://api.telegram.org/botxxxxxxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxx/sendMessage -d chat_id=xxxxxxxxxx -d text="$(hostname): journalctl -p 3 -n 25
$(journalctl -p 3 -n 25)" >/dev/null &
```
Что нужно изменить:
 - **botxxxxxxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxx** - прописать API и Token бота;
 - **chat_id=xxxxxxxxxx** - прописать ID чата или группы с ботом.
 ---
 > Функционал бота можно дополнять самостоятельно своими скриптами, всё ограничивается лишь фантазией.

## Как запустить/отключить бота ?
Для запуска бота используется команда:

    nohup python3 путь_до_бота/telegrambot/TelegramBot.py >/dev/null &
Для остановки бота используется команда:

    pkill -f путь_до_бота/telegrambot/TelegramBot.py
## Как это работает ?
После запуска бота открываем чат бота в **Telegram**. Для вызова клавиатуры используется знак "**/**" + "**название клавиатуры**". 

На текущий момент доступно три варианта клавиатуры: 

 1. mikrotik;
 2. debian;
 3. raspberry.

Пример вызова клавиатуры **debian**:

    /debian
Думаю общий смысл...

## Благодарности
Мне, маме с папой, Telegram'у, [Макаронному монстру](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D1%81%D1%82%D0%B0%D1%84%D0%B0%D1%80%D0%B8%D0%B0%D0%BD%D1%81%D1%82%D0%B2%D0%BE).

## Лицензия
open-source software