 # Telegram Bot (MikroTik & Linux)
Однажды, в голове созрела идея, написать собственного бота для **Telegram**. Чтобы, он подключался к домашнему устройству **MikroTik** и осуществлял простенькие запросы на маршрутизаторе, например: *вывод статистики интерфейса*, *сделать бэкап конфига*, *перезапустить VPN соединение и т.д. и .т.п. *

В последствии к функционалу бота добавились ещё функции для вывода данных с домашних серверов на ОС **Debian** и **Raspberry Pi**, а также контроль работоспособности важных сервисов на этих серверах. 


## Условия использования

Для того, чтобы проект заработал, необходимо:
 1. Бот **Telegram** созданный через **[BotFather](https://telegram.org/botFather)**;
 2. **Token** и **API** созданного бота;
 3. Сервер работающий 24/7 (ОС **Debian**, **Linux**), где будет работать бот; 
 4. Установлен на сервере **Bash**;
 5. Установлена на сервере утилита **Curl**;
 6. Установлен на сервере язык программирования **[Python](https://www.python.org)** (желательно 3-й версии);
 7. Неглубокие знания **Bash**.

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

### telegrambot
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
Содержит в себе файлы-скрипты написанные на языке **Python** в которых описан функционал для работы с устройством. Что нужно изменить в этих файлах:
```python
connection = routeros_api.RouterOsApiPool('192.168.0.1', username='Admin', password='Admin', plaintext_login=True, use_ssl=True, ssl_verify=False, ssl_verify_hostname=True)
api = connection.get_api()
```

 - **192.168.0.1** - IP вашего устройства;
 - **username** - логин от вашего устройства;
 - **password** - пароль от вашего устройства.

### Каталог debian и raspberry
Содержит в себе файлы-скрипты написанные на языке **Bash** в которых описан функционал для работы с устройством. Что нужно изменить в этих файлах:
```bash
nohup curl -s -X POST https://api.telegram.org/botxxxxxxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxx/sendMessage -d chat_id=xxxxxxxxxx -d text="$(hostname): journalctl -p 3 -n 25
$(journalctl -p 3 -n 25)" >/dev/null &
```
