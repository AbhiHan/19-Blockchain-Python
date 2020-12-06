What does a wallet do? 
Wallet is mean to store keys.


Code: ** Could not get to execute


from constants import *
from mn import *
import subprocess
import json
import os
from dotenv import load_dotenv
load_dotenv()
from web3 import Web3
from eth_account import Account
import bit


# ************* Define Derive Wallet Function ************

def derive_wallets(coin,numderive,mnemonic):
    command =f'php hd-wallet-derive/hd-wallet-derive.php -g --mnemonic="{mnemonic}" --cols=path,address,privkey,pubkey --format=json --coin={coin} --numderive={numderive}'
    #print(command)
    p = subprocess.Popen(command, stdout=subprocess.PIPE, shell=True)
    
    output, err = p.communicate()
    p_status = p.wait()
    keys = json.loads(output)
    print(keys)
    print('')
    print('')
    print('')

    
    return keys

# Creatn a coin object 
coins = {BTCTEST: derive_wallets(BTCTEST,3,mnemonic), ETH:derive_wallets(ETH,3,mnemonic)}


print(' ********** Final Output - FIrst BTC-TEST COIN Priv Key ********** ')
print(coins[BTCTEST][0]['privkey'])
print('')
print('')
print('')
 
# ********************************************************
def priv_key_to_account():
    btc_acct = bit.PrivateKeyTestnet(coins[BTCTEST][0]['privkey'])
    eth_acct = Account.privateKeyToAccount(coins[ETH][0]['privkey'])
    
    return btc_acct, eth_acct

# get accounts from keys
btc_acct, eth_acct = priv_key_to_account()

# Create Raw Transaction
def create_raw_tx(account, recipient, amount):
    gasEstimate = w3.eth.estimateGas({
        "from":  account.address, 
        "to":    recipient, 
        "value": amount
    })
    return {
        "from":     account.address, 
        "to"  :     recipient,
        "value":    amount, 
        "gasPrice": w3.eth.gasPrice,
        "gas":      gasEstimate,
        "nonce":    w3.eth.getTransactionCount(account.address)       
    }

def send_tx(account, recipient, amount):
    tx = create_raw_tx(account, recipient, amount)
    signed_tx = account.sign_transaction(tx)
    result = w3.eth.sendRawTransaction(signed_tx.rawTransaction)
    print(result.hex())
    return result.hex()


send_tx(act, "0x85434Ac781bA4fC75292A6533bA9F0676d0f88BA", 1)
