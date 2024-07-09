# multi_account_telegram_bot
Этот проект позволяет управлять несколькими аккаунтами Telegram с помощью библиотеки Telethon. Скрипт поддерживает отправку сообщений от имени разных аккаунтов и включает режим поддержания онлайна.
УстановкаКлонируйте репозиторий:git clone https://github.com/<ваш_пользователь>/multi_account_telegram_bot.gitПерейдите в директорию проекта:cd multi_account_telegram_botУстановите необходимые зависимости:pip install telethonСоздайте файл config.json и добавьте в него свои API-данные и номера телефонов:{
    "accounts": [
        {"api_id": 26940063, "api_hash": "a73324d1790b0a66e67b27eb1d2cb79d", "phone": "+380683407958"},
        {"api_id": 25272983, "api_hash": "cd744cbf48e0a9240a8e7abffaf6cf33", "phone": "+6282277987358"}
    ]
}Запустите скрипт:python bot.pyИспользованиеРежим поддержания онлайна:Отправьте команду /stay_online, чтобы включить режим постоянного онлайн.Отправьте команду /stop_online, чтобы отключить режим онлайн.Отправка сообщений:Используйте команду /send <recipient> <message>, чтобы отправить сообщение от имени одного из ваших аккаунтов.Пример:/send @username Привет!ЛогированиеВсе входящие сообщения и действия будут записываться в файл telegram.log.Пример кодаfrom telethon import TelegramClient, events, functions
import asyncio
import logging

# Загрузка конфигурации
import json
with open('config.json', 'r') as f:
    config = json.load(f)

accounts = config['accounts']
clients = []

# Настройка логирования
```logging.basicConfig(filename='telegram.log', level=logging.INFO)

# Флаг для поддержания онлайна
keep_alive = False

async def main():
    global keep_alive
    
    for account in accounts:
        client = TelegramClient(account['phone'], account['api_id'], account['api_hash'])
        await client.start(phone=account['phone'])
        clients.append(client)
        
        @client.on(events.NewMessage)
        async def handler(event):
            logging.info(f"Message from {event.sender_id}: {event.raw_text}")
            # Обработка команд для отправки сообщений
            if event.raw_text.startswith('/send '):
                parts = event.raw_text.split(' ', 2)
                if len(parts) == 3:
                    recipient = parts[1]
                    message = parts[2]
                    try:
                        await client.send_message(recipient, message)
                        await event.reply(f"Message sent to {recipient}")
                    except Exception as e:
                        await event.reply(f"Failed to send message: {e}")
                else:
                    await event.reply("Usage: /send <recipient> <message>")
            
            # Команда для включения режима онлайн
            if event.raw_text == '/stay_online':
                keep_alive = True
                await event.reply("Keep alive mode activated.")
            
            # Команда для отключения режима онлайн
            if event.raw_text == '/stop_online':
                keep_alive = False
                await event.reply("Keep alive mode deactivated.")
    
    # Функция для поддержания онлайна
    async def stay_online():
        while True:
            if keep_alive:
                for client in clients:
                    await client.send_message('me', '.')
                    await client(functions.account.UpdateStatusRequest(offline=False))
            await asyncio.sleep(300)  # Пауза в 5 минут между отправками keep-alive сообщений
    
    await asyncio.gather(
        *[client.run_until_disconnected() for client in clients],
        stay_online()
    )

if __name__ == '__main__':
    asyncio.run(main())ЛицензияЭтот проект распространяется под лицензией MIT. Смотрите файл LICENSE для подробностей.```
