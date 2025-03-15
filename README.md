import logging
import time
import os
from datetime import datetime, timedelta
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

# Defina seu token do BotFather
TOKEN = '6700961816:AAE92A5KjDmruwUsUd_Urmh8Y9TlH5OGMd0'

# Ativando o log para debug
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)
logger = logging.getLogger(name)

# Caminho da pasta onde estão os arquivos TXT
BASE_PATH = "/storage/emulated/0/Documents/"

def search_in_files(keyword):
    """ Procura a palavra-chave em todos os arquivos .txt no diretório. """
    start_time = time.time()
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    found_data = {}

    # Lista todos os arquivos .txt na pasta
    txt_files = [f for f in os.listdir(BASE_PATH) if f.endswith(".txt")]

    if not txt_files:
        return "❌ Nenhum arquivo .txt encontrado na pasta."

    for file_name in txt_files:
        file_path = os.path.join(BASE_PATH, file_name)
        
        try:
            with open(file_path, 'r', encoding='utf-8') as infile:
                linhas = {linha.strip() for linha in infile if keyword.lower() in linha.lower()}

            if linhas:
                found_data[file_name] = list(linhas)[:100]  # Limita a exibição a 100 linhas por arquivo

        except Exception as e:
            found_data[file_name] = [f"❌ Erro ao acessar '{file_name}': {e}"]

    if not found_data:
        return f"⚠️ Nenhum login encontrado para '{keyword}'"

    execution_time = timedelta(seconds=(time.time() - start_time))
    resposta = f"📅 Data e Hora da Busca: {now}\n🔢 Tempo de Execução: {execution_time}\n\n"

    for file, lines in found_data.items():
        resposta += f"📂 Arquivo: {file}\n" + "\n".join(lines) + "\n\n"

    return resposta[:4000]  # Limita a resposta para evitar erro do Telegram

# Função de comando /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("💻 Digite sua url")

# Função para lidar com a busca nos arquivos
async def handle_keyword(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    keyword = update.message.text.strip()

    if not keyword:
        await update.message.reply_text("⚠️ Envie uma palavra-chave para buscar!")
        return

    loading_message = await update.message.reply_text("🔍 Buscando seu long ")

    response_text = search_in_files(keyword)

    await loading_message.edit_text(response_text)

# Função principal para rodar o bot
def main():
    application = Application.builder().token(TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_keyword))

    application.run_polling()

if name == 'main':
    main()