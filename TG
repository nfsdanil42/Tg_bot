import json
import requests
import time

# ---------- НАСТРОЙКИ ----------
BOT_TOKEN = "8633873027:AAHPpb1OKWwQqPnAM03dGnF6kr3yyNXoV0U"
CHANNEL_ID = "-1003857725967"  # канал
ADMIN_CHAT_ID = "423736356"  # личка

# API для проверки онлайна
MODEL_URL = "https://bngprm.com/api/v2/models-online?c=836219&client_ip=0.0.0.0&username=HeyAlise"

# Ссылка в кнопке уведомления — оставлена как была
MODEL_ONLINE = "https://ru.bonga4.com/heyalise"

CHECK_INTERVAL = 30
NOTIFY_SOUND = True
ONLINE_GIF = "/home/container/heyalise-meu.gif"

# ---------- ПЕРЕМЕННЫЕ ----------
previous_state = None
error_sent = False
online_message_id = None

# ---------- TELEGRAM ----------
def send_telegram(text, chat_id, silent=False):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = {"chat_id": chat_id, "text": text, "disable_notification": silent}

    r = requests.post(url, data=payload)
    print("send_telegram response:", r.status_code, r.text)

    try:
        return r.json()
    except Exception:
        return None


def send_online_notification():
    global NOTIFY_SOUND

    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendAnimation"

    keyboard = {
        "inline_keyboard": [[{"text": "🎥 Смотреть стрим", "url": MODEL_ONLINE}]]
    }

    payload = {
        "chat_id": CHANNEL_ID,
        "caption": "Я вышла в онлайн. Заходите на стрим ❤️",
        "reply_markup": json.dumps(keyboard),
        "disable_notification": not NOTIFY_SOUND,
    }

    try:
        with open(ONLINE_GIF, "rb") as f:
            files = {"animation": f}
            r = requests.post(url, data=payload, files=files)
            print("send_online_notification response:", r.status_code, r.text)
            return r.json()

    except Exception as e:
        print("Ошибка при отправке GIF:", e)
        send_telegram(f"⚠️ Ошибка при отправке GIF:\n{e}", ADMIN_CHAT_ID)
        return None


def delete_message(chat_id, message_id):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/deleteMessage"
    payload = {"chat_id": chat_id, "message_id": message_id}

    r = requests.post(url, data=payload)
    print("delete_message response:", r.status_code, r.text)


# ---------- ПРОВЕРКА МОДЕЛИ ----------
def check_model():
    global previous_state, error_sent, online_message_id

    try:
        headers = {"User-Agent": "Mozilla/5.0"}

        response = requests.get(MODEL_URL, headers=headers, timeout=30)

        response.raise_for_status()
        data = response.json()

        """
        Если модель оффлайн, API возвращает:
        {"message":"Invalid model username or offline model"}

        Если модель онлайн, API возвращает большой JSON с username, chat_url и т.д.
        """

        is_online = (
            isinstance(data, dict)
            and data.get("username", "").lower() == "heyalise"
            and "message" not in data
        )

        # ---------- ONLINE / OFFLINE ----------
        if previous_state != is_online:
            previous_state = is_online

            if is_online:
                msg = send_online_notification()

                if msg and msg.get("ok"):
                    online_message_id = msg["result"]["message_id"]

                print("MODEL ONLINE")

            else:
                if online_message_id:
                    delete_message(CHANNEL_ID, online_message_id)
                    online_message_id = None

                print("MODEL OFFLINE")

        else:
            print("Статус не изменился:", "ONLINE" if is_online else "OFFLINE")

        error_sent = False

    except Exception as e:
        print("Ошибка при получении данных модели:", e)

        if not error_sent:
            send_telegram(f"⚠️ Ошибка сайта или парсинга:\n{e}", ADMIN_CHAT_ID)
            error_sent = True


# ---------- ОСНОВНОЙ ЦИКЛ ----------
while True:
    check_model()
    time.sleep(CHECK_INTERVAL)
