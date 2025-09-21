# telegram-copilot-bot
Telegram-бот, который помогает с написанием и редактированием текстов с помощью нейросети Google Gemini.
[tgbot.py](https://github.com/user-attachments/files/22452894/tgbot.py)
import google.generativeai as genai
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes
import os

# Твой API-КЛЮЧ GEMINI
genai.configure(api_key="AIzaSyAyjZPFRlyfdGbbIrpc5FrtTHEhJOYu5tw")

# ТВОЙ ТОКЕН БОТА ИЗ BOTFATHER
BOT_TOKEN = "8336489533:AAGNuViOVh9dmjnWJ5FGG_GqiijySFNR8xQ"

# Выбери модель, которая у тебя работает
model = genai.GenerativeModel('models/gemini-1.5-flash-latest')

# Создаем словарь для хранения сессий чата для каждого пользователя
# Ключ: ID пользователя, Значение: сессия чата
chat_sessions = {}

# Команда /start
async def start_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    # Создаем новую сессию с инструкциями
    if update.effective_user.id not in chat_sessions:
        chat_sessions[update.effective_user.id] = model.start_chat(
            history=[
                {"role": "user", "parts": "Ты — профессиональный копирайтер. Твоя задача — помогать пользователю с написанием текстов. Отвечай кратко, по делу и в деловом стиле. Можешь генерировать статьи, посты, идеи для контента и редактировать тексты, которые тебе присылают. Всегда спрашивай, чем ты можешь помочь, в конце каждого ответа."},
                {"role": "model", "parts": "Отлично, я готов к работе. Чем я могу помочь?"},
            ]
        )
    
    await update.message.reply_text("Привет! Я — твой бот-помощник для написания текстов. Просто напиши, что нужно, и я помогу.")
# Обработчик текстовых сообщений
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    user_message = update.message.text
    
    # Получаем сессию для текущего пользователя
    if user_id not in chat_sessions:
        chat_sessions[user_id] = model.start_chat(history=[])
    
    chat = chat_sessions[user_id]

    try:
        response = chat.send_message(user_message)
        await update.message.reply_text(response.text)
    except Exception as e:
        print(f"Ошибка: {e}")
        await update.message.reply_text("Извини, произошла ошибка. Попробуй ещё раз.")

# Основная функция, которая запускает бота
def main():
    print("Бот запущен...")
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    # Добавляем обработчики
    app.add_handler(CommandHandler("start", start_command))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    app.run_polling()

if __name__ == "__main__":
    main()
