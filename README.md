# Aizazotpbot1
This bot is designed for otp
import requests
import time
import re
import json
from datetime import datetime

# === CONFIGURATION ===
BASE_URL = 'http://94.23.120.156'
LOGIN_URL = f'{BASE_URL}/ints/login'
SIGNIN_URL = f'{BASE_URL}/ints/signin'
DATA_URL = f'{BASE_URL}/ints/agent/res/data_smscdr.php'
USERNAME = 'aizazalishah1'
PASSWORD = '@aizaz123'

# === Telegram Bot Info ===
BOT_TOKEN = '8739617420:AAFyIWk-5B7Y2utKOqzapaAbl5CYfqbOGKE'
CHAT_ID = '-1003946331034'
SENT_FILE = 'already_sent.json'

# === CAPTCHA SOLVER ===
def solve_captcha(html):
    match = re.search(r'(\d+)\s*\+\s*(\d+)', html)
    if match:
        return int(match.group(1)) + int(match.group(2))
    return None

# === SEND TO TELEGRAM ===
def send_to_telegram(message):
    url = f'https://api.telegram.org/bot{BOT_TOKEN}/sendMessage'
    keyboard = {
        "inline_keyboard": [
            [{"text": "All Numbers File", "url": "https://t.me/+UX3qX1V1JoxkZWQ1"}]
        ]
    }
    data = {
        'chat_id': CHAT_ID,
        'text': message,
        'parse_mode': 'HTML',
        'reply_markup': json.dumps(keyboard)
    }
    try:
        response = requests.post(url, data=data)
        if response.status_code != 200:
            print(f'[!] Telegram API Error: {response.text}')
    except Exception as e:
        print(f'❌ Telegram Error: {e}')

# === LOGIN FUNCTION ===
def login(session):
    print("⚠️ Logging in...")
    try:
        resp = session.get(LOGIN_URL)
        captcha = solve_captcha(resp.text)
        if captcha is None:
            print("Captcha not found")
            return False

        print(f"Solved captcha: {captcha}")
        
        payload = {
            'username': USERNAME,
            'password': PASSWORD,
            'capt': captcha
        }

        res = session.post(SIGNIN_URL, data=payload)
        if 'dashboard' in res.text.lower() or 'logout' in res.text.lower():
            print("✅ Login successful.")
            return True
        else:
            print("[!] Login failed: dashboard/logout not found in response.")
            print(f"🔓 Login response snippet: {res.text[:300]}")
            return False
    except Exception as e:
        print(f"🔑 Login Exception: {e}")
        return False

# === FETCH DATA ===
def fetch_data(session):
    today = datetime.now().strftime('%Y-%m-%d')
    from_time = f'{today} 00:00:00'
    to_time = f'{today} 23:59:59'
    url = f"{DATA_URL}?fdate1={from_time}&fdate2={to_time}&iDisplayLength=-1"
    try:
        res = session.get(url, headers={
            "X-Requested-With": "XMLHttpRequest",
            "Referer": f"{BASE_URL}/ints/agent/SMSCDRStats"
        }, timeout=10)
        data = res.json()
        return data.get("aaData", [])
    except Exception as e:
        print(f"[!] Fetch data error: {e}")
        return []

# === NUMBER MASKING ===
def mask_number(number: str) -> str:
    if len(number) >= 11:
        return number[:6] + "***" + number[-4:]
    return number

# === COUNTRY DETECTION ===
def detect_country(number):
    number = number.replace(' ', '').replace('-', '')
    countries = {
        '7': 'Russia',
        '20': 'Egypt',
        '27': 'South Africa',
        '34': 'Spain',
        '39': 'Italy',
        '44': 'United Kingdom',
        '46': 'Sweden',
        '48': 'Poland',
        '49': 'Germany',
        '55': 'Brazil',
        '58': 'Venezuela',
        '60': 'Malaysia',
        '62': 'Indonesia',
        '63': 'Philippines',
        '66': 'Thailand',
        '81': 'Japan',
        '84': 'Vietnam',
        '86': 'China',
        '255': 'Tanzania',
        '92': 'Pakistan',
        '93': 'Afghanistan',
        '94': 'Sri Lanka',
        '212': 'Morocco',
        '213': 'Algeria',
        '218': 'Libya',
        '221': 'Senegal',
        '223': 'Mali',
        '225': 'Ivory Coast',
        '228': 'Togo',
        '234': 'Nigeria',
        '235': 'Chad',
        '249': 'Sudan',
        '251': 'Ethiopia',
        '254': 'Kenya',
        '256': 'Uganda',
        '263': 'Zimbabwe',
        '359': 'Bulgaria',
        '383': 'Kosovo',
        '504': 'Honduras',
        '880': 'Bangladesh',
        '961': 'Lebanon',
        '964': 'Iraq',
        '98': 'Iran',
        '257': 'Burundi',
        '966': 'Saudi Arabia',
        '967': 'Yemen',
        '968': 'Oman',
        '970': 'Palestine',
        '386': 'Slovenia',
        '972': 'Israel',
        '976': 'Mongolia',
        '992': 'Tajikistan',
        '998': 'Uzbekistan',
    }
    for code in sorted(countries.keys(), key=lambda x: -len(x)):
        if number.startswith('+' + code) or number.startswith(code):
            return countries[code]
    return "Unknown"

# === COUNTRY FLAG ===
def get_flag(country: str) -> str:
    flags = {
       'Iraq': '🇮🇶',
       'Senegal': '🇸🇳',
       'United Kingdom': '🇬🇧',
       'Tanzania': '🇹🇿',
       'Bangladesh': '🇧🇩',
       'Pakistan': '🇵🇰',
       'Russia': '🇷🇺',
       'Oman': '🇴🇲',
       'Germany': '🇩🇪',
       'Spain': '🇪🇸',
       'Italy': '🇮🇹',
       'Japan': '🇯🇵',
       'China': '🇨🇳',
       'Indonesia': '🇮🇩',
       'Philippines': '🇵🇭',
       'Brazil': '🇧🇷',
       'Vietnam': '🇻🇳',
       'Thailand': '🇹🇭',
       'Egypt': '🇪🇬',
       'Uzbekistan': '🇺🇿',
       'Mongolia': '🇲🇳',
       'Malaysia': '🇲🇾',
       'Palestine': '🇵🇸',
       'Israel': '🇮🇱',
       'Lebanon': '🇱🇧',
       'Burundi': '🇧🇮',
       'Sri Lanka': '🇱🇰',
       'Poland': '🇵🇱',
       'Sweden': '🇸🇪',
       'Nigeria': '🇳🇬',
       'Uganda': '🇺🇬',
       'Iran': '🇮🇷',
       'Morocco': '🇲🇦',
       'Sudan': '🇸🇩',
       'Togo': '🇹🇬',
       'Kenya': '🇰🇪',
       'Ethiopia': '🇪🇹',
       'Chad': '🇹🇩',
       'Algeria': '🇩🇿',
       'Zimbabwe': '🇿🇼',
       'Libya': '🇱🇾',
       'Afghanistan': '🇦🇫',
       'Venezuela': '🇻🇪',
       'South Africa': '🇿🇦',
       'Tajikistan': '🇹🇯',
       'Saudi Arabia': '🇸🇦',
       'Kosovo': '🇽🇰',
       'Mali': '🇲🇱',
       'Honduras': '🇭🇳',
       'Ivory Coast': '🇨🇮',
       'Bulgaria': '🇧🇬',
       'Slovenia': '🇸🇮',
       'Yemen': '🇾🇪',
    }
    return flags.get(country, '🏳️')

# === SENT FILE LOAD/SAVE ===
def load_sent():
    try:
        with open(SENT_FILE, 'r', encoding='utf-8') as f:
            return set(json.load(f))
    except Exception:
        return set()

def save_sent(sent_set):
    with open(SENT_FILE, 'w', encoding='utf-8') as f:
        json.dump(list(sent_set), f, ensure_ascii=False, indent=2)

# === MAIN ===
def main():
    session = requests.Session()
    session.headers.update({"User-Agent": "Mozilla/5.0"})
    if not login(session):
        print("❌ Login failed. Exiting.")
        return

    sent_set = load_sent()
    print("✅ Monitoring started...")

    while True:
        print("🔍 Checking for messages...")
        rows = fetch_data(session)
        print(f"📥 Today received: {len(rows)}")

        if not rows:
            print("🔕 No new data found. Sleeping for 5 seconds...")
            time.sleep(5)
            continue

        rows.sort(key=lambda x: x[0], reverse=True)

        new_sent = 0
        for row in rows:
            Now = datetime.now().strftime('%Y-%m-%d %I:%M:%S %p')
            number = str(row[2]).strip()
            service = str(row[3]).strip()
            message = str(row[5]).strip()

            unique_id = f"{number}_{message}"
            if unique_id in sent_set:
                continue

            match = re.search(r'\b(\d{4,8}|\d{3}-\d{3}|\d{2,4}-\d{2,4})\b', message)
            otp = match.group() if match else "No OTP found"

            country_name = detect_country(number)
            flag = get_flag(country_name)

            telegram_msg = (
                f"<b>🔔 {flag} {country_name} {service} Otp Code Received Successfully...</b>\n\n"
                f"⏰<b>Time: </b>{Now}\n"
                f"📲<b>Number:</b> <code>{mask_number(number)}</code>\n"
                f"🌐<b>Country:</b> {country_name} {flag}\n"
                f"💬<b>Service:</b> {service}\n"
                f"🔐<b>Otp Code:</b> <code>{otp}</code>\n"
                f"📩<b>Message:</b>\n<pre>{message}</pre>\n"
            )

            send_to_telegram(telegram_msg)
            print(f"🔔 Sent OTP from {service}: {otp}")
            sent_set.add(unique_id)
            new_sent += 1
            save_sent(sent_set)
            time.sleep(1)

        if new_sent == 0:
            print("[*] No new messages found. Sleeping 3 seconds...")
            time.sleep(3)

if __name__ == "__main__":
    main()
