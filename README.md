import urllib.request
import urllib.parse
import json
import time

TOKEN = "8978257081:AAHhHF60WI2I4gMOjT4DEuFqETIrIs0MAP4"
ADMIN_ID = 6816736501

def api(method, data=None):
    url = "https://api.telegram.org/bot" + TOKEN + "/" + method

    if data:
        data = urllib.parse.urlencode(data).encode()
        req = urllib.request.Request(url, data=data)
        return json.loads(urllib.request.urlopen(req, timeout=30).read())

    return json.loads(urllib.request.urlopen(url, timeout=30).read())


def send(chat_id, text, keyboard=None):
    data = {
        "chat_id": chat_id,
        "text": text
    }

    if keyboard:
        data["reply_markup"] = json.dumps(
            keyboard,
            ensure_ascii=False
        )

    api("sendMessage", data)


def menu(chat_id):

    keyboard = {
        "keyboard": [
            [{"text": "🛍️ المنتجات"}],
            [{"text": "💰 رصيدي"}, {"text": "💳 شحن الرصيد"}],
            [{"text": "📦 طلباتي"}, {"text": "👤 حسابي"}]
        ],
        "resize_keyboard": True
    }

    if chat_id == ADMIN_ID:
        keyboard["keyboard"].append(
            [{"text": "⚙️ لوحة الأدمن"}]
        )

    send(
        chat_id,
        "🛒 أهلاً بك في متجر Zentro\n\n"
        "اختر الخدمة من القائمة 👇",
        keyboard
    )


def products(chat_id):

    send(
        chat_id,
        "🛍️ منتجات المتجر\n\n"
        "1️⃣ شدات PUBG\n"
        "2️⃣ رصيد سيريتل\n"
        "3️⃣ شحن تطبيقات\n\n"
        "🚧 المنتجات سيتم إضافتها لاحقاً."
    )


def balance(chat_id):

    send(
        chat_id,
        "💰 رصيدك الحالي\n\n"
        "0 SYP"
    )


def charge(chat_id):

    send(
        chat_id,
        "💳 شحن الرصيد\n\n"
        "قم بالتحويل إلى حساب الشحن، "
        "ثم أرسل إشعار التحويل هنا 📩\n\n"
        "سيتم مراجعة طلبك يدوياً."
    )


def orders(chat_id):

    send(
        chat_id,
        "📦 طلباتك\n\n"
        "لا توجد طلبات حالياً."
    )


def account(chat_id):

    send(
        chat_id,
        "👤 حسابك\n\n"
        f"🆔 ID: {chat_id}\n"
        "💰 الرصيد: 0 SYP"
    )


def admin(chat_id):

    if chat_id != ADMIN_ID:
        send(chat_id, "❌ هذه القائمة للأدمن فقط.")
        return

    send(
        chat_id,
        "⚙️ لوحة أدمن Zentro\n\n"
        "➕ إضافة منتجات\n"
        "💰 إدارة الأرصدة\n"
        "📦 إدارة الطلبات\n\n"
        "🚧 سيتم تفعيلها بالخطوة القادمة."
    )


# تشغيل البوت

me = api("getMe")

if not me.get("ok"):
    print("التوكن غير صحيح")
    raise SystemExit

print("Zentro Store يعمل ✅")
print("Bot:", me["result"].get("username"))

offset = 0

while True:

    try:

        result = api(
            "getUpdates",
            {
                "offset": offset,
                "timeout": 20
            }
        )

        for update in result.get("result", []):

            offset = update["update_id"] + 1

            message = update.get("message")

            if not message:
                continue

            chat_id = message["chat"]["id"]
            text = message.get("text", "")

            if text == "/start":
                menu(chat_id)

            elif text == "🛍️ المنتجات":
                products(chat_id)

            elif text == "💰 رصيدي":
                balance(chat_id)

            elif text == "💳 شحن الرصيد":
                charge(chat_id)

            elif text == "📦 طلباتي":
                orders(chat_id)

            elif text == "👤 حسابي":
                account(chat_id)

            elif text == "⚙️ لوحة الأدمن":
                admin(chat_id)

            else:
                send(
                    chat_id,
                    "❓ اختر أحد الخيارات من القائمة."
                )

    except Exception as e:

        print("Error:", e)
        time.sleep(5)
