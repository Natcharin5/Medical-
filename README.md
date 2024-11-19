# Medical-
medication reminder system
from fastapi import FastAPI, Request
from linebot import LineBotApi
from linebot.models import TextSendMessage
import schedule
import time
from datetime import datetime

app = FastAPI()
line_bot_api = LineBotApi('YOUR_CHANNEL_ACCESS_TOKEN')
user_id = 'USER_LINE_ID'

medication_schedule = [
    {"name": "Aspirin", "time": "08:00 AM"},
    {"name": "Vitamin C", "time": "01:00 PM"},
    {"name": "Antibiotic", "time": "07:00 PM"}
]

def send_medication_reminder():
    current_time = datetime.now().strftime("%I:%M %p")
    for med in medication_schedule:
        if med["time"] == current_time:
            message = f"Reminder: It's time to take your {med['name']}!"
            line_bot_api.push_message(user_id, TextSendMessage(text=message))

def job():
    schedule.every().minute.at(":00").do(send_medication_reminder)

@app.post("/callback")
async def callback(request: Request):
    body = await request.body()
    signature = request.headers['X-Line-Signature']
    line_bot_api.handle(body.decode(), signature)
    return 'OK'

if __name__ == "__main__":
    job()
    while True:
        schedule.run_pending()
        time.sleep(60)
