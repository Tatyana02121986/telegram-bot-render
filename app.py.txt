from flask import Flask, request
import telegram
import os

TOKEN = os.environ.get("BOT_TOKEN")
bot = telegram.Bot(token=TOKEN)

app = Flask(__name__)

@app.route('/')
def index():
    return 'Bot is running!'

@app.route('/webhook', methods=['POST'])
def webhook():
    update = telegram.Update.de_json(request.get_json(force=True), bot)

    if update.callback_query:
        data = update.callback_query.data
        chat_id = update.callback_query.message.chat.id

        if data.startswith('approve'):
            bot.send_message(chat_id, "✅ Утвердить")
        elif data.startswith('decline'):
            bot.send_message(chat_id, "❌ Отклонить")

        bot.answer_callback_query(update.callback_query.id)

    return 'ok'

@app.route('/send/<chat_id>')
def send_message(chat_id):
    keyboard = telegram.InlineKeyboardMarkup([
        [telegram.InlineKeyboardButton("✅ Утвердить", callback_data='approve')],
        [telegram.InlineKeyboardButton("❌ Отклонить", callback_data='decline')]
    ])
    bot.send_message(chat_id=chat_id, text="Согласуйте документ:", reply_markup=keyboard)
    return 'sent'