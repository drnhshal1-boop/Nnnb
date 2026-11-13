# Meaningful Host V8 — خطوات النشر

هذه نسخة تطويرية، وليست منصة استضافة جاهزة للإنتاج. تتطلب حساب Cloudflare وإعداد Workers وD1 وصلاحيات نشر Workers.

1. ثبّت Node.js 22+ ثم نفّذ `npm install`.
2. سجّل الدخول إلى Cloudflare باستخدام `npx wrangler login`.
3. أنشئ قاعدة بيانات: `npx wrangler d1 create meaningful-host-v8`.
4. انسخ `database_id` إلى `wrangler.jsonc`، واضبط `OWNER_ID` و`CF_ACCOUNT_ID` و`CF_WORKERS_SUBDOMAIN`.
5. نفّذ `npm run db:init`.
6. أضف الأسرار بالأوامر التالية (لا تضع القيم في المستودع):
   - `npx wrangler secret put BOT_TOKEN`
   - `npx wrangler secret put WEBHOOK_SECRET`
   - `npx wrangler secret put CF_API_TOKEN`
   - `npx wrangler secret put TOKEN_ENCRYPTION_KEY` (قيمة base64 لـ 32 بايت عشوائية)
7. نفّذ `npm run check` ثم `npm run deploy`.
8. اربط Webhook الرئيسي من جهازك عبر Telegram `setWebhook` إلى `https://<worker-url>/telegram/webhook` مع `secret_token` المطابق.
9. افتح محادثة خاصة مع البوت وأرسل `/start`.

## حدود مهمة

- تدعم هذه النسخة ملفات JS/TS مفردة أو ZIP يحوي ملف JS/TS واحدًا فقط دون imports خارجية.
- يجب أن يصدّر ملف المستخدم `export default { async fetch(request, env, ctx) { ... } }` ويعالج تحديثات Telegram بنفسه.
- يحتاج بوت المستخدم إلى توكن منفصل عبر `/token PROJECT_ID TOKEN` في محادثة خاصة، ثم الضغط على نشر.
- النشر التلقائي لا يضمن تفعيل نطاق workers.dev أو سلامة كود المستخدم أو عمل Webhook في المشروع المرفوع.
- نشر كود مستخدمين غير موثوقين داخل حساب Cloudflare واحد خطر أمني؛ لا تفتح الخدمة للعامة قبل تطبيق قيود نشر وفحص وعزل مناسب.
- لم تُختبر النسخة على حساب Cloudflare فعلي، ولا تتضمن كل خصائص V7.
