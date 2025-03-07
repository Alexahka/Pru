import os
import random
from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.utils import executor
from dotenv import load_dotenv

# Загрузка переменных окружения
load_dotenv()
API_TOKEN = os.getenv('TELEGRAM_BOT_TOKEN')

# Инициализация бота
bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# База данных для хранения состояния игрока
user_data = {}

# Стартовая команда
@dp.message_handler(commands=['start'])
async def start(message: types.Message):
    user_id = message.from_user.id
    if user_id not in user_data:
        user_data[user_id] = {
            'coins': 100,  # Начальный баланс монет
            'cards': [],   # Колода игрока
            'bet': 0       # Текущая ставка
        }
    
    await message.answer(
        "Добро пожаловать в игру 'Секретные карты'! 🃏\n"
        "У вас есть демо-счет с 100 монетами.\n"
        "Используйте команды для игры.",
        reply_markup=get_main_menu()
    )

# Главное меню
def get_main_menu():
    keyboard = InlineKeyboardMarkup(row_width=2)
    buttons = [
        InlineKeyboardButton("Начать игру", callback_data="start_game"),
        InlineKeyboardButton("Обменник монет", callback_data="exchange_coins"),
        InlineKeyboardButton("Мои монеты", callback_data="show_balance")
    ]
    keyboard.add(*buttons)
    return keyboard

# Обработчик кнопок главного меню
@dp.callback_query_handler(lambda c: c.data in ["start_game", "exchange_coins", "show_balance"])
async def handle_main_menu(callback_query: types.CallbackQuery):
    user_id = callback_query.from_user.id
    data = user_data[user_id]

    if callback_query.data == "start_game":
        await start_game(callback_query.message, user_id, data)
    elif callback_query.data == "exchange_coins":
        await exchange_coins(callback_query.message, user_id, data)
    elif callback_query.data == "show_balance":
        await show_balance(callback_query.message, user_id, data)

# Показать баланс
async def show_balance(message: types.Message, user_id, data):
    await message.answer(f"Ваш баланс: {data['coins']} монет 💰")

# Обменник монет
async def exchange_coins(message: types.Message, user_id, data):
    keyboard = InlineKeyboardMarkup(row_width=2)
    buttons = [
        InlineKeyboardButton("Купить монеты", callback_data="buy_coins"),
        InlineKeyboardButton("Продать монеты", callback_data="sell_coins"),
        InlineKeyboardButton("Назад", callback_data="back_to_menu")
    ]
    keyboard.add(*buttons)
    await message.answer("Выберите действие:", reply_markup=keyboard)

# Начать игру
async def start_game(message: types.Message, user_id, data):
    if data['coins'] < 10:
        await message.answer("Недостаточно монет для игры! Минимальная ставка: 10 монет.")
        return

    # Создаем колоду карт
    deck = create_deck()
    random.shuffle(deck)

    # Раздаем карты игроку и дилеру
    player_cards = [deck.pop(), deck.pop()]
    dealer_cards = [deck.pop(), deck.pop()]

    # Сохраняем состояние игры
    data['cards'] = {'player': player_cards, 'dealer': dealer_cards, 'deck': deck}
    data['bet'] = 10  # Фиксированная ставка
    data['coins'] -= data['bet']

    # Отправляем карты и кнопки действий
    await message.answer(
        f"Ваши карты: {format_cards(player_cards)}\n"
        f"Карта дилера: {format_cards([dealer_cards[0]])}\n\n"
        f"Ставка: {data['bet']} монет",
        reply_markup=get_game_menu()
    )

# Меню игры
def get_game_menu():
    keyboard = InlineKeyboardMarkup(row_width=2)
    buttons = [
        InlineKeyboardButton("Взять карту", callback_data="hit"),
        InlineKeyboardButton("Остановиться", callback_data="stand"),
        InlineKeyboardButton("Сдаться", callback_data="surrender")
    ]
    keyboard.add(*buttons)
    return keyboard

# Обработчик действий в игре
@dp.callback_query_handler(lambda c: c.data in ["hit", "stand", "surrender"])
async def handle_game_actions(callback_query: types.CallbackQuery):
    user_id = callback_query.from_user.id
    data = user_data[user_id]
    game_data = data['cards']
    player_cards = game_data['player']
    dealer_cards = game_data['dealer']
    deck = game_data['deck']

    if callback_query.data == "hit":
        # Игрок берет карту
        player_cards.append(deck.pop())
        if calculate_score(player_cards) > 21:
            await callback_query.message.answer(
                f"Вы проиграли! Ваши карты: {format_cards(player_cards)} (очки: {calculate_score(player_cards)})",
                reply_markup=get_main_menu()
            )
            return
        await callback_query.message.edit_text(
            f"Ваши карты: {format_cards(player_cards)}\n"
            f"Карта дилера: {format_cards([dealer_cards[0]])}",
            reply_markup=get_game_menu()
        )
    elif callback_query.data == "stand":
        # Игрок останавливается
        while calculate_score(dealer_cards) < 17:
            dealer_cards.append(deck.pop())
        
        player_score = calculate_score(player_cards)
        dealer_score = calculate_score(dealer_cards)

        result = determine_winner(player_score, dealer_score)
        if result == "win":
            data['coins'] += data['bet'] * 2
            await callback_query.message.answer(
                f"Вы выиграли! 🎉\n"
                f"Ваши карты: {format_cards(player_cards)} (очки: {player_score})\n"
                f"Карты дилера: {format_cards(dealer_cards)} (очки: {dealer_score})",
                reply_markup=get_main_menu()
            )
        elif result == "lose":
            await callback_query.message.answer(
                f"Вы проиграли! 😔\n"
                f"Ваши карты: {format_cards(player_cards)} (очки: {player_score})\n"
                f"Карты дилера: {format_cards(dealer_cards)} (очки: {dealer_score})",
                reply_markup=get_main_menu()
            )
        else:
            data['coins'] += data['bet']
            await callback_query.message.answer(
                f"Ничья! 🤝\n"
                f"Ваши карты: {format_cards(player_cards)} (очки: {player_score})\n"
                f"Карты дилера: {format_cards(dealer_cards)} (очки: {dealer_score})",
                reply_markup=get_main_menu()
            )
    elif callback_query.data == "surrender":
        # Игрок сдается
        data['coins'] += data['bet'] // 2
        await callback_query.message.answer(
            "Вы сдались. Половина ставки возвращена.",
            reply_markup=get_main_menu()
        )

# Вспомогательные функции
def create_deck():
    suits = ['♠️', '♥️', '♣️', '♦️']
    ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A']
    return [f"{rank}{suit}" for suit in suits for rank in ranks]

def format_cards(cards):
    return ", ".join(cards)

def calculate_score(cards):
    score = 0
    aces = 0
    for card in cards:
        rank = card[:-2]
        if rank.isdigit():
            score += int(rank)
        elif rank in ['J', 'Q', 'K']:
            score += 10
        elif rank == 'A':
            aces += 1
            score += 11
    while score > 21 and aces:
        score -= 10
        aces -= 1
    return score

def determine_winner(player_score, dealer_score):
    if player_score > 21:
        return "lose"
    if dealer_score > 21 or player_score > dealer_score:
        return "win"
    if player_score < dealer_score:
        return "lose"
    return "draw"

# Запуск бота
if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
