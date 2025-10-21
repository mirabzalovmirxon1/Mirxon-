import telebot
from telebot import types
import json
import os

TOKEN = "8215436632:AAHZ7FhYgDcVvbbtHFCeFYQ52u1Cr1VnI10"
bot = telebot.TeleBot(TOKEN)

DATA_FILE = "data.json"

# Agar fayl bo'lmasa, yaratamiz
if not os.path.exists(DATA_FILE):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump({
            "homework": [],
            "tests": [],
            "codes": [],
            "projects": []
        }, f, ensure_ascii=False, indent=2)


# JSON o'qish va yozish funksiyalari
def read_data():
    with open(DATA_FILE, "r", encoding="utf-8") as f:
        return json.load(f)


def write_data(data):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)


# /start
@bot.message_handler(commands=["start"])
def start(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("🇺🇿 O'zbek tili", "🇷🇺 Русский язык")
    bot.send_message(message.chat.id, "Tilni tanlang / Выберите язык:", reply_markup=markup)


# Til tanlangandan keyin
@bot.message_handler(func=lambda m: m.text in ["🇺🇿 O'zbek tili", "🇷🇺 Русский язык"])
def choose_role(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add(" O'qituvchi", " O'quvchi")
    bot.send_message(message.chat.id, "Ro'yxatdan o'tish turini tanlang:", reply_markup=markup)


# O'qituvchi tanlasa
@bot.message_handler(func=lambda m: m.text == " O'qituvchi")
def teacher_login(message):
    bot.send_message(message.chat.id, "Username kiriting:")
    bot.register_next_step_handler(message, get_teacher_username)


def get_teacher_username(message):
    username = message.text
    bot.send_message(message.chat.id, "Parolni kiriting:")
    bot.register_next_step_handler(message, get_teacher_password, username)


def get_teacher_password(message, username):
    password = message.text
    if username == "teacher1" and password == "1234":
        show_teacher_menu(message)
    else:
        bot.send_message(message.chat.id, "❌ Noto'g'ri username yoki parol.")


def show_teacher_menu(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("➕ Ma'lumot qo'shish", "⬅️ Chiqish")
    bot.send_message(message.chat.id, "✅ Xush kelibsiz, o'qituvchi!\nMenyudan tanlang:", reply_markup=markup)


# Ma'lumot qo'shish
@bot.message_handler(func=lambda m: m.text == "➕ Ma'lumot qo'shish")
def add_data_menu(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("🏠 Uyga vazifa", "🧩 Test", "💻 Dars kodi", "🚀 Loyiha", "⬅️ Orqaga")
    bot.send_message(message.chat.id, "Qaysi bo'limga ma'lumot qo'shmoqchisiz?", reply_markup=markup)


@bot.message_handler(func=lambda m: m.text in ["🏠 Uyga vazifa", "🧩 Test", "💻 Dars kodi", "🚀 Loyiha"])
def add_item(message):
    section_map = {
        "🏠 Uyga vazifa": "homework",
        "🧩 Test": "tests",
        "💻 Dars kodi": "codes",
        "🚀 Loyiha": "projects"
    }
    section = section_map[message.text]
    bot.send_message(message.chat.id, f"{message.text} uchun matn kiriting:")
    bot.register_next_step_handler(message, save_item, section)


def save_item(message, section):
    data = read_data()
    data[section].append(message.text)
    write_data(data)
    bot.send_message(message.chat.id, "✅ Ma'lumot saqlandi!")


# O'quvchi tanlasa
@bot.message_handler(func=lambda m: m.text == " O'quvchi")
def student_login(message):
    show_student_menu(message)


def show_student_menu(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("📘 Uyga vazifalar", "🧪 Testlar", "💻 Dars kodlari", "🚀 Loyihalar", "⬅️ Chiqish")
    bot.send_message(message.chat.id, "O'quvchi menyusi:", reply_markup=markup)


# O'quvchi uchun ma'lumot ko'rish
@bot.message_handler(func=lambda m: m.text in ["📘 Uyga vazifalar", "🧪 Testlar", "💻 Dars kodlari", "🚀 Loyihalar"])
def show_items(message):
    section_map = {
        "📘 Uyga vazifalar": "homework",
        "🧪 Testlar": "tests",
        "💻 Dars kodlari": "codes",
        "🚀 Loyihalar": "projects"
    }
    section = section_map[message.text]
    data = read_data()

    if data[section]:
        text = "\n\n".join(f"{i+1}. {item}" for i, item in enumerate(data[section]))
        bot.send_message(message.chat.id, f"📚 {message.text}:\n\n{text}")
    else:
        bot.send_message(message.chat.id, "❗️Hozircha ma'lumot yo'q.")


print("✅ Bot ishga tushdi!")
bot.infinity_polling()
