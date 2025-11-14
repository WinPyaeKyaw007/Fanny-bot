from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, filters, ContextTypes
import random

BOT_TOKEN = "YOUR_BOT_TOKEN"
OWNER_ID = 1794465007
CHANNEL_ID = "@G_Fatt_Music"

active_users = set()

FUNNY_REPLIES = [
    "ဟုတ်啦 🤣 AI Brain.exe ကိုကလည်း နောက်ကျသွားသလို!",
    "Wait wait... 🤔 စဉ်းစားတယ်—ဟုတ်ပေမဲ့ မကြာခင် ပြန်ပြောမယ်!",
    "အေးပေမယ့် မင်းကတော့ Question Machine လေးပဲ 🤭",
    "ကံကောင်းတယ် AI မဆန်လောက်တော့် ပြန်မပြောဘူးဟာ 😆",
    "ဒါမျိုးမေးတာ မင်းတင်ပဲ bro 😂"
]


async def check_join(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    active_users.add(user.id)

    try:
        member = await context.bot.get_chat_member(CHANNEL_ID, user.id)
        if member.status not in ["member", "administrator", "creator"]:
            await update.message.reply_text(
                "👋 Bot သုံးနိုင်ဖို့ ဒီ Channel ကို Join ပါ👇\n"
                f"➡️ https://t.me/{CHANNEL_ID[1:]}"
            )
            return False
    except:
        pass

    return True


async def funny_reply(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_join(update, context):
        return
    await update.message.reply_text(random.choice(FUNNY_REPLIES))


async def admin_panel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != OWNER_ID:
        return
    
    text = update.message.text
    
    if text == "/users":
        await update.message.reply_text(f"Active users: {len(active_users)}\n{active_users}")

    if text.startswith("/broadcast "):
        message = text.replace("/broadcast ", "")
        for uid in active_users:
            try:
                await context.bot.send_message(uid, f"📢 Admin Broadcast:\n\n{message}")
            except:
                pass
        await update.message.reply_text("✔️ Message sent to all active users!")


app = ApplicationBuilder().token(BOT_TOKEN).build()

app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, funny_reply))
app.add_handler(MessageHandler(filters.COMMAND, admin_panel))

app.run_polling()
