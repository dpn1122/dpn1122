from flask import Flask, request
import requests, hmac, hashlib, time, os

app = Flask(__name__)

API_KEY = os.getenv("API_KEY")
API_SECRET = os.getenv("API_SECRET")

def sign_request(params, secret):
    query = '&'.join([f"{k}={v}" for k, v in sorted(params.items())])
    return hmac.new(secret.encode(), query.encode(), hashlib.sha256).hexdigest()

def send_market_order(side):
    url = "https://api.lbkex.com/v2/supplement/placeOrder"
    symbol = "btc_usdt"
    params = {
        "apiKey": API_KEY,
        "symbol": symbol,
        "type": "buy_market" if side == "buy" else "sell_market",
        "amount": "0.001",  # مقدار سفارش، قابل تنظیم
        "timestamp": int(time.time() * 1000)
    }
    params["sign"] = sign_request(params, API_SECRET)
    res = requests.post(url, data=params)
    print("LBank response:", res.json())

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.get_json()
    action = data.get("action")
    if action in ["buy", "sell"]:
        send_market_order(action)
    return {"status": "received"}

if __name__ == '__main__':
    app.run(debug=True)

