# MSA To Full Mark

تطبيق أندرويد لمساعد دراسي ذكي (package: `com.msa.fullmark`).

مفتاح Gemini **مش** متسجل جوه التطبيق خالص. التطبيق بيكلم proxy صغير شغال
على Cloudflare Workers (مجاني)، وهو اللي بيحتفظ بالمفتاح ويكلم Gemini بالنيابة
عن التطبيق. ده عشان لو حد فك ضغط الـ APK بعد النشر، مش هيقدر يطلع المفتاح.

## الخطوة 1: نشر الـ Worker (المفتاح هيتخزن هنا)

1. اعمل حساب مجاني على https://dash.cloudflare.com/sign-up (لو معندكش).
2. من لوحة تحكم Cloudflare: **My Profile → API Tokens → Create Token** → اختار
   template "Edit Cloudflare Workers"، وخد الـ Token.
3. من نفس اللوحة، هتلاقي **Account ID** ظاهر في الصفحة الرئيسية (Workers & Pages).
4. ارفع المشروع ده كامل على ريبو جديد في GitHub.
5. من **Settings → Secrets and variables → Actions** في الريبو، ضيف 3 secrets:
   - `CLOUDFLARE_API_TOKEN` → التوكن من خطوة 2
   - `CLOUDFLARE_ACCOUNT_ID` → الـ Account ID من خطوة 3
   - `GEMINI_API_KEY` → مفتاح Gemini بتاعك (هيتخزن كـ secret جوه الـ Worker بس)
6. من تبويب **Actions**، شغّل workflow اسمه **Deploy Worker Proxy**.
7. بعد ما يخلص بنجاح، روح على Cloudflare Dashboard → Workers & Pages، هتلاقي
   Worker اسمه `msa-full-mark-proxy` وجنبه رابطه (شكله
   `https://msa-full-mark-proxy.<اسم-حسابك>.workers.dev`).

## الخطوة 2: تحديث رابط الـ proxy في التطبيق

1. افتح الملف `app/src/main/assets/script.js`.
2. غيّر السطر:
   ```js
   const PROXY_URL = "https://msa-full-mark-proxy.YOUR-SUBDOMAIN.workers.dev";
   ```
   حط رابط الـ Worker الحقيقي اللي طلع في الخطوة السابقة.
3. احفظ الملف وارفعه (commit) على نفس الريبو.

## الخطوة 3: بناء الـ APK

1. من تبويب **Actions**، شغّل workflow اسمه **Build APK**.
2. بعد النجاح، هتلاقي الـ APK جاهز في **Artifacts** باسم `msa-to-full-mark-debug`.

## الشاشات

1. **رفع مذاكرة** – رفع PDF أو صورة وسؤال المساعد عنها (بيرفض أي سؤال خارج الدراسة).
2. **التسجيل الصوتي** – تسجيل مباشر أو رفع تسجيل، وتحويله لنص ثم تلخيصه.
3. **المميزات** – صفحة تعريفية بالتطبيق + اسم المصمم محمد سعد.

## الأمان

- الضابط اللي بيمنع الرد على أي حاجة خارج الدراسة موجود جوه الـ Worker بس،
  مش جوه التطبيق، عشان محدش يقدر يتخطاه بفك ضغط الـ APK.
- مفتاح Gemini موجود كـ secret جوه Cloudflare بس، وميتسجلش في أي كود بيتنشر.
- لو حبيت تغيّر المفتاح بعدين: غيّر قيمة الـ `GEMINI_API_KEY` secret في GitHub،
  وشغّل workflow "Deploy Worker Proxy" تاني.
