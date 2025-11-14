# Telegram Funny AI Bot (Railway Deploy)

This is a Telegram bot that:
- Automatically replies to users with funny messages 😆
- Tracks active users
- Requires users to join a specific Telegram channel before using the bot
- Allows admin to broadcast messages to all users
- Works 24/7 on Railway

---

## ✨ Features
- 😂 Funny auto replies
- 🧾 Active user tracking
- 🔐 Channel join check
- 👑 Admin panel  
  - `/users` → Show active users  
  - `/broadcast <message>` → Send message to all users
- 🚀 Railway deploy support
- ⚙ Built using python-telegram-bot v20

---

## 📁 Project Files        if member.status not in ["member", "administrator", "creator"]:
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
