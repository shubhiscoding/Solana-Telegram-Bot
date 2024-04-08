import json
import requests
from solana.rpc.api import Client

# Define your tokens
TELEGRAM_BOT_TOKEN = '6887834486:AAFKTrtNt875erdO46dZAkJ-Dj8u76vSSdU'
SOLANA_RPC_URL = 'https://api.mainnet-beta.solana.com'

# Initialize Solana client
solana_client = Client(SOLANA_RPC_URL)

# Function to get updates using long polling
def get_updates(offset=None, limit=100, timeout=0):
    url = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/getUpdates'
    params = {'offset': offset, 'limit': limit, 'timeout': timeout}
    response = requests.get(url, params=params)
    if response.status_code == 200:
        return response.json().get('result', [])
    else:
        print(f"Failed to fetch updates. Status code: {response.status_code}")
        return []

# Function to send a message
def send_message(chat_id, text):
    url = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage'
    data = {'chat_id': chat_id, 'text': text}
    response = requests.post(url, data=data)
    if response.status_code != 200:
        print(f"Failed to send message. Status code: {response.status_code}")

# Function to retrieve Solana token balance
def get_solana_balance(wallet_address):
    try:
        print(f"Retrieving Solana balance for wallet: {wallet_address}")
        url = "https://api.mainnet-beta.solana.com"
        payload = {
            "jsonrpc": "2.0",
            "id": 1,
            "method": "getBalance",
            "params": [
                wallet_address
            ]
        }
        headers = {
            "Content-Type": "application/json"
        }
        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            balance = response.json().get('result', {}).get('value')
            if balance is not None:
                return balance/10**9
            else:
                return "Failed to retrieve balance. Please try again later."
        else:
            return f"Failed to retrieve balance. Status code: {response.status_code}"
    except Exception as e:
        return str(e)


def get_token_list(wallet, chat_id):
    url = f"https://api.solana.fm/v1/addresses/{wallet}/tokens"
    headers = {"accept": "application/json"}
    print(f"Fetching token lists for wallet: {wallet}")
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        data = response.json()
        # data_list = json.loads(data)
        token_info = ""  # Initialize token_info as an empty string
        i =0;
        tokens = data['tokens']
        for token_data in tokens:
            element = tokens[token_data]
            if element['balance'] == 0 or element['balance'] == None:
                continue
            token_balance = element['balance']
            tkn_data = element['tokenData']
            if tkn_data == None or tkn_data == {}:
                continue
            tkn_list = tkn_data['tokenList']
            token_name = tkn_list["name"]
            token_symbol = tkn_list["symbol"]

            token_details = (
                f"Token Name: {token_name}\n"
                f"Token Symbol: {token_symbol}\n"
                f"Balance: {token_balance}\n\n"
            )
            token_info += token_details
        
        send_message(chat_id, f"Your Solana balance is:\n {token_info}")
    else:
        send_message(chat_id, f"Failed to retrieve tokens. Status code: {response.status_code}")


def get_recent_transactions(wallet_address):
    try:
        print(f"Fetching recent transactions for wallet: {wallet_address}")
        url = f"https://api.solana.fm/v0/accounts/{wallet_address}/transfers?page=1"
        response = requests.get(url)
        if response.status_code == 200:
            data = response.json()
            results = data.get('results', [])
            transactions_info = "------Your most recent transactions------\n\n"
            i=0;
            for result in results:
                transaction_hash = result.get('transactionHash')
                data = result.get('data', [])
                for item in data:
                    if item.get('action') == 'transfer':
                        sender = item.get('source')
                        receiver = item.get('destination')
                        amount = item.get('amount')
                        amount_sol = "more"
                        transaction_link = f"[{transaction_hash}]"
                        transactions = (
                            f"Transaction Hash: {transaction_link}\n"
                            f" \n"
                            f"Sender: {sender}\n"
                            f" \n"
                            f"Receiver: {receiver}\n"
                            f" \n"
                            f"Read more: (https://solscan.io/tx/{transaction_hash})\n\n"
                        )
                        transactions_info += transactions
                i+=1
                if i == 2:
                    break
            return transactions_info  # Return only the 10 most recent transactions
        else:
            print(f"Failed to fetch recent transactions. Status code: {response.status_code}")
            return []
    except Exception as e:
        print(f"An error occurred while fetching recent transactions: {str(e)}")
        return []



# Process updates
def process_updates(updates):
    for update in updates:
        # Extract chat_id and text from the update
        chat_id = update.get('message', {}).get('chat', {}).get('id')
        text = update.get('message', {}).get('text')

        # Handle each update based on your bot's logic
        if text == '/start':
            send_message(chat_id, "Welcome to the Solana Explorer. Type /help see the available commands.")
        
        if text.startswith('/balance'):
            try:
                # Extract the wallet address from the command
                wallet_address = text.split('/balance ')[1].strip()
                # Retrieve the Solana token balance
                balance = get_solana_balance(wallet_address)
                # Send the balance as a reply
                send_message(chat_id, f"Your Solana balance is: {str(balance)}sol")
            except IndexError:
                send_message(chat_id, "Please provide a valid Solana wallet address.")
            except Exception as e:
                send_message(chat_id, f"An error occurred: {str(e)}")

        if text.startswith('/tokens'):
            try:
                # Extract the wallet address from the command
                wallet_address = text.split('/tokens ')[1].strip()
                # Retrieve the Solana token balance
                get_token_list(wallet_address, chat_id)
                # Send the balance as a reply
            except IndexError:
                send_message(chat_id, "Please provide a valid Solana wallet address.")
            except Exception as e:
                send_message(chat_id, f"An error occurred: {str(e)}")

        if text == '/help':
            help_text = (
                "Welcome to the Solana Explorer!\n"
                "Here are the available commands:\n"
                "/balance <wallet_address> - Get the Solana balance of a wallet\n"
                "/tokens <wallet_address> - Get the Solana tokens of a wallet\n"
                "/transactions <wallet_address> - Get the recent transactions of a wallet\n"

            )
            send_message(chat_id, help_text)
        
        if text.startswith('/transactions'):
            try:
                # Extract the wallet address from the command
                wallet_address = text.split('/transactions ')[1].strip()
                # Retrieve the recent transactions
                transactions = get_recent_transactions(wallet_address)
                # Send the transactions as a reply
                if transactions !=" ":   
                    send_message(chat_id, transactions)
                else:
                    send_message(chat_id, "No recent transactions found.")
            except IndexError:
                send_message(chat_id, "Please provide a valid Solana wallet address.")
            except Exception as e:
                send_message(chat_id, f"An error occurred: {str(e)}")

# Main function to continuously fetch and process updates
def main():
    offset = None
    while True:
        updates = get_updates(offset)
        if updates:
            process_updates(updates)
            # Set the new offset to fetch only new updates next time
            offset = updates[-1]['update_id'] + 1

if __name__ == '__main__':
    main()
