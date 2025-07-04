from flask import Flask, request
import requests

app = Flask(__name__)

# === Thông số cấu hình ===
TELEGRAM_TOKEN = "7412201763:AAF7cHSUPb_N043LqhJfi2wmeY20h4jib8M"
TELEGRAM_CHAT_ID = "6522628107"
MIN_SOL = 150
MAX_SOL = 170

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    payload = {
        "chat_id": TELEGRAM_CHAT_ID,
        "text": message,
        "parse_mode": "Markdown"
    }
    requests.post(url, json=payload)

@app.route("/webhook", methods=["POST"])
def webhook():
    data = request.get_json()
    txs = data.get("transactions", [])

    for tx in txs:
        signature = tx.get("signature", "")
        transfers = tx.get("nativeTransfers", [])
        for transfer in transfers:
            amount = int(transfer.get("amount", 0)) / 1e9
            if MIN_SOL <= amount <= MAX_SOL:
                sender = transfer.get("fromUserAccount", "unknown")
                receiver = transfer.get("toUserAccount", "unknown")
                link = f"https://solscan.io/tx/{signature}"
                msg = f"🚨 *Giao dịch từ {MIN_SOL} đến {MAX_SOL} SOL phát hiện!*\n" \
                      f"💸 {amount:.3f} SOL\n👤 Từ: `{sender}`\n👤 Đến: `{receiver}`\n🔗 [Xem trên Solscan]({link})"
                send_telegram_message(msg)
    return "ok", 200

@app.route("/", methods=["GET"])
def index():
    return "Solana Bot đang chạy."

if __name__ == "__main__":
    app.run()
