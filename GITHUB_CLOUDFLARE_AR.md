# نشر Meaningful Host V8 من GitHub إلى Cloudflare

## تنبيه قبل البدء
هذا إصدار تطويري يحتاج اختبارًا على حساب Cloudflare فعلي. البوت الرئيسي يعمل عبر Telegram Webhook، لكنه ليس استضافة عامة لأي كود Node.js. ملفات المستخدمين المقبولة هي Worker مستقل متوافق مع `export default { async fetch(request, env, ctx) { ... } }` دون imports خارجية. ملفات ZIP تقبل ملف JS/TS واحدًا فقط. لا يوجد تحويل تلقائي من polling إلى webhook في بوت المستخدم. لا ترفع توكنات إلى GitHub.

## 1. أنشئ حساب GitHub ومستودعًا خاصًا
1. افتح https://github.com/new وأنشئ مستودعًا Private باسم `meaningful-host-v8`.
2. فك ZIP وارفع **محتويات مجلد Meaningful_Host_V8** إلى جذر المستودع، بما فيها `.github/workflows/deploy.yml` و`src/` و`sql/` و`wrangler.jsonc`.
3. لا ترفع `.env` أو `.dev.vars` أو أي توكنات حقيقية.

## 2. أنشئ قاعدة D1
1. افتح https://dash.cloudflare.com/ ثم Workers & Pages > D1 (قد يظهر تحت Storage & databases).
2. أنشئ قاعدة باسم `meaningful-host-v8`.
3. انسخ Database ID إلى `database_id` في `wrangler.jsonc` وادفع التغيير إلى GitHub.
4. من SQL Console الصق محتويات `sql/schema.sql` ونفّذها **مرة واحدة**. أو نفّذ من جهازك `npx wrangler d1 execute meaningful-host-v8 --remote --file=./sql/schema.sql`.

## 3. إعداد متغيرات wrangler
غيّر `OWNER_ID` إلى رقم تيليجرام الرقمي لمالك البوت، و`CF_ACCOUNT_ID` إلى Cloudflare Account ID، و`CF_WORKERS_SUBDOMAIN` إلى workers.dev subdomain الخاص بك. اترك `WORKER_PREFIX` كما هو. هذه ليست أسرارًا، لكن لا تضع التوكن في هذا الملف.

## 4. أنشئ API Token محدود الصلاحيات
من Cloudflare > My Profile > API Tokens، أنشئ Token بصلاحيات Workers Scripts Edit اللازمة للنشر وإدارة Worker، وأي صلاحيات إضافية تطلبها عمليات إعداد النطاق/الأسرار. قلّل النطاق إلى حسابك فقط. صلاحيات API الدقيقة قد تتغير؛ راجع أخطاء API ووثائق Cloudflare قبل التوسيع. هذا المفتاح حساس جدًا لأنه يستطيع نشر كود ضمن حسابك.

## 5. اربط GitHub Actions
في GitHub > Settings > Secrets and variables > Actions أضف Repository secrets:
- `CLOUDFLARE_API_TOKEN`: مفتاح نشر البوت الرئيسي.
- `CLOUDFLARE_ACCOUNT_ID`: معرّف حساب Cloudflare.
ادفع الملفات إلى فرع `main`؛ سيحاول GitHub Actions تشغيل `npm install` و`npm run check` ثم `wrangler deploy`. راقب تبويب Actions. إذا فشل TypeScript check لا تعتبر النشر ناجحًا.

## 6. إعداد أسرار Worker الرئيسي
من Cloudflare Worker > Settings > Variables and Secrets أضف Secret لكل اسم:
- `BOT_TOKEN`: توكن البوت الرئيسي (ليس توكن مستخدم).
- `WEBHOOK_SECRET`: سلسلة عشوائية آمنة من حروف وأرقام و `_` و `-` بطول 16 إلى 64.
- `CF_API_TOKEN`: توكن Cloudflare الخاص بالنشر الديناميكي للمشاريع.
- `TOKEN_ENCRYPTION_KEY`: مفتاح عشوائي بطول 32 بايت ومشفّر Base64؛ مثال توليد محلي: `openssl rand -base64 32`.
لا تضع هذه القيم في GitHub أو رسائل عامة. إذا أعيد نشر Worker، تحقق من استمرار وجود الأسرار.

## 7. اربط Webhook للبوت الرئيسي
بعد نجاح النشر، احصل على رابط Worker من Cloudflare. عنوان webhook هو:
`https://YOUR-WORKER.YOUR-SUBDOMAIN.workers.dev/telegram/webhook`

نفّذ طلب POST إلى `https://api.telegram.org/bot<BOT_TOKEN>/setWebhook` باستخدام JSON يحتوي `url` و`secret_token` (القيمة نفسها `WEBHOOK_SECRET`). مثال من جهازك بعد استبدال المتغيرات البيئية محليًا:

```bash
curl -fsS "https://api.telegram.org/bot${BOT_TOKEN}/setWebhook" \
  -H 'content-type: application/json' \
  -d "{\"url\":\"${WORKER_URL}/telegram/webhook\",\"secret_token\":\"${WEBHOOK_SECRET}\"}"
```

لا تستخدم getUpdates أو polling لنفس توكن البوت بعد إعداد webhook.

## 8. الاختبار
1. افتح `https://YOUR-WORKER.../health` وتأكد من `ok`.
2. افتح البوت الرئيسي وأرسل `/start`.
3. جرّب موافقة المالك على مستخدم تجريبي.
4. ارفع ملف JS صغيرًا متوافقًا مع Workers.
5. أرسل توكن بوت تجريبي في المحادثة الخاصة بالأمر `/token PROJECT_ID BOT_TOKEN` ثم احذف الرسالة من تيليجرام. الأفضل مستقبلًا واجهة إدخال سرية بدل إرسال التوكن داخل الرسائل.
6. اضغط نشر، تحقق من رابط Worker المستقل، ثم أرسل رسالة للبوت التجريبي.
7. اختبر إيقاف المشروع وحذفه وتحقق من إزالة Webhook.

## القيود المهمة
- هذا المشروع لم يخضع لاختبار نشر حي هنا؛ لا تفتح المنصة لعامة المستخدمين قبل اختبارات أمان وفوترة وحصص.
- النشر إلى حسابك الرئيسي يجعل المشاريع تتشارك حصص حساب Cloudflare، وقد تستهلك الموارد أو تتسبب في إساءة استخدام. يلزم إضافة مراجعة صارمة، وحدود لكل مستخدم، ومراقبة قبل الإطلاق العام.
- لا يمكن تشغيل بوتات Python أو Node.js التقليدية المستمرة أو المشاريع ذات imports خارجية بهذه النسخة.
- ملف ZIP لا يشغّل مشروعًا كاملاً متعدد الملفات، بل يختار ملف Worker واحدًا.
- لا توجد ضمانة أن كل كود JS/TS مرفوع يعمل، حتى لو اجتاز فحص النص.
- لا توجد نسخ احتياطية كاملة أو لوحة مالك مطابقة 100% لـ V7 حتى الآن.
