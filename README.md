# TeleAuthX — اتصال رسمی و امن به تلگرام  
![TeleAuthX Banner](https://www.digikala.com/mag/wp-content/uploads/2024/10/11-1-13.jpg)  
**اتصال امن، سریع و رسمی کاربران تلگرام به پروژه‌های شما**  

---  

## معرفی پروژه  
**TeleAuthX** یک کتابخانه و راهکار جامع برای پیاده‌سازی **Telegram Login Authorization** (ویجت ورود رسمی تلگرام) است. این پروژه با هدف حذف کامل مشکلات روش‌های سنتی اتصال به تلگرام — مانند مدیریت فایل‌های سشن، ریسک بلاک شدن اکانت و امنیت پایین — توسعه یافته است.  

با استفاده از **Telegram Login Widget**، کاربران بدون نیاز به وارد کردن شماره تلفن، کد تأیید یا ذخیره سشن، به صورت **رسمی، امن و فوری** به پروژه‌های شما متصل می‌شوند.  

این سیستم کاملاً بر پایه استانداردهای رسمی تلگرام ساخته شده و برای انواع پروژه‌ها — از ربات‌های ساده تا پنل‌های پیچیده مدیریتی — قابل استفاده است.  

---  

## چرا TeleAuthX؟  
روش‌های قدیمی اتصال به تلگرام (مانند استفاده از `API ID/Hash` یا فایل‌های سشن) همواره با چالش‌هایی همراه بوده‌اند:  

| مشکل | توضیح | راه‌حل در TeleAuthX |
|------|-------|---------------------|
| **امنیت پایین** | ذخیره سشن یا اطلاعات حساس در سرور | استفاده از توکن‌های یک‌بارمصرف و تأیید رسمی تلگرام |
| **بلاک شدن اکانت** | لاگین‌های مکرر و مشکوک | ورود از طریق ویجت رسمی — بدون ریسک تشخیص اسپم |
| **مدیریت پیچیده** | فایل‌های سشن جداگانه برای هر کاربر | سیستم توکن‌بیسد و بدون نیاز به ذخیره‌سازی دائمی |
| **عدم انعطاف‌پذیری** | محدود به یک فریم‌ورک یا پلتفرم | پشتیبانی از Flask, Next.js, Express, Django, Node.js و ... |

**TeleAuthX** این مشکلات را با بسته‌بندی هوشمند ویجت ورود تلگرام و افزودن لایه‌های امنیتی و توسعه‌پذیری حل کرده است.  

---  

## مزایای استفاده از Telegram Login Widget  

| ویژگی | توضیح رسمی | مزیت عملی |
|-------|-------------|-----------|
| **امنیت بالا** | تمام ارتباطات از طریق HTTPS و تأیید امضای HMAC-SHA256 | هیچ داده حساسی (شماره، سشن) در سرور ذخیره نمی‌شود |
| **ورود بدون سشن** | استفاده از توکن‌های موقت و یک‌بارمصرف | حذف کامل فایل‌های `.session` |
| **پشتیبانی چندپلتفرمی** | وب، موبایل، PWA، اپ دسکتاپ | ویجت responsive و موبایل‌فرندلی |
| **ادغام آسان** | فقط چند خط کد HTML + یک endpoint | راه‌اندازی در کمتر از ۵ دقیقه |
| **بدون ریسک بلاک** | استفاده از کانال رسمی تلگرام | لاگین‌های معتبر و غیرمشکوک |
| **زبان‌های متعدد** | پشتیبانی از `fa`, `en`, `ru` و ... | دکمه ورود به زبان پارسی |
| **توسعه‌پذیر** | قابلیت اتصال به JWT, OAuth, دیتابیس | ساخت سیستم‌های احراز هویت پیشرفته |

---  

## راه‌اندازی گام‌به‌گام  

### 1️⃣ ایجاد ربات در [@BotFather](https://t.me/BotFather)  
```text
/newbot → انتخاب نام → دریافت توکن
```
> توکن را در متغیر محیطی (`BOT_TOKEN`) ذخیره کنید.  

### 2️⃣ تنظیم دامنه مجاز (Domain Whitelisting)  
```text
/setdomain → انتخاب ربات → وارد کردن دامنه (مثلاً https://yourproject.com)
```
> **حتماً از HTTPS استفاده کنید.**  
> برای تست لوکال: از `ngrok` یا `localhost.run` استفاده کنید.  

### 3️⃣ افزودن ویجت ورود به صفحه  
```html
<script async src="https://telegram.org/js/telegram-widget.js?22"
        data-telegram-login="YourBotUsername_bot"
        data-size="large"
        data-radius="12"
        data-auth-url="/api/auth/telegram/callback"
        data-request-access="write"
        data-userpic="true"
        data-lang="fa"></script>
```
> - `YourBotUsername_bot`: نام کاربری ربات **بدون @**  
> - `data-auth-url`: آدرس endpoint سرور شما  
> - `?22`: آخرین نسخه امن اسکریپت تلگرام  

### 4️⃣ پیاده‌سازی endpoint تأیید (Flask Example)  
```python
from flask import Flask, request, jsonify
import hashlib
import hmac
import time
import os

app = Flask(__name__)
BOT_TOKEN = os.getenv("BOT_TOKEN")

def verify_telegram_auth(data: dict) -> bool:
    if 'hash' not in data:
        return False
    
    received_hash = data.pop('hash')
    data_check_string = '\n'.join(
        f"{k}={v}" for k, v in sorted(data.items()) if k != 'hash'
    )
    secret_key = hashlib.sha256(BOT_TOKEN.encode()).digest()
    calculated_hash = hmac.new(
        secret_key, data_check_string.encode(), hashlib.sha256
    ).hexdigest()

    # جلوگیری از Replay Attack
    auth_date = int(data.get('auth_date', 0))
    if abs(int(time.time()) - auth_date) > 86400:
        return False

    return hmac.compare_digest(calculated_hash, received_hash)

@app.route('/api/auth/telegram/callback', methods=['GET', 'POST'])
def telegram_callback():
    data = request.args.to_dict() if request.method == 'GET' else request.form.to_dict()
    
    if verify_telegram_auth(data.copy()):
        user = {
            "id": data['id'],
            "first_name": data.get('first_name'),
            "last_name": data.get('last_name'),
            "username": data.get('username'),
            "photo_url": data.get('photo_url')
        }
        # TODO: صدور JWT، ثبت در دیتابیس، شروع سشن
        return jsonify({"status": "success", "user": user})
    
    return jsonify({"status": "error", "message": "احراز هویت ناموفق"}), 401

if __name__ == '__main__':
    app.run(ssl_context='adhoc')  # برای تست لوکال با HTTPS
```

---  

## نمونه‌های کاربردی (پروژه‌های واقعی)  

| نوع پروژه | کاربرد | قابلیت‌های پیاده‌سازی شده |
|-----------|--------|-----------------------------|
| **سلف‌بات / تبچی** | تأیید مالکیت بدون سشن | اتصال بیش از ۵۰ اکانت بدون بلاک |
| **ربات ضد لینک** | پنل مدیریتی ادمین | ورود فقط با تأیید تلگرام |
| **داشبورد تحلیلی** | نمایش PV/IM | ادغام با Chart.js و WebSocket |
| **پنل خدمات** | مدیریت سفارش و پرداخت | ترکیب با درگاه Stripe/Zarinpal |
| **اتوماسیون وب** | کنترل ربات از داشبورد | آپدیت real-time با Webhook |
| **PWA / اپ موبایل** | لاگین در اپ | سازگار با React Native / Flutter |

---  

## نکات امنیتی حیاتی  

| نکته | توضیح |
|------|-------|
| **HTTPS الزامی** | ویجت بدون SSL کار نمی‌کند |
| **تأیید HMAC-SHA256** | همیشه hash را در سرور بررسی کنید |
| **عدم ذخیره API ID/Hash** | فقط `BOT_TOKEN` کافی است |
| **Rate Limiting** | از حملات Brute Force جلوگیری کنید |
| **حداقل داده** | فقط `id` و `username` ذخیره شود |
| **مهلت زمانی** | داده‌های قدیمی‌تر از ۲۴ ساعت رد شوند |
| **استفاده از `.env`** | توکن را در کد هاردکد نکنید |

---  

## پیاده‌سازی در زبان‌های دیگر  

### Node.js / Express  
```javascript
const crypto = require('crypto');

function verifyTelegramAuth(data, botToken) {
  const hash = data.hash;
  delete data.hash;
  
  const dataCheckString = Object.keys(data)
    .sort()
    .map(k => `${k}=${data[k]}`)
    .join('\n');
  
  const secretKey = crypto.createHash('sha256').update(botToken).digest();
  const calculatedHash = crypto.createHmac('sha256', secretKey)
    .update(dataCheckString).digest('hex');
  
  const authDate = parseInt(data.auth_date);
  if (Math.abs(Date.now() / 1000 - authDate) > 86400) return false;
  
  return crypto.timingSafeEqual(Buffer.from(calculatedHash), Buffer.from(hash));
}
```

### PHP  
```php
function verify_telegram_auth(array $data, string $botToken): bool {
    $hash = $data['hash'] ?? '';
    unset($data['hash']);
    
    uksort($data, 'strcmp');
    $dataCheckString = http_build_query($data, '', "\n");
    
    $secretKey = hash('sha256', $botToken, true);
    $calculatedHash = hash_hmac('sha256', $dataCheckString, $secretKey);
    
    return hash_equals($calculatedHash, $hash);
}
```

---  

## پیش‌نیازها  

| مورد | نسخه پیشنهادی |
|------|----------------|
| Python | 3.8 یا بالاتر |
| فریم‌ورک وب | Flask / FastAPI / Django / Express / Next.js |
| دامنه | با گواهی SSL معتبر |
| ربات تلگرام | از [@BotFather](https://t.me/BotFather) |

---  

## شروع سریع  
```bash
git clone https://github.com/theesmaeil1/TeleAuthX.git
cd TeleAuthX
pip install -r requirements.txt
export BOT_TOKEN="your_bot_token_here"
uvicorn app:app --reload --ssl-keyfile=localhost.key --ssl-certfile=localhost.crt
```

---  

## پیشنهادات توسعه  

- [ ] اتصال به دیتابیس (PostgreSQL, MongoDB, Redis)  
- [ ] صدور JWT با انقضا و Refresh Token  
- [ ] پنل مدیریت کاربران لاگین‌شده  
- [ ] پشتیبانی از چند ربات همزمان  
- [ ] داشبورد آمار ورودها (با Chart.js)  
- [ ] سیستم Logout از راه دور  
- [ ] ادغام با OAuth2 / OpenID  

---  

## مشارکت در پروژه  
ما از مشارکت شما استقبال می‌کنیم!  

- **باگ پیدا کردید؟** → [ایجاد Issue](https://github.com/theesmaeil1/TeleAuthX/issues)  
- **ایده دارید؟** → [ارسال Pull Request](https://github.com/theesmaeil1/TeleAuthX/pulls)  
- **استفاده کردید؟** → ⭐ ستاره فراموش نشود!  

---  

**ساخته شده با ❤️ توسط [theesmaeil1](https://github.com/theesmaeil1)**  
**لایسنس: MIT**  
**مستندات رسمی: [README.md](https://github.com/theesmaeil1/TeleAuthX/blob/main/README.md)**  

---  

> **TeleAuthX — آینده احراز هویت در اکوسیستم تلگرام**
