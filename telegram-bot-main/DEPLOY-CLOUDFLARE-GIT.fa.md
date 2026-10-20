# استقرار ezpacket با GitHub و Cloudflare Workers

این پروژه یک **Cloudflare Worker** است؛ آن را به‌عنوان Pages Static Site نسازید.

## ۱. پیش از اتصال GitHub

1. پروژه را در یک مخزن GitHub بارگذاری کنید؛ فایل‌های `package.json`، `package-lock.json` و `wrangler.toml` باید در ریشه مخزن باشند.
2. در Cloudflare به **Workers & Pages → Create → Connect to Git** بروید و همان مخزن را انتخاب کنید.
3. در تنظیمات Build این مقادیر را تنظیم کنید:
   - **Build command:** `npm run build`
   - **Deploy command:** `npx wrangler deploy`
   - **Root directory:** خالی یا `/`، اگر فایل `package.json` در ریشه مخزن است.
   - **Node.js version:** `22.16.0` یا جدیدترِ سازگار با `package.json`.
4. مطمئن شوید پروژه به‌عنوان Worker مستقر می‌شود، نه Cloudflare Pages.

## ۲. ساخت KV مخصوص حساب خودتان

پروژه از binding به نام `BOT_KV` استفاده می‌کند. شناسهٔ KV موجود در نسخهٔ اصلی ممکن است متعلق به حساب دیگری باشد و نباید به آن تکیه کرد.

1. در Cloudflare داشبورد، **Storage & databases → KV** را باز کنید و یک namespace بسازید؛ مثلاً `ezpacket-bot-kv`.
2. شناسهٔ Namespace را کپی کنید.
3. در `wrangler.toml` مقدار `REPLACE_WITH_YOUR_CLOUDFLARE_KV_NAMESPACE_ID` را با شناسهٔ واقعی خود جایگزین کنید.
4. این تغییر را به GitHub commit و push کنید تا Build خودکار دوباره اجرا شود.

**تا زمانی که شناسهٔ واقعی KV جایگزین نشده، Deploy را اجرا نکنید.** این پروژه داده‌های کاربران و تنظیمات را در KV ذخیره می‌کند.

## ۳. متغیرها و اسرار

توکن ربات تلگرام و هر کلید محرمانه را در فایل‌های Git قرار ندهید. اسرار موردنیاز Worker را از بخش **Settings → Variables and Secrets** تنظیم کنید. پس از استقرار، لاگ‌های Build و Worker را بررسی کنید.

## ۴. بررسی خطا

اگر Build یا Deploy شکست خورد، از بخش Build logs متن کامل خطا را بردارید. خطاهای رایج:

- خطای KV namespace: شناسهٔ KV جایگزین نشده یا اشتباه است.
- خطای نسخه Node: نسخهٔ Node باید با مقدار `engines` در `package.json` سازگار باشد.
- خطای وابستگی یا build: `npm ci` یا `npm run build` شکست خورده؛ متن کامل لاگ مشخص می‌کند کدام بسته یا فایل مشکل دارد.
- خطای Durable Objects: binding و migrationهای تعریف‌شده در `wrangler.toml` را حذف یا تغییر ندهید مگر با migration برنامه‌ریزی‌شده.

## ۵. نکته دربارهٔ اولین استقرار

تنظیمات Git اتصال مخزن را انجام می‌دهد، اما به‌تنهایی منابع حساب Cloudflare را ایجاد یا اسرار ربات را وارد نمی‌کند. KV و متغیرهای محرمانه را پیش از استفاده واقعی تنظیم کنید.
