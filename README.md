from telegram import Update, KeyboardButton, ReplyKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
import requests
from bs4 import BeautifulSoup

BOT_TOKEN = "8408645381:AAH7fPaXKlK7colPFOLYYSvAG5F-nXXPAZw"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [KeyboardButton("1) Online Market Bot")],
        [KeyboardButton("2) 3 ta sahifadan Parsing")]
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Iltimos, kerakli bo'limni tanlang:", reply_markup=reply_markup)

async def online_market(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [KeyboardButton("O'zbek tili"), KeyboardButton("Русский"), KeyboardButton("English")],
        [KeyboardButton("Orqaga")]
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Tilni tanlang:", reply_markup=reply_markup)

async def market_menu(update: Update, context: ContextTypes.DEFAULT_TYPE, lang: str):
    if lang == "uz":
        keyboard = [
            [KeyboardButton("Mahsulotlar"), KeyboardButton("Aksiya")],
            [KeyboardButton("Yordam"), KeyboardButton("Orqaga")]
        ]
        text = "Siz O'zbek tilini tanladingiz. Menyudan birini tanlang:"
    elif lang == "ru":
        keyboard = [
            [KeyboardButton("Товары"), KeyboardButton("Акции")],
            [KeyboardButton("Помощь"), KeyboardButton("Назад")]
        ]
        text = "Вы выбрали русский язык. Выберите раздел:"
    else:
        keyboard = [
            [KeyboardButton("Products"), KeyboardButton("Sales")],
            [KeyboardButton("Help"), KeyboardButton("Back")]
        ]
        text = "You selected English. Choose a section:"

    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text(text, reply_markup=reply_markup)

async def parsing(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Parsing boshlanmoqda...")

    urls = [
        "https://asaxiy.uz/product/kompyutery-i-orgtehnika/noutbuki/noutbuki-2",
        "https://asaxiy.uz/product/telefony-i-gadzhety/smartfony",
        "https://asaxiy.uz/product/televizory-i-video/televizory"
    ]

    all_data = []
    for url in urls:
        try:
            response = requests.get(url, headers={"User-Agent": "Mozilla/5.0"})
            soup = BeautifulSoup(response.text, "html.parser")
            products = soup.find_all("div", class_="product__item")

            for p in products[:5]: 
                name = p.find("span", class_="product__item__info-title")
                price = p.find("span", class_="product__item-price")

                name = name.text.strip() if name else "Nom topilmadi"
                price = price.text.strip() if price else "Narx topilmadi"

                all_data.append(f"{name} - {price}")
        except Exception as e:
            all_data.append(f"Xatolik: {e}")

    if all_data:
        await update.message.reply_text("\n".join(all_data[:20]))
    else:
        await update.message.reply_text("Ma'lumot topilmadi.")

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text

    if text == "1) Online Market Bot":
        await online_market(update, context)
    elif text == "2) 3 ta sahifadan Parsing":
        await parsing(update, context)
    elif text == "Orqaga" or text == "Назад" or text == "Back":
        await start(update, context)

    elif text == "O'zbek tili":
        await market_menu(update, context, "uz")
    elif text == "Русский":
        await market_menu(update, context, "ru")
    elif text == "English":
        await market_menu(update, context, "en")

    elif text in ["Mahsulotlar", "Товары", "Products"]:
        await update.message.reply_text("Hozircha mahsulotlar mavjud emas.")
    elif text in ["Aksiya", "Акции", "Sales"]:
        await update.message.reply_text("Hozircha aksiyalar mavjud emas.")
    elif text in ["Yordam", "Помощь", "Help"]:
        await update.message.reply_text("Yordam uchun admin bilan bog'laning.")

    else:
        await update.message.reply_text("Noto'g'ri buyruq. /start buyrug'ini bosing.")


def main():
    app = Application.builder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    application = Application.builder().token(BOT_TOKEN).read_timeout(30).connect_timeout(30).build()

    print("Bot ishga tushdi...")
    app.run_polling()

if __name__ == "__main__":
    main()
