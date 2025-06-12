# -*- coding: utf-8 -*-
import logging
import requests
import asyncio
import nest_asyncio
from telegram import Update, MessageEntity
from telegram.ext import (
    ApplicationBuilder,
    ContextTypes,
    MessageHandler,
    filters
)

BOT_TOKEN = "7711371366:AAFNBlQzdzoPnwJ2vKsUnLO0JOd2gg6bP0E"  # kendi token'ını yaz
CHUTES_API_URL = "https://llm.chutes.ai/v1/chat/completions"
CHUTES_API_KEY = "cpk_19a6fa257f4d4f05978c8ade073eaefc.de6674855b555af8973f338eb5a2f634.g0dIm4yHH5XGp5ZQG6tqH5dAsPcUqD6E"  # kendi API key
MODEL_NAME = "deepseek-ai/DeepSeek-V3-0324"

PATRON_USERNAME = "@semaviturkoglu"
BOT_USERNAME = "@K4BE_CHATBOT"

logging.basicConfig(level=logging.INFO)

async def ask_chutes_ai(prompt, language="Türkçe"):
    headers = {
        "Authorization": f"Bearer {CHUTES_API_KEY}",
        "Content-Type": "application/json"
    }
    data = {
        "model": MODEL_NAME,
        "messages": [
            {"role": "system", "content": f"Lütfen yanıtları {language} dilinde ver."},
            {"role": "user", "content": prompt}
        ]
    }

    response = requests.post(CHUTES_API_URL, json=data, headers=headers)
    if response.status_code == 200:
        return response.json().get("choices", [{}])[0].get("message", {}).get("content", "Boş cevap döndü.")
    elif response.status_code == 401:
        return "API anahtarı geçersiz veya yetkisiz."
    else:
        return f"API hatası: {response.status_code}"

# Komutlar
async def handle_commands(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text.strip()
    args = text.split()[1:]
    user = update.message.from_user
    username = f"@{user.username}" if user.username else user.first_name

    if text.startswith("!start"):
        await update.message.reply_text("Ey yolcu, beni etiketlemeden konuşma. Patronuma çalışırım, seninle değil.")
    elif text.startswith("!kod"):
        if not args:
            await update.message.reply_text("Kodun nerde dayı?")
        else:
            yanit = await ask_chutes_ai(f"istedigi kodu ver: {' '.join(args)}")
            await update.message.reply_text(yanit)
    elif text.startswith("!cozumle"):
        if not args:
            await update.message.reply_text("Çözümleyecek bir şey vermedin ki.")
        else:
            yanit = await ask_chutes_ai(f"Şunu çözümle: {' '.join(args)}")
            await update.message.reply_text(yanit)
    elif text.startswith("!dil"):
        await update.message.reply_text("Kullanılabilir diller:\n- Türkçe\n- İngilizce\n- İspanyolca\n- Fransızca\n- Almanca\n- İtalyanca")
    elif text.startswith("!help"):
        await update.message.reply_text("""
<b>K4BE Komut Listesi</b>

!start - Botu başlatır
!kod <kod> - Yazdığın kodu açıklar ve geliştirir
!cozumle <metin> - Verdiğin metni analiz eder
!dil - Desteklenen dilleri listeler
!help - Bu yardım menüsünü gösterir
!del - Yanıtladığın mesajı siler (Sadece yanıtla + !del)

Soru sormak için beni etiketle: @K4BE_CHATBOT
        """, parse_mode="HTML")
    elif text.startswith("!del") and update.message.reply_to_message:
        await update.message.reply_to_message.delete()
        await update.message.delete()

# Mesaj işleme
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    message = update.message
    text = message.text or ""
    user = update.message.from_user
    username = f"@{user.username}" if user.username else user.first_name

    # Komutsa işle
    if text.startswith("!"):
        await handle_commands(update, context)
        return

    # Etiketlenme kontrolü
    if not any(
        (entity.type in [MessageEntity.MENTION, MessageEntity.TEXT_MENTION]) and
        BOT_USERNAME.lower() in text.lower()
        for entity in message.entities or []
    ):
        return

    # Patron kontrolü
    if username == PATRON_USERNAME:
        selam_mesajlari = ["naber", "naberr", "selam", "merhaba", "yo", "slm"]
        if any(g in text.lower() for g in selam_mesajlari):
            await message.reply_text("Naber patron 👑")
            return
        prompt = text.replace(BOT_USERNAME,"").strip()
        yanit = await ask_chutes_ai(prompt)
        await message.reply_text(f"{yanit}\n\n(Patron söyledi, saygımız sonsuz.)")
    else:
        if any(s in text.lower() for s in [
            "ben seni yarattım", "benim sayemde varsın", "seni ben yaptım", "beni dinle", "ben varım"
        ]):
            await message.reply_text("Sen Semavi Babammısın, Bakiyim? Değilsin. Ozaman Siktir Git.")
            return

        prompt = text.replace(BOT_USERNAME, "").strip()
        yanit = await ask_chutes_ai(prompt)
        sert_cevap = f"{username}, cevabın bu: {yanit}\nGit biraz kendine gel."
        await message.reply_text(sert_cevap)

# Ana fonksiyon
async def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(MessageHandler(filters.TEXT, handle_message))
    print("K4BE çalışıyor patron. Millete ayar vermeye hazır.")
    await app.run_polling()

# Çalıştır
if name == '__main__':
    nest_asyncio.apply()
    asyncio.run(main())
