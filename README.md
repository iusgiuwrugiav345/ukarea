import requests
import time
import telebot

# Твой Telegram токен бота
BOT_TOKEN = 'токен бота'
bot = telebot.TeleBot(BOT_TOKEN)

# ID канала или чата, куда слать тревоги
CHANNEL_ID = '@юз канала'

# Заголовки с токеном авторизации
headers = {
    "Authorization": "Bearer 05a30d64:17b06ea560b4c2a19fac19f0ea79f5fc"
}

# Храним ID уже отправленных тревог
sent_alerts = set()

def check_alerts():
    try:
        url = "https://api.ukrainealarm.com/api/v3/alerts"
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        data = response.json()

        for region in data:
            region_name = region['regionName']
            for alert in region.get('activeAlerts', []):
                alert_id = f"{region['regionId']}_{alert['type']}_{alert['lastUpdate']}"
                if alert_id not in sent_alerts:
                    alert_type = alert['type']
                    time_utc = alert['lastUpdate'].replace("T", " ").replace("Z", "")
                    msg = f"🔴 *{alert_type} тревога* в регионе: *{region_name}*\n🕒 Время (UTC): `{time_utc}`\n🔗 [Источник](https://t.me/UkraineAreaAlert)"
                    bot.send_message(CHANNEL_ID, msg, parse_mode='Markdown')
                    sent_alerts.add(alert_id)

    except Exception as e:
        print("Ошибка при обращении к API:", e)

# Основной цикл
print("Бот запущен. Ожидание тревог...")
while True:
    check_alerts()
    time.sleep(20)
