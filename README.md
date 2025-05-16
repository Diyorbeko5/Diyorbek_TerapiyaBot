# Diyorbek_TerapiyaBot
import os
from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton
from aiogram.fsm.storage.memory import MemoryStorage
from aiogram.fsm.context import FSMContext
import asyncio
import json

API_TOKEN = os.getenv("API_TOKEN")

bot = Bot(token=API_TOKEN)
dp = Dispatcher(storage=MemoryStorage())

questions = [
    "Bugun o'zingizni qanday his qilyapsiz?",
    "Bugun nimadan xursand bo'ldingiz?",
    "Bugun qaysi vazifani bajardingiz?",
    # ... yana savollar
]

user_answers = {}

@dp.message(Command(commands=["start"]))
async def start_handler(message: types.Message):
    user_answers[message.from_user.id] = []
    await message.answer("Salom! Bugungi psixologik savollar bilan tanishing.")
    await ask_question(message.from_user.id, 0)

async def ask_question(user_id, index):
    if index < len(questions):
        await bot.send_message(user_id, questions[index])
    else:
        await bot.send_message(user_id, "Savollar tugadi, rahmat!")

@dp.message()
async def answer_handler(message: types.Message, state: FSMContext):
    user_id = message.from_user.id
    if user_id not in user_answers:
        await message.answer("Iltimos, /start buyrug'ini bosing.")
        return
    user_answers[user_id].append(message.text)
    current_index = len(user_answers[user_id])
    if current_index < len(questions):
        await ask_question(user_id, current_index)
    else:
        # Javoblarni saqlash
        with open(f"{user_id}_answers.json", "w") as f:
            json.dump(user_answers[user_id], f)
        await message.answer("Javoblaringiz uchun rahmat!")

if __name__ == "__main__":
    import asyncio
    asyncio.run(dp.start_polling(bot))
