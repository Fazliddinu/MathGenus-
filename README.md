# MathGenus-
Telegram bot
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes
import sympy as sp
import logging

# Premium foydalanuvchilar ro'yxati
premium_users = []

# Admin Telegram ID va username
ADMIN_ID = 7849481708  # Adminning Telegram ID
ADMIN_USERNAME = "isroilovbrat"  # Adminning Telegram username

# Loggerni sozlash
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Foydalanuvchiga xush kelibsiz xabarini yuboradi va menyuni ko‘rsatadi."""
    keyboard = [
        [InlineKeyboardButton("Oddiy Kalkulyator", callback_data="basic")],
        [InlineKeyboardButton("Premium Bo'lim", callback_data="premium")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("Salom! Botga xush kelibsiz. Birini tanlang:", reply_markup=reply_markup)

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Menyudan tanlangan bo‘limni boshqaradi."""
    query = update.callback_query
    await query.answer()

    if query.data == "basic":
        await query.edit_message_text(
            "Oddiy Kalkulyatorga xush kelibsiz!\nFoydalanish uchun arifmetik misolni yuboring.\n"
            "Masalan: `12 + 34`, `56 * 78` yoki `90 / 5`.",
        )
    elif query.data == "premium":
        if query.from_user.id in premium_users:
            await query.edit_message_text(
                "Premium bo‘limga xush kelibsiz!\nAlgebra va geometriya misollarini yuboring.\n"
                "Masalan:\n- Tenglama: `solve(x**2 - 5*x + 6)`\n"
                "- Aylana yuzi: `pi*r**2`"
            )
        else:
            await query.edit_message_text(
                f"Premium bo‘limga ulanish uchun admin bilan bog‘laning: [Admin](https://t.me/{ADMIN_USERNAME})",
                parse_mode="Markdown"
            )

async def solve_basic_math(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Oddiy arifmetik amallarni bajaradi."""
    expression = update.message.text
    try:
        result = eval(expression)
        await update.message.reply_text(f"Natija: {result}")
    except Exception as e:
        await update.message.reply_text(f"Xatolik yuz berdi: {e}")

async def solve_premium_math(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Premium bo‘lim uchun matematik masalalar yechadi."""
    user_id = update.message.from_user.id
    if user_id in premium_users:
        expression = update.message.text
        try:
            result = sp.sympify(expression)
            await update.message.reply_text(f"Natija: {result}")
        except Exception as e:
            await update.message.reply_text(f"Xatolik yuz berdi: {e}")
    else:
        await update.message.reply_text("Siz premium foydalanuvchi emassiz. Iltimos, admin bilan bog‘laning.")

async def add_premium(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Admin tomonidan foydalanuvchini premiumga qo‘shish."""
    if update.message.from_user.id == ADMIN_ID:
        try:
            user_id = int(context.args[0])
            if user_id not in premium_users:
                premium_users.append(user_id)
                await update.message.reply_text(f"Foydalanuvchi {user_id} premiumga qo‘shildi.")
                await context.bot.send_message(chat_id=user_id, text="Tabriklaymiz! Siz Premium rejimga ulandingiz!")
            else:
                await update.message.reply_text(f"Foydalanuvchi {user_id} allaqachon premiumda.")
        except (IndexError, ValueError):
            await update.message.reply_text("Foydalanuvchi ID sini kiriting: /addpremium <user_id>")
    else:
        await update.message.reply_text("Sizda bunday huquq yo‘q.")

async def remove_premium(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Admin tomonidan foydalanuvchini premiumdan olib tashlash."""
    if update.message.from_user.id == ADMIN_ID:
        try:
            user_id = int(context.args[0])
            if user_id in premium_users:
                premium_users.remove(user_id)
                await update.message.reply_text(f"Foydalanuvchi {user_id} premiumdan olib tashlandi.")
            else:
                await update.message.reply_text(f"Foydalanuvchi {user_id} premiumda emas.")
        except (IndexError, ValueError):
            await update.message.reply_text("Foydalanuvchi ID sini kiriting: /removepremium <user_id>")
    else:
        await update.message.reply_text("Sizda bunday huquq yo‘q.")

def main() -> None:
    """Botni ishga tushiradi."""
    application = ApplicationBuilder().token('7849481708:AAFJDvSKDOvrcxF8p4JxgYxhOozhvwNPAUg').build()

    application.add_handler(CommandHandler('start', start))
    application.add_handler(CallbackQueryHandler(button_handler))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, solve_premium_math))
    application.add_handler(CommandHandler('addpremium', add_premium))
    application.add_handler(CommandHandler('removepremium', remove_premium))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, solve_basic_math))

    application.run_polling()

if __name__ == '__main__':
