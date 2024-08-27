# Tg_Tron_MySnipingBot

```
sudo apt update
sudo apt install python3 python3-pip -y
pip install tronpy requests python-telegram-bot

```
```
screen -S my_sniping_bot
nano my_sniping_bot.py
```

```
from tronpy import Tron
import requests
import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes

# Logging configuration
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

# Connecting to the Tron blockchain
client = Tron()

# Bot token
TELEGRAM_TOKEN = 'YOUR_TELEGRAM_BOT_TOKEN'

# User ID (should be defined as an integer)
ADMIN_ID = YOUR_ADMIN_ID

# Start the Telegram bot
application = Application.builder().token(TELEGRAM_TOKEN).build()

# Function to get information about a specific token on TRON
async def check_token(update: Update, context: ContextTypes.DEFAULT_TYPE):
    logger.info("Check token command executed.")
    
    # Check user ID
    if update.effective_user.id != ADMIN_ID:
        await update.message.reply_text("You do not have permission to use this command.")
        return

    # Get the token address
    if not context.args:
        await update.message.reply_text("No token address provided.")
        return
    
    token_address = context.args[0]
    url = f"https://apilist.tronscan.org/api/token_trc20/{token_address}"

    try:
        response = requests.get(url)
        response.raise_for_status()  # Triggers HTTP errors
        data = response.json()

        if 'contractMap' in data:
            contract_info = data['contractMap']
            locked_liquidity = contract_info.get('liquidityLocked', False)

            if locked_liquidity:
                await update.message.reply_text(f"Token {token_address} appears to be secure. Liquidity is locked.")
            else:
                await update.message.reply_text(f"Token {token_address} is risky. Liquidity is not locked.")
        else:
            await update.message.reply_text("Token information not found.")
    except requests.exceptions.RequestException as e:
        logger.error(f"HTTP error: {e}")
        await update.message.reply_text(f"HTTP error: {e}")
    except ValueError as e:
        logger.error(f"JSON error: {e}")
        await update.message.reply_text(f"JSON error: {e}")

# Add command handlers
application.add_handler(CommandHandler("check_token", check_token))

# Run the bot
if __name__ == "__main__":
    logger.info("Bot is starting...")
    application.run_polling()

```

```
python3 my_sniping_bot.py
```
for control

```
/check_token contract adress
```
