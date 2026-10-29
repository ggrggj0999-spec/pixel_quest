import os
import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, WebAppInfo
from telegram.ext import (
    Application,
    CommandHandler,
    ContextTypes,
)

logging.basicConfig(level=logging.INFO)

BOT_TOKEN = os.getenv("BOT_TOKEN")
WEB_APP_URL = "https://ggrggj0999-spec.github.io/pixel_quest/"


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [[
        InlineKeyboardButton(
            "⚔️ Играть в PIXEL QUEST",
            web_app=WebAppInfo(url=WEB_APP_URL)
        )
    ]]

    await update.message.reply_text(
        "🎮 PIXEL QUEST\n\n"
        "Маленький герой. Большая легенда!\n\n"
        "Нажми кнопку ниже, чтобы открыть игру.",
        reply_markup=InlineKeyboardMarkup(keyboard)
    )


async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "📜 Команды PIXEL QUEST:\n"
        "/start — открыть игру\n"
        "/help — помощь\n"
        "/app — открыть игру"
    )


async def app_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await start(update, context)


def main():
    if not BOT_TOKEN:
        raise RuntimeError("Не задан BOT_TOKEN")

    app = Application.builder().token(BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_command))
    app.add_handler(CommandHandler("app", app_command))

    print("PIXEL QUEST bot is starting...")
    app.run_polling()


if __name__ == "__main__":
    main()
