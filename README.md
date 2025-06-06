import logging
from telegram import Update, ChatMember
from telegram.ext import Application, CommandHandler, MessageHandler, ContextTypes, filters

BOT_TOKEN = "6776859888:AAFfuWtAjrKXZj07-47cjr_IAKXEE28kVgI"
CHANNEL_USERNAME = "@gelik_uzb_o1"  

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Kanalga a'zo ekanligini tekshirish funksiyasi
async def is_user_subscribed(user_id: int, context: ContextTypes.DEFAULT_TYPE) -> bool:
    try:
        member = await context.bot.get_chat_member(CHANNEL_USERNAME, user_id)
        return member.status in [ChatMember.MEMBER, ChatMember.ADMINISTRATOR, ChatMember.OWNER]
    except Exception as e:
        logging.error(f"Kanal a'zo tekshirish xatoligi: {e}")
        return False

# /start buyrug'i
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if not await is_user_subscribed(user_id, context):
        await update.message.reply_text(
            f"❌ Siz kanalga a'zo emassiz!\n"
            f"Iltimos, kanalga a'zo bo‘ling: {CHANNEL_USERNAME}\n"
            f"Keyin /start ni qayta bosing."
        )
    else:
        await update.message.reply_text("👋 Salom! Kino kodini yuboring.")

# /help buyrug'i
async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🆘 Yordam bo‘limi:\n"
        "Admin bilan bog‘lanish: @z1yodullayev_777\n"
        "/start - Boshlash\n"
        "/save - Kino kodi yuborish\n"
    )

# /save buyrug'i
async def save(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if not await is_user_subscribed(user_id, context):
        await update.message.reply_text(
            f"❌ Kino kodini faqat kanal a’zolari yuborishi mumkin.\n"
            f"Iltimos, kanalga a’zo bo‘ling: {CHANNEL_USERNAME}"
        )
        return
    await update.message.reply_text("🎬 Kino kodini yuboring:")

# Kino kodi kelganda ishlovchi funksiya
async def handle_movie_code(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if not await is_user_subscribed(user_id, context):
        await update.message.reply_text(
            f"❌ Kino kodini olish uchun kanalga a’zo bo‘ling: {CHANNEL_USERNAME}"
        )
        return

    code = update.message.text.strip()

    if code == "1":
        video_link = "https://t.me/gelik_uzb_o1/45("
        # Video yuborish uchun reply_video ishlatilmaydi, chunki tg:// link video emas, xabar linki
        # Shuning uchun oddiy text sifatida yuboramiz
        await update.message.reply_text(
            f"🎬 Siz so‘ragan video: {video_link}"
        )
    else:
        await update.message.reply_text(
            "❗ Bunday kino kodi topilmadi. Iltimos, boshqa kod yuboring."
        )

def main():
    app = Application.builder().token(BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_command))
    app.add_handler(CommandHandler("save", save))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_movie_code))

    logging.info("Bot ishga tushdi.")
    app.run_polling()

if __name__ == "__main__":
    main()
