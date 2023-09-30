import random
import string 
import requests
import os
import time
import json
import datetime
from colorama import Fore, init
from configparser import ConfigParser
from flask import Flask, request

app = Flask(__name__)
init(autoreset=True)

__version__ = "Author:Olmaz#0"
__github__ = "https://github.com/GokuReal"
dir_path = os.path.dirname(os.path.realpath(__file__))
configur = ConfigParser()
configur.read(os.path.join(dir_path, "config.ini"))
tokens_list = os.path.join(dir_path, "tokens.txt")
integ_0 = 0
sys_url = "https://discord.com/api/v9/users/@me"
URL = "https://discord.com/api/v9/users/@me/pomelo-attempt"

def s_sys_h():
    if configur.getboolean("sys", "MULTI_TOKEN"):
        return {
            "Content-Type": "Application/json",
            "Origin": "https://discord.com/",
            "Authorization": avail_tokens(tokens_list)[integ_0]
        }
    else:
        return {
            "Content-Type": "Application/json",
            "Origin": "https://discord.com/",
            "Authorization": configur.get("sys", "TOKEN")
        }

def avail_tokens(path):
    with open(path, 'r') as at:
        tokens = at.read().splitlines()
    return tokens

def rate_limit_handler(response):
    if response.status_code == 429:
        retry_after = response.json().get("retry_after", 1)
        print(f"{Fore.LIGHTBLACK_EX}[!]{Fore.RED} Rate limit yedik. {retry_after}s bekleniyor...")
        time.sleep(retry_after)
        return True
    return False

@app.route("/", methods=["GET"])
def home():
    return "GokuReal's Discord Nick Sniper is running."

def validate_username(username):
    body = {
        "username": username
    }
    response = requests.post(URL, headers=s_sys_h(), json=body)
    json_response = response.json()

    if response.status_code == 429:
        retry_after = json_response["retry_after"] / 1000
        time.sleep(retry_after)
        return validate_username(username)  

    if json_response.get("taken") is not None:
        return not json_response["taken"]
    else:
        return False

@app.route('/check', methods=['POST'])
def check_username():
    data = request.get_json()
    if "username" in data:
        username = data["username"]
        is_available = validate_username(username)
        return jsonify({"available": is_available})
    else:
        return jsonify({"error": "Invalid request data"})



    
    response = requests.post(URL, headers=s_sys_h(), json={"username": username})
    
    if rate_limit_handler(response):
        return check_username()
    
    if response.status_code == 200:
        if response.json().get("taken") is False:
            return f"'{username}' alınabilir!"
        else:
            return f"'{username}' zaten alınmış."
    else:
        return f"Sunucudan dönen hata: {response.status_code}", 500

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
    
