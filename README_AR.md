# Meaningful Host V8 — نسخة أولية قابلة للتطوير

**الحالة:** مشروع أولي (MVP) وليس بديلًا كاملًا ومختبرًا لـ V7. يدعم رفع JS/TS المفردة، تخزين المشروع في D1، طلب الموافقة، ونشر Worker من خلال Cloudflare API. **لا يدعم ZIP أو إدارة Webhook لبوت المستخدم أو تشغيل/إيقاف البوتات فعليًا بعد**. نشر Worker لا يعني أن البوت المرفوع سيعمل تلقائيًا. لا تستخدمه لاستضافة ملفات غير موثوقة: مفتاح النشر يمتلك صلاحيات على حساب Cloudflare.

## خطوات الإعداد
1. أنشئ حساب Cloudflare، ثم Workers & Pages، وD1 باسم `meaningful-host-v8`.
2. ثبّت Node.js 20+ محليًا ونفذ `npm install` ثم `npx wrangler login`.
3. احصل على معرف D1 عبر `npx wrangler d1 list`، وضعه في `wrangler.jsonc` بدل `REPLACE_WITH_D1_ID`.
4. ضع معرف حساب Cloudflare في `CF_ACCOUNT_ID` ورقم حساب تيليجرام في `OWNER_ID` داخل `wrangler.jsonc`.
5. نفذ `npm run db:init` لإنشاء الجداول.
6. أنشئ Cloudflare API Token محدود الصلاحيات المطلوبة لنشر Workers في حسابك. نفذ `npx wrangler secret put CF_API_TOKEN`.
7. نفذ `npx wrangler secret put BOT_TOKEN` ثم `npx wrangler secret put WEBHOOK_SECRET`. اختر سر Webhook عشوائيًا من أحرف وأرقام و `_` و `-` بطول 1–256 حرفًا.
8. نفذ `npm run deploy`، وانسخ عنوان Worker.
9. سجّل Webhook لدى Telegram باستخدام طلب POST وليس بوضع التوكن في روابط تُشارك مع الآخرين:

```bash
curl -sS -X POST "https://api.telegram.org/bot${BOT_TOKEN}/setWebhook" \
  -d "url=https://YOUR-WORKER.workers.dev/telegram/webhook" \
  -d "secret_token=${WEBHOOK_SECRET}" \
  -d 'allowed_updates=["message","callback_query"]'
```

10. افتح البوت وأرسل `/start`، وللمالك `/admin`.

## قيود معروفة
- ملفات `.zip` و`.py` غير مدعومة في هذه النسخة؛ TypeScript المفرد يُحوّل باستخدام TypeScript transpileModule، دون حل imports أو تجميع dependencies.
- ملفات Worker المرفوعة يجب أن تصدّر fetch handler، وليست سكربتات polling أو برامج Node.js مستمرة.
- يتم نشر كود المستخدم باستخدام صلاحيات حساب Cloudflare الخاص بك؛ لا تفتح النشر للعامة قبل إضافة تدقيق أمني وعزل وصلاحيات أدق.
- لا يتم إنشاء أسرار بوتات المستخدمين ولا ربط Webhook لهم تلقائيًا. يحتاج ذلك تصميم إعداد آمن لتوكن كل بوت وWorker subdomain، وموافقة صاحب البوت.
- تخزين الشيفرة في D1 غير مناسب للمشاريع الكبيرة؛ يُفضّل R2 في إصدار لاحق.
- واجهة V7 مطابقة جزئيًا فقط، وليست جميع الوظائف القديمة منقولة.
- حدود Cloudflare المجانية تنطبق على كل حساب، وقد تتغير.
