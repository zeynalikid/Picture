# گزارش کلاد: جایگزین رایگان ChatbotX برای @zeynalikids

تاریخ بررسی: ۱۸ اکتبر ۲۰۲۶ — تمام قیمت‌ها و محدودیت‌ها از صفحات رسمی و مستندات همان تاریخ استخراج شده است. هر ادعا یا لینک مستقیم دارد یا صراحتاً «تأییدنشده» علامت خورده است.

---

## ۰. خلاصه اجرایی

**نتیجه: راه‌حل صفر‌دلاری وجود دارد و خود Zernio است.** دو فرض در پرسش اصلی باید اصلاح شود، چون هر دو به نفع شماست:

1. **Inbox دیگر «افزونه پولی» نیست.** در مدل قیمت‌گذاری فعلی، Inbox (کامنت، دایرکت، ریویو) روی هر اکانت متصل **شامل** است و دو اکانت اول بدون کارت اعتباری و بدون محدودیت زمانی رایگان‌اند.
2. **سقف ۱۰٬۰۰۰ فقط برای «دایرکت‌های ارسالی» است، نه کل پیام‌ها.** پیام‌های دریافتی، **پاسخ به کامنت‌ها** و پاسخ به ریویوها همیشه رایگان و نامحدودند. بعد از ۱۰٬۰۰۰ پیام ارسالی، هزینه ۱ دلار به ازای هر ۱۰٬۰۰۰ پیام است (یعنی ۵۰٬۰۰۰ پیام ارسالی در ماه = ۴ دلار).

**مصرف واقعی شما:** اگر بیشتر حجم ۵۰۰ تا ۱۰۰۰ پیام روزانه کامنت باشد و پاسخ عمومی (public reply) بگیرد، مصرف سهمیه بسیار کم می‌شود. فقط وقتی همه‌چیز دایرکت باشد (۳۰٬۰۰۰ در ماه) از سقف عبور می‌کنید؛ آن هم با کارت اعتباری فقط ≈ ۲ دلار.

**نکته حیاتی:** بدون کارت اعتباری، ارسال پس از ۱۰٬۰۰۰ پیام با خطای `403 message_allowance_exceeded` شکست می‌خورد (نه اینکه هزینه شود). پس اگر کارت بین‌المللی ندارید، طراحی باید سقف سخت‌گیرانه در Supabase اعمال کند (بخش ۶ و ۸).

**دو ریسک غیرفنی که باید بدانید** (بخش ۱۰): پلن Hobby ورسل «غیرتجاری» است، و پروژه Supabase رایگان بعد از ۷ روز بی‌فعالیتی pause می‌شود.

---

## ۱. وضعیت دقیق Zernio (قیمت و محدودیت‌ها)

| مورد | وضعیت (اکتبر ۲۰۲۶) | منبع |
|---|---|---|
| اکانت‌های رایگان | ۲ اکانت اول، دائمی، بدون کارت | [zernio.com/pricing](https://zernio.com/pricing) |
| Inbox (DM + کامنت + ریویو) | روی هر اکانت شامل است؛ بدون افزونه | [Comments API](https://zernio.com/social-media-comments) |
| دایرکت ارسالی | ۱۰٬۰۰۰ در ماه رایگان (سقف برای کل تیم، نه هر اکانت)، سپس $0.0001 به‌ازای هر پیام ($1 به‌ازای ۱۰٬۰۰۰) | [Messages Pricing](https://docs.zernio.com/pricing/messages) |
| پیام دریافتی، پاسخ به کامنت، پاسخ به ریویو | رایگان و نامحدود | همان صفحه |
| شروع شمارش | ۱ اکتبر ۲۰۲۶ (ارسال‌های قبل از آن شمرده شده ولی هرگز هزینه نشده) | همان صفحه |
| بدون کارت | ایمیل هشدار در ۷٬۵۰۰ پیام؛ ارسالِ بعد از ۱۰٬۰۰۰ → `403 message_allowance_exceeded` | همان صفحه |
| با کارت | چیزی متوقف نمی‌شود؛ اضافه‌مصرف محاسبه می‌شود | همان صفحه |
| شمارنده مصرف | **هیچ endpoint ای مصرف پیام‌ها را گزارش نمی‌کند** → شمارنده محلی لازم است | همان صفحه |
| ریست سهمیه | ساعت 00:00 UTC روز اول هر ماه؛ انتقال به ماه بعد ندارد | همان صفحه |
| سقف درخواست API (پلن رایگان) | ۶۰ درخواست در دقیقه؛ از اکانت سوم به بعد ۶۰۰ | [Rate Limits](https://docs.zernio.com/guides/rate-limits) |
| Webhook ها | `message.received`، `comment.received`، `message.sent` و … ؛ امضای HMAC-SHA256 در هدر `X-Zernio-Signature` | [Webhooks](https://docs.zernio.com/webhooks)، [Inbox webhooks](https://docs.zernio.com/webhooks/inbox) |

**ریز محاسبه هزینه پیام (طبق جدول رسمی):** ۸٬۰۰۰ و ۱۰٬۰۰۰ → رایگان؛ ۵۰٬۰۰۰ → $4؛ ۲۵۰٬۰۰۰ → $24.

> ملاحظه: برخی مقاله‌های قدیمی (و حتی یک صفحه راهنمای adapter) هنوز از «inbox addon» نام می‌برند؛ این برای ساختار قیمت قدیمیِ Late/Zernio است. مستندات فعلی خلاف آن را می‌گوید. اگر اکانت شما قدیمی (legacy) است، در داشبورد بررسی کنید که Inbox فعال باشد.

---

## ۲. محدودیت‌های Meta (دور زدنی نیستند؛ روی هر ابزاری اعمال می‌شوند)

- **پاسخ خصوصی به کامنت (private reply):** فقط یک پیام برای هر کامنت، حداکثر ۷ روز بعد از کامنت ([Zernio: Send private reply](https://docs.zernio.com/comments/send-private-reply-to-comment)). اگر تلاش ناموفق باشد (مثلاً دکمه برای غیرفالوور) همان یک فرصت هم می‌سوزد.
- **پیام‌های عادی:** پاسخ باید ظرف ۲۴ ساعت از آخرین پیام کاربر ارسال شود؛ خارج از آن فقط با برچسب `HUMAN_AGENT` (مخصوص پاسخ انسانی، تا ۷ روز) ([Zernio: Instagram](https://docs.zernio.com/platforms/instagram)).
- **نرخ:** ۷۵۰ private reply در ساعت، ۱۰۰ ارسال در ثانیه ([Inrō glossary به نقل از مستندات Meta، منبع ثانویه](https://www.inro.social/glossary/instagram-rate-limit)).
- **Message Requests:** DM از افراد ناشناس در پوشه Requests می‌نشیند؛ quick reply در آنجا نمایش داده نمی‌شود، دکمه (buttons) نمایش داده می‌شود.
- **اکانت:** فقط Business/Creator. برای دسترسی به کامنت/DM باید اتصال با scope های `comments` و `messaging` انجام شده باشد و در خود اینستاگرام دسترسی ابزارهای متصل به پیام‌ها فعال باشد (Settings → Website permissions → Connected tools).

---

## ۳. پاسخ سؤال ۱ — جایگزین رایگان ChatbotX؟ جدول مقایسه ۳ گزینه برتر

| معیار | **۱) Zernio** | **۲) Postproxy** | **۳) Ayrshare** |
|---|---|---|---|
| هزینه برای نیاز شما | **$0** (۱ اکانت اینستاگرام) | پلن رایگان فقط ۱۰ پست در ماه؛ پلن Agentic **$17/ماه** (۵ پروفایل، engagement نامحدود) | Premium **$149/ماه** + افزونه Messaging **$49/ماه** به‌ازای اکانت؛ در Launch/Business: **$0.09 به‌ازای هر conversation فعال** |
| پلن رایگان واقعی | ۲ اکانت، دائمی، بدون کارت | بله، ولی سقف engagement در پلن رایگان در صفحه قیمت **ذکر نشده** (تأییدنشده) | **ندارد**؛ فقط ۲۸ روز trial |
| کامنت و DM از طریق API | ✅ هر دو، به‌علاوه comment-to-DM و workflow داخلی | ✅ Comments API + DM API | ✅ کامنت از Premium به بالا؛ DM با افزونه |
| Webhook | ✅ (HMAC) | ✅ (روی همه پلن‌ها) | ✅ (Business به بالا) |
| سقف ماهانه | ۱۰٬۰۰۰ DM ارسالی رایگان؛ کامنت نامحدود | ندارد (پلن پولی) | ندارد/بر اساس conversation |
| سقف نرخ | ۶۰ req/min در پلن رایگان | نامشخص | نامشخص |
| نتیجه برای شما | **بهترین و تنها گزینه واقعاً رایگان** | در صورت نیاز به پلن پولی ارزان‌تر | غیرقابل‌توجیه (۱۵۰ تا ۲۰۰ دلار) |

**منابع:** [Zernio pricing](https://zernio.com/pricing) · [Postproxy pricing](https://postproxy.dev/pricing/) · [Ayrshare pricing](https://www.ayrshare.com/pricing/) · [Ayrshare Messaging API](https://www.ayrshare.com/docs/apis/messages/overview.md) · [مقایسه DEV، اکتبر ۲۰۲۶](https://dev.to/jesse_eisenbart/the-7-best-social-media-apis-for-developers-ai-agents-in-2026-3e15) (قیمت Ayrshare و نبود free tier).

### گزینه‌هایی که بررسی و کنار گذاشته شدند

| گزینه | دلیل رد |
|---|---|
| **Buffer** (رایگان: ۳ کانال، ۳٬۰۰۰ درخواست API در ماه) | Inbox API ندارد ([مقایسه](https://social-api.ai/blog/best-social-media-apis-2026)) |
| **Publer** (رایگان: ۳ اکانت) | فهرست امکانات پلن رایگان فقط زمان‌بندی است؛ در صفحات بررسی‌شده Inbox/DM API دیده نشد ([Publer Free](https://publer.com/help/en/article/dliovh/)) — «ندیدم»، نه «ثابت شد ندارد» |
| **Inrō** (رایگان: ۱۰۰ contact فعال در ماه) | HTTP webhooks و Private API در پلن رایگان **غیرفعال**؛ از پلن €12.99 به بالا ([Inrō pricing](https://inro.social/pricing)) |
| **InstantDM / LinkDM / ReplyRush** (رایگان با سقف ۱٬۰۰۰ تا ۱٬۵۰۰ DM) | منبع اطلاعات صفحات خود فروشندگان است؛ حلقه «webhook ورودی → LLM خودتان → پاسخ» در پلن رایگان گزارش نشده (اطمینان پایین) |
| **Unipile** | رایگان ندارد؛ ۷ روز trial و سپس €49/ماه (به نقل از [صفحه مقایسه Zernio](https://zernio.com/alternatives/unipile)، منبع غیرمستقل) |
| ManyChat، Vardast، Postiz، n8n، instagrapi، اتوماسیون مرورگر | طبق درخواست شما بررسی نشد |

---

## ۴. پاسخ سؤال ۲ — API رسمی بدون Meta App شخصی؟

**بله.** Zernio خودش «اپ تأییدشده Meta» است. شما از طریق OAuth (با Instagram Login، بدون نیاز به Facebook Page) اکانت را به آن وصل می‌کنید و Zernio با API رسمی Meta کامنت و DM را دریافت و ارسال می‌کند ([Zernio: Instagram](https://docs.zernio.com/platforms/instagram)).

جریان کار برای Vercel شما:

- Zernio روی هر کامنت/پیام جدید یک `POST` (webhook) به endpoint شما می‌زند.
- شما همان لحظه 2xx برمی‌گردانید و کار سنگین را async انجام می‌دهید (راهنمای رسمی: ack سریع، پردازش در job).
- شناسه رویداد webhook را به‌عنوان کلید dedup نگه دارید.
- پاسخ را با REST API به Zernio می‌فرستید.

**لایه هوشمند صرفه‌جویی** قبل از هر ارسال اجرا می‌شود (بخش ۶). نکته: چون Zernio به Meta به‌صورت یک اپلیکیشن واحد وصل است، برخی سقف‌های سمت پلتفرم بین همه مشتریان مشترک است ([Rate Limits](https://docs.zernio.com/guides/rate-limits)).

**پوشش کامنت:** webhook مستقیم کامنت می‌آورد؛ علاوه‌بر آن هر ۱۰ دقیقه Zernio خودش آخرین کامنت‌ها (پست مشخص‌شده یا ۵ پست جدید برای اتوماسیون سطح‌اکانت) را می‌خواند و اگر webhook گم شده باشد پردازش می‌کند؛ هر کامنت حداکثر یک‌بار پاسخ می‌گیرد ([ManyChat-style automations](https://docs.zernio.com/guides/manychat-patterns)).

---

## ۵. پاسخ سؤال ۳ — راه Serverless با Supabase + LLM و یک درخواست ساده

**بله، و به MCP هم نیازی ندارید.** ارسال به اینستاگرام یک `POST` عادی HTTPS به Zernio است:

| کار | Endpoint | هزینه سهمیه |
|---|---|---|
| پاسخ به DM | `POST https://zernio.com/api/v1/inbox/conversations/{conversationId}/messages` ([مستندات](https://docs.zernio.com/messages/send-inbox-message)) | ۱ |
| پاسخ عمومی به کامنت | `POST https://zernio.com/api/v1/inbox/comments/{postId}` با `commentId` در body ([مستندات](https://docs.zernio.com/comments/reply-to-inbox-post)) | **۰** |
| پاسخ خصوصی به کامنت | `POST .../inbox/comments/{postId}/{commentId}/private-reply` ([مستندات](https://docs.zernio.com/comments/send-private-reply-to-comment)) | ۱ |

هر دو endpoint هدر `Idempotency-Key` را می‌پذیرند (نگهداری ۲۴ ساعت)؛ برای retry بی‌خطر از `event_key` استفاده کنید. سرور MCP خود Zernio (`mcp.zernio.com`) برای مدیریت توسط agent مناسب است، نه برای مسیر runtime ربات.

### لایه LLM رایگان — واقع‌بینانه

| ارائه‌دهنده | وضعیت رایگان | ملاحظه |
|---|---|---|
| Gemini API (Flash / Flash-Lite) | محدودیت‌ها بارها کاهش یافته؛ گزارش‌های سپتامبر ۲۰۲۶: حدود ۲۰ درخواست/روز برای Flash نسل ۳ و حدود ۵۰۰ برای Flash-Lite ([منبع](https://blog.laozhang.ai/en/posts/gemini-api-pricing.md)) | برای ۵۰۰ تا ۱۰۰۰ رویداد روزانه **قابل اتکا نیست** |
| Mistral (Experiment) | ≈ ۱ درخواست/ثانیه، ۵۰۰K توکن/دقیقه، ≈ ۱B توکن/ماه ([منبع](https://mintlify.com/cheahjs/free-llm-api-resources/providers/free/mistral-plateforme)) | نیاز به تأیید شماره تلفن؛ **ورود به برنامه استفاده از داده برای آموزش مدل** الزامی |

**توصیه معماری:** LLM را فقط مرحله آخر قرار دهید. ابتدا قاعده/FAQ در Supabase (کلیدواژه → پاسخ آماده)؛ فقط پرسش‌های باقی‌مانده به LLM بروند. این معمولاً بخش عمده حجم را بدون مصرف توکن پاسخ می‌دهد (تخمین طراحی؛ ثابت‌شده نیست). به‌دلیل ماهیت پیام‌ها (پرسش والدین درباره سلامت و تغذیه کودک)، داده بالینی را به سرویس‌هایی که داده را برای آموزش مدل مصرف می‌کنند نفرستید و پرسش‌های بالینی را به شما ارجاع دهید (بخش ۶).

---

## ۶. پاسخ سؤال ۴ — مدیریت سهمیه، صف و اسپم زیر ۱۰٬۰۰۰ پیام

### ۶.۱ قابلیت‌های خود Zernio برای جلوگیری از تکرار

| قابلیت | عملکرد |
|---|---|
| `repeatPolicy: once` (پیش‌فرض) | هر نفر برای هر اتوماسیون فقط یک DM می‌گیرد |
| `repeatPolicy: every_comment` + `cooldownHours` | هر کامنت واجد شرایط است، ولی تکرار در بازه cooldown نادیده گرفته می‌شود |
| `dedupeSameTextHours` | اگر همان متن (پس از شخصی‌سازی) ظرف N ساعت از همین اکانت به همان گیرنده رفته باشد، ارسال نمی‌شود (ثبت `skipped`) |
| `cooldownHours` در workflow (۱ تا ۷۲۰) | جلوگیری از شروع مجدد workflow برای همان مخاطب |
| `publicReplyPolicy` | پاسخ عمومی `always` یا فقط `after_dm` |
| `Idempotency-Key` | جلوگیری از ارسال دوباره هنگام retry |

منبع: [ManyChat-style automations](https://docs.zernio.com/guides/manychat-patterns)، [Comment to DM](https://zernio.com/comment-to-dm).

**Block کردن کاربر:** در مستندات بررسی‌شده endpoint برای block کردن کاربر **پیدا نکردم** (تأییدنشده). endpoint های Hide، Delete و Set moderation status برای کامنت وجود دارد. راه عملی: جدول `contacts.blocked` در Supabase و رد کردن پیام پیش از هر ارسال (که هزینه سهمیه ندارد)، و block دستی در خود اینستاگرام.

### ۶.۲ سیاست «طبقه‌بندی قبل از ارسال»

| طبقه | شرط | اقدام | مصرف سهمیه |
|---|---|---|---|
| ۰ — نادیده | فقط ایموجی، کمتر از ۳ نویسه، کامنت خود اکانت، کاربر مسدود، متن تکراری اخیر | هیچ | ۰ |
| ۱ — پاسخ عمومی | سؤال عمومی یا تعامل | پاسخ عمومی کوتاه (قالب/FAQ یا LLM) | **۰** |
| ۲ — DM | فقط وقتی لازم است (قیمت، نوبت، لینک، اطلاعات خصوصی) | private reply یا DM | ۱ |
| ۳ — ارجاع انسانی | پرسش بالینی (دوز، بیماری، علائم)، شکایت، کلمات حساس | اعلان در Telegram و پاسخ دستی؛ برای بعد از ۲۴ ساعت با `HUMAN_AGENT` | ۰ تا ۱ |

### ۶.۳ بودجه و شمارنده

- چون Zernio مصرف را گزارش نمی‌دهد، شمارنده خودتان را بسازید (تابع `reserve_send` در بخش ۸). ماه را با **UTC** حساب کنید.
- پیشنهاد: **سقف نرم ۹٬۰۰۰** (۱٬۰۰۰ ذخیره برای پاسخ‌های انسانی/اضطراری). بودجه روزانه = (۹٬۰۰۰ − مصرف تاکنون) ÷ روزهای باقی‌مانده.
- هشدار هماهنگ: Zernio در ۷٬۵۰۰ ایمیل می‌دهد (فقط بدون کارت)؛ شمارنده شما باید زودتر در Telegram هشدار بدهد (مثلاً ۸۰٪).
- ارسال ناموفق هزینه ندارد؛ پس در صورت خطا شمارنده را برگردانید (`release_send`).

### ۶.۴ سناریوهای هزینه (ارسال‌های ماهانه = DM و private reply؛ پاسخ عمومی کامنت حساب نمی‌شود)

| سناریو | ارسال/ماه | با کارت | بدون کارت |
|---|---|---|---|
| ۳۰۰ DM در روز | ۹٬۰۰۰ | $0 | $0 |
| ۵۰۰ DM در روز | ۱۵٬۰۰۰ | $0.50 | توقف حدود روز ۲۰ ماه |
| ۱٬۰۰۰ DM در روز | ۳۰٬۰۰۰ | $2.00 | توقف حدود روز ۱۰ ماه |

(فرمول: (ارسال − ۱۰٬۰۰۰) ÷ ۱۰٬۰۰۰ × $1 — با جدول رسمی هماهنگ است.)

---

## ۷. پاسخ سؤال ۵ — پل خلاقانه با Telegram

**پل بدون هیچ محدودیت وجود ندارد**، به دو دلیل: (الف) ارسال به اینستاگرام فقط از طریق اپی که مجوز Meta دارد ممکن است؛ Telegram چنین دسترسی‌ای ندارد؛ (ب) حتی بدون Zernio، سقف‌های خود Meta (بخش ۲) اعمال می‌شود.

کاری که Telegram **می‌تواند** بکند و ارزش دارد:

1. **کنسول انسانی (human-in-the-loop):** اعلان پرسش‌های طبقه ۳، نمایش پیش‌نویس LLM با دکمه «ارسال / ویرایش / رد».
2. **داشبورد مصرف:** دستور `/usage` برای نمایش شمارنده ماهانه و بودجه روزانه.
3. **کنترل اضطراری:** دستور pause/resume برای قطع کل پاسخ خودکار.

**ترفند قانونی برای کاهش بار:** اتوماسیون‌های ثابت (مثلاً «کامنت کن guide ← DM لینک + پاسخ عمومی») را با comment-to-DM داخلی Zernio بسازید؛ این‌ها بدون هیچ فراخوانی به Vercel اجرا می‌شوند. فقط DM های آزاد (free-form) را به مسیر LLM خودتان بدهید. به‌دلیل اینکه مستندات می‌گویند هر پیامی که Zernio از طرف شما تحویل دهد (شامل workflow ها) متر می‌شود، فرض کنید DM های اتوماسیون هم از همان ۱۰٬۰۰۰ کم می‌شوند.

مرجع کدباز مفید: [ZernFlow](https://github.com/zernio-dev/zernflow) (MIT؛ Next.js + Supabase + Zernio؛ شامل flow builder، inbox، comment-to-DM و cron). برای پروژه Python/FastAPI شما مرجع طراحی است، نه جایگزین مستقیم.

---

## ۸. معماری پیشنهادی (Vercel + Supabase + Zernio)

```
Instagram ⇄ Meta ⇄ Zernio (اپ رسمی)
                      │ webhook: message.received / comment.received
                      ▼
        Vercel  POST /api/zernio/webhook
        (۱) تأیید HMAC  (۲) INSERT در inbox_events (event_key یکتا = dedup)  (۳) پاسخ 200
                      │
                      ▼
        Supabase: جدول inbox_events (status = queued)
          ├─ Trigger + pg_net  → فوراً POST به /api/worker/drain
          └─ pg_cron هر ۱ دقیقه → همان endpoint (جارو برای رویدادهای گیرکرده)
                      │
                      ▼
        Vercel  POST /api/worker/drain  (محافظت‌شده با Bearer)
        claim_events(n)  [FOR UPDATE SKIP LOCKED]
          ├─ طبقه ۰: noise / blocklist / dedup  → skipped
          ├─ طبقه ۱: FAQ → (در صورت نیاز) LLM → پاسخ عمومی (رایگان)
          ├─ طبقه ۲: reserve_send() → DM / private reply
          └─ طبقه ۳: اعلان Telegram (ارجاع انسانی)
                      │
                      ▼
        Zernio REST  →  Instagram          و          Telegram bot (اعلان/کنترل)
```

**چرا این شکل؟**

- Cron ورسل در پلن Hobby فقط **روزی یک‌بار** و با دقت ±۵۹ دقیقه اجرا می‌شود ([Vercel docs](https://vercel.com/docs/cron-jobs/usage-and-pricing)). پس زمان‌بندی را به `pg_cron` در Supabase بسپارید؛ `pg_net` از داخل دیتابیس به endpoint ورسل HTTP می‌زند ([Supabase pg_net](https://supabase.com/docs/guides/database/extensions/pg_net)).
- Trigger روی INSERT باعث می‌شود پاسخ‌ها «تقریباً لحظه‌ای» بدون polling تولید شوند؛ cron فقط شبکه ایمنی است.
- پردازش در یک invocation جدا انجام می‌شود تا webhook سریع پاسخ دهد.

### ۸.۱ SQL (Supabase SQL Editor)

```sql
create extension if not exists pg_net;
create extension if not exists pg_cron;

create table inbox_events (
  id bigserial primary key,
  event_key text unique not null,            -- شناسه رویداد webhook (dedup)
  kind text not null,                        -- 'dm' | 'comment'
  account_id text not null,
  conversation_id text,
  post_id text,
  comment_id text,
  sender_id text,
  text text,
  payload jsonb not null,
  status text not null default 'queued',     -- queued|processing|done|skipped|failed
  attempts int not null default 0,
  created_at timestamptz default now(),
  locked_at timestamptz
);
create index on inbox_events(status, created_at);

create table contacts (
  sender_id text primary key,
  blocked boolean default false,
  last_dm_sent_at timestamptz
);

create table usage_monthly (month text primary key, sent int not null default 0);

create or replace function claim_events(n int)
returns setof inbox_events language sql as $$
  update inbox_events
     set status='processing', locked_at=now(), attempts=attempts+1
   where id in (
     select id from inbox_events
      where status='queued'
         or (status='processing' and locked_at < now() - interval '3 minutes')
         or (status='failed' and attempts < 3)
      order by created_at
      limit n
      for update skip locked)
  returning *;
$$;

create or replace function reserve_send(cap int)
returns boolean language plpgsql as $$
declare m text := to_char(now() at time zone 'utc','YYYY-MM'); ok boolean;
begin
  insert into usage_monthly(month) values (m) on conflict do nothing;
  update usage_monthly set sent = sent + 1
   where month = m and sent < cap returning true into ok;
  return coalesce(ok, false);
end $$;

create or replace function release_send() returns void language sql as $$
  update usage_monthly set sent = greatest(sent - 1, 0)
   where month = to_char(now() at time zone 'utc','YYYY-MM');
$$;

-- فراخوانی worker هنگام ثبت رویداد جدید
create or replace function notify_worker() returns trigger language plpgsql as $$
begin
  perform net.http_post(
    url     := 'https://YOUR-APP.vercel.app/api/worker/drain?limit=3',
    headers := jsonb_build_object('Authorization', 'Bearer YOUR_CRON_SECRET'),
    body    := '{}'::jsonb);
  return new;
end $$;
create trigger trg_notify after insert on inbox_events
  for each row execute function notify_worker();

-- جارو هر یک دقیقه
select cron.schedule('drain-every-minute', '* * * * *', $$
  select net.http_post(
    url     := 'https://YOUR-APP.vercel.app/api/worker/drain?limit=10',
    headers := jsonb_build_object('Authorization', 'Bearer YOUR_CRON_SECRET'),
    body    := '{}'::jsonb) $$);
```

> برای تولید واقعی، `YOUR_CRON_SECRET` را در Supabase Vault نگه دارید ([مستندات scheduling](https://supabase.com/docs/guides/functions/schedule-functions.md)).

### ۸.۲ اسکلت FastAPI (آزمایش‌نشده — نام فیلدهای payload را با «Test webhook» و «Webhook logs» داشبورد Zernio تطبیق دهید)

```python
# api/index.py
import os, hmac, hashlib, httpx
from fastapi import FastAPI, Request, HTTPException
from supabase import create_client

app = FastAPI()
sb = create_client(os.environ["SUPABASE_URL"], os.environ["SUPABASE_SERVICE_KEY"])
ZB = "https://zernio.com/api/v1"
ZH = {"Authorization": f"Bearer {os.environ['ZERNIO_API_KEY']}"}
CAP = 9000

def _valid(raw: bytes, sig: str) -> bool:
    mac = hmac.new(os.environ["ZERNIO_WEBHOOK_SECRET"].encode(), raw, hashlib.sha256).hexdigest()
    return hmac.compare_digest(mac, sig or "")   # TODO: فرمت دقیق امضا را تأیید کنید

@app.post("/api/zernio/webhook")
async def webhook(req: Request):
    raw = await req.body()
    if not _valid(raw, req.headers.get("x-zernio-signature", "")):
        raise HTTPException(401)
    ev = await req.json()
    # TODO: نگاشت فیلدها طبق payload واقعی
    row = {
        "event_key": ev.get("id"), "kind": "dm" if ev.get("event") == "message.received" else "comment",
        "account_id": ev.get("accountId"), "conversation_id": ev.get("conversationId"),
        "payload": ev,
    }
    sb.table("inbox_events").upsert(row, on_conflict="event_key", ignore_duplicates=True).execute()
    return {"ok": True}

async def send_dm(c, ev, text):
    if not sb.rpc("reserve_send", {"cap": CAP}).execute().data:
        return "budget"
    r = await c.post(f"{ZB}/inbox/conversations/{ev['conversation_id']}/messages",
        headers={**ZH, "Idempotency-Key": f"dm-{ev['event_key']}"},
        json={"accountId": ev["account_id"], "message": text})
    if r.status_code >= 300:
        sb.rpc("release_send").execute()
    return r.status_code

async def reply_comment(c, ev, text):            # رایگان و نامحدود
    r = await c.post(f"{ZB}/inbox/comments/{ev['post_id']}",
        headers={**ZH, "Idempotency-Key": f"cm-{ev['event_key']}"},
        json={"accountId": ev["account_id"], "commentId": ev["comment_id"], "message": text})
    return r.status_code

@app.post("/api/worker/drain")
async def drain(req: Request, limit: int = 5):
    if req.headers.get("authorization") != f"Bearer {os.environ['CRON_SECRET']}":
        raise HTTPException(401)
    rows = sb.rpc("claim_events", {"n": limit}).execute().data or []
    async with httpx.AsyncClient(timeout=20) as c:
        for ev in rows:
            # 1) noise/blocklist/dedup  2) FAQ  3) LLM  4) escalation → Telegram
            # 5) reply_comment (طبقه ۱) یا send_dm (طبقه ۲)
            ...
    return {"claimed": len(rows)}
```

### ۸.۳ مدل داده و پیکربندی

- **متغیرهای محیطی ورسل:** `ZERNIO_API_KEY`، `ZERNIO_WEBHOOK_SECRET`، `SUPABASE_URL`، `SUPABASE_SERVICE_KEY`، `CRON_SECRET`، کلید LLM.
- **Webhook در Zernio:** فقط رویدادهای `message.received` و `comment.received` (برای شروع)؛ `message.sent` اختیاری برای تطبیق مصرف.
- **سقف زمانی ورسل:** برخی منابع ۲۰۲۶ برای Hobby سقف ۳۰۰ ثانیه (با Fluid Compute) و برخی ۱۰ ثانیه می‌گویند؛ در داشبورد خودتان بررسی کنید و worker را با `limit` کوچک اجرا کنید.
- **نرخ:** پلن رایگان ۶۰ درخواست/دقیقه؛ worker را در حدود ۴۰ درخواست/دقیقه throttle کنید.

---

## ۹. مقایسه نهایی هزینه معماری

| جزء | هزینه |
|---|---|
| Zernio (۱ اکانت) | $0 |
| Supabase Free | $0 (۵۰۰MB دیتابیس، ۵۰۰K فراخوانی Edge Function، کاربر تجاری مجاز) |
| Vercel Hobby | $0 — **ولی غیرتجاری** (بخش ۱۰) |
| LLM | $0 در حد آزمایش؛ برای حجم واقعی احتمالاً پولی یا ترکیبی با AvalAI |
| اضافه‌مصرف پیام | $0 تا $2 (فقط با کارت) |

---

## ۱۰. ریسک‌ها و نکات تأییدنشده

1. **Vercel Hobby غیرتجاری است.** صفحه قیمت Vercel می‌گوید Hobby برای استفاده شخصی و غیرتجاری است ([توضیح](https://zplatform.ai/guides/is-vercel-free/)، به نقل از FAQ رسمی). کسب‌وکار زینالیکید تجاری است؛ این یک ریسک سیاستی است، نه فنی. پلن Pro $20/ماه است و با هدف «صفر دلار» نمی‌خواند. تصمیم با شماست.
2. **Pause شدن Supabase رایگان بعد از ۷ روز بی‌فعالیتی** ([منبع](https://www.itpathsolutions.com/supabase-free-tier-limits)). ترافیک webhook معمولاً فعالیت می‌سازد، ولی اگر پیج یک هفته ساکت بماند ریسک دارد.
3. **تغییر Data API سوپابیس:** برای پروژه‌های رایگان موجود از ۳۰ اکتبر ۲۰۲۶ باید grant های صریح Postgres برای دسترسی PostgREST تعریف شود. تا آن تاریخ بررسی کنید که ربات شما (supabase-py با service key) بعد از تغییر هنوز به جدول‌ها دسترسی دارد (دامنه دقیق تغییر را در مستندات Supabase تأیید کنید).
4. **وابستگی به Zernio:** اگر اپ Meta آن محدود شود، همه مشتریان آن اثر می‌گیرند. معماری باید تعویض‌پذیر باشد: فقط دو تابع `send_dm` و `reply_comment` به Zernio وابسته‌اند.
5. **کارت اعتباری:** پرداخت Zernio از طریق Stripe با Visa/Mastercard/Amex است ([Zernio pricing](https://zernio.com/pricing))؛ اگر کارت بین‌المللی ندارید، سقف ۱۰٬۰۰۰ برای شما «سقف سخت» است.
6. **LLM و داده کودکان:** پلن‌های رایگان بعضی سرویس‌ها داده را برای آموزش مدل می‌خواهند (Mistral Experiment این را الزامی می‌کند). برای پرسش‌های بالینی از LLM استفاده نکنید و آن‌ها را به خودتان ارجاع دهید.
7. **تأییدنشده:** نام دقیق فیلدهای payload و فرمت امضای `X-Zernio-Signature`؛ نبود endpoint برای block کاربر؛ سقف engagement پلن رایگان Postproxy؛ نبود Inbox API در Publer.

---

## ۱۱. چک‌لیست اجرا (به ترتیب)

1. در Zernio، اکانت اینستاگرام را با scope های `comments` و `messaging` وصل (یا دوباره وصل) کنید؛ در اینستاگرام دسترسی ابزارهای متصل به پیام‌ها را روشن کنید.
2. Webhook بسازید (URL + secret + دو رویداد) و با **Test webhook** و **Webhook logs** ساختار payload را ببینید.
3. SQL بخش ۸.۱ را اجرا کنید؛ secret ها را در Vault بگذارید.
4. دو endpoint را روی ورسل مستقر کنید.
5. **حالت سایه (shadow mode) ۴۸ ساعت:** فقط تصمیم‌ها را در Telegram لاگ کنید، بدون ارسال.
6. ابتدا پاسخ عمومی کامنت‌ها را فعال کنید (بدون هزینه سهمیه)، سپس DM.
7. دستور `/usage` و هشدار ۸۰٪ را در ربات Telegram بیفزایید.
8. تا ۳۰ اکتبر وضعیت grant های Supabase را بررسی کنید.
9. تکلیف پلن Vercel را مشخص کنید.

---

## ۱۲. منابع

**Zernio:** [Pricing](https://zernio.com/pricing) · [Docs: Pricing](https://docs.zernio.com/pricing) · [Messages Pricing](https://docs.zernio.com/pricing/messages) · [Billing](https://docs.zernio.com/billing) · [Rate Limits](https://docs.zernio.com/guides/rate-limits) · [ManyChat-style automations](https://docs.zernio.com/guides/manychat-patterns) · [Instagram](https://docs.zernio.com/platforms/instagram) · [Webhooks](https://docs.zernio.com/webhooks) · [Inbox webhooks](https://docs.zernio.com/webhooks/inbox) · [Send message](https://docs.zernio.com/messages/send-inbox-message) · [Reply to comment](https://docs.zernio.com/comments/reply-to-inbox-post) · [Private reply](https://docs.zernio.com/comments/send-private-reply-to-comment) · [Comment-to-DM](https://zernio.com/comment-to-dm) · [ZernFlow](https://github.com/zernio-dev/zernflow) · [بررسی مستقل قیمت (DroidCrunch، ۲۸ سپتامبر ۲۰۲۶)](https://droidcrunch.com/zernio-pricing/)

**جایگزین‌ها:** [Postproxy pricing](https://postproxy.dev/pricing/) · [Ayrshare pricing](https://www.ayrshare.com/pricing/) · [Ayrshare Messaging API](https://www.ayrshare.com/docs/apis/messages/overview.md) · [Inrō pricing](https://inro.social/pricing) · [Publer Free](https://publer.com/help/en/article/dliovh/) · [مقایسه APIهای اجتماعی](https://dev.to/jesse_eisenbart/the-7-best-social-media-apis-for-developers-ai-agents-in-2026-3e15) · [مقایسه ۲](https://social-api.ai/blog/best-social-media-apis-2026)

**زیرساخت:** [Vercel Cron limits](https://vercel.com/docs/cron-jobs/usage-and-pricing) · [Vercel Hobby (منبع ثانویه)](https://zplatform.ai/guides/is-vercel-free/) · [Supabase pg_net](https://supabase.com/docs/guides/database/extensions/pg_net) · [Supabase scheduling](https://supabase.com/docs/guides/functions/schedule-functions.md) · [Supabase Free limits](https://www.itpathsolutions.com/supabase-free-tier-limits)

**Meta و LLM:** [Instagram Messaging overview (Meta)](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/overview) · [Instagram rate limits (منبع ثانویه)](https://www.inro.social/glossary/instagram-rate-limit) · [Gemini pricing/free tier](https://blog.laozhang.ai/en/posts/gemini-api-pricing.md) · [Mistral free tier](https://mintlify.com/cheahjs/free-llm-api-resources/providers/free/mistral-plateforme)
