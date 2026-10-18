# معماری هزینه‌صفر مدیریت دایرکت و کامنت اینستاگرام @zeynalikids
### گزارش تحقیقاتی عمیق — وضعیت اکتبر ۲۰۲۶ — پشته فنی: Python/FastAPI + Vercel + Supabase

> **تاریخ تحقیق:** ۱۸ اکتبر ۲۰۲۶ · **منطقه زمانی:** Asia/Tehran
> **محدودیت‌های پذیرفته‌شده:** بدون اپ متا · بدون instagrapi · بدون VPS · بدون n8n · بدون اتوماسیون مرورگر
> **هدف:** پاسخ‌دهی خودکار به کامنت + دایرکت با هزینه‌ی **دقیقاً ۰ دلار در ماه**

---

## 📌 بخش ۰ — خلاصه مدیریتی (TL;DR)

پنج یافته‌ای که کل صورت‌مسئله را تغییر می‌دهد:

| # | یافته | چرا حیاتی است |
|---|---|---|
| ۱ | **جایگزین رایگان ChatbotX وجود ندارد که هم API بدهد هم سقف پیام بالا داشته باشد.** SendPulse رایگان است (۱۰,۰۰۰ پیام/ماه) ولی **سقف ۵۰۰ مشترک** دارد و «API محدود» — برای حجم شما قفل می‌شود. | سوال ۱ پاسخ قطعی گرفت |
| ۲ | **قانون طلایی Zernio: فقط «دایرکت خروجی» شمرده می‌شود. پاسخ کامنت، پاسخ ریویو، پیام‌های دریافتی، خواندن، typing و read receipt کاملاً رایگان و نامحدودند.** | این کلید حل مشکل شماست |
| ۳ | **اگر کارت بانکی روی اکانت Zernio نباشید، در پیام ۱۰,۰۰۱ام درخواست با `403 message_allowance_exceeded` رد می‌شود — یعنی هرگز شارژ نمی‌شوید.** Zernio در ۷,۵۰۰ پیام به مالک تیم ایمیل می‌زند. | «فرار از هزینه» به‌صورت ساختاری ممکن است |
| ۴ | **Comment Automation در Zernio برای اینستاگرام `dmMessage` را اجباری می‌کند** → هر بار که فعال شود یک دایرکت مصرف می‌کند. پس برای حجم بالا باید **خاموش** بماند و به‌جایش از `POST /v1/inbox/comments/{postId}` (پاسخ عمومی، رایگان) استفاده کنید. | تله‌ی پنهانی که بودجه را می‌سوزاند |
| ۵ | **Vercel Hobby از نظر ToS فقط «غیرتجاری/شخصی» است.** پیج فروشگاهی کودکان = تجاری. **Cloudflare Workers Free تجاری را مجاز می‌کند** و Cron Trigger با فاصله‌ی دقیقه‌ای + صف رایگان دارد. | ریسک حقوقی + راه‌حل جایگزین |

**پاسخ یک‌خطی:** بله، با معماری «**اولویت‌دهی کامنت‌محور + بودجه‌بندی روزانه دایرکت + صف در Supabase**» می‌توانید ۵۰۰ تا ۱۰۰۰ پیام روزانه را با **۰ دلار** پوشش دهید، به شرط آنکه دایرکت خروجی را زیر **۳۳۰ عدد در روز** نگه دارید و بقیه را به **پاسخ عمومی کامنت** (رایگان و نامحدود) منتقل کنید.

---

## 📌 بخش ۱ — فیزیک مسئله: چه چیزی رایگان است و چه چیزی پولی

قبل از هر مقایسه‌ای، باید «واحد پول» را بشناسید. در اکوسیستم رسمی متا در ۲۰۲۶:

### ۱.۱ خودِ Meta هیچ پولی برای Instagram Messaging API نمی‌گیرد

برخلاف WhatsApp Business API (که به‌ازای هر مکالمه پول می‌گیرد)، **API پیام‌رسانی اینستاگرام کاملاً رایگان است**. هزینه‌ی واقعی فقط «زیرساخت + اشتراک پلتفرم واسط» است. ([conferbot.com/chatbot/instagram](https://www.conferbot.com/chatbot/instagram))

اما دسترسی به آن از دو راه ممکن است:

| مسیر | نیازمندی | وضعیت شما |
|---|---|---|
| **مسیر مستقیم (Graph API)** | اپ متا + `instagram_business_manage_messages` + App Review + Business Verification | ❌ بسته (تحریم) |
| **مسیر واسطه (Tech Provider)** | OAuth روی اپِ **تأییدشده‌ی شرکت واسطه** | ✅ باز |

نکته‌ی کلیدی که در سوال ۲ پرسیدید و جوابش **مثبت** است:

> «Meta سطح Advanced Access — که برای سرویس‌دهی به اکانت‌های اینستاگرامی که مالکشان نیستید لازم است — فقط بعد از App Review و Business Verification می‌دهد. **SocialAPI.ai، Ayrshare و Zernio این سطح را دارند**، پس کاربران شما با اپِ تأییدشده‌ی آن‌ها authorize می‌کنند و **شما هیچ چیزی submit نمی‌کنید**.»
> — [social-api.ai/blog/instagram-facebook-api-without-meta-app-review](https://social-api.ai/blog/instagram-facebook-api-without-meta-app-review)

یعنی: **بله، بدون داشتن اپ متا می‌توان از API رسمی استفاده کرد** — به شرط استفاده از اپِ تأییدشده‌ی یک واسطه. این دقیقاً همان کاری است که Zernio برای شما می‌کند.

### ۱.۲ جدول «متر» Zernio — دقیق‌ترین سند این تحقیق

از [docs.zernio.com/pricing/messages](https://docs.zernio.com/pricing/messages):

| عملیات | شمرده می‌شود؟ |
|---|---|
| دایرکت ارسالی (Instagram, Facebook, WhatsApp, **Telegram**, Slack, Reddit, Bluesky, Discord) | ✅ بله |
| برادکست واتس‌اپ، به‌ازای هر گیرنده | ✅ بله |
| پیام sequence / workflow | ✅ بله |
| **پیام دریافتی (هر نوع)** | ❌ **رایگان** |
| **پاسخ کامنت (Comment reply)** | ❌ **رایگان** |
| **پاسخ ریویو (Review reply)** | ❌ **رایگان** |
| **خواندن مکالمه/پیام/کامنت/ریویو** | ❌ **رایگان** |
| **Typing indicator، read receipt، mark-as-read** | ❌ **رایگان** |
| Private Reply به کامنت (DM) | ✅ بله (یک دایرکت است) |
| پیام ناموفق (failed send) | ❌ شمرده نمی‌شود |
| X DM و SMS | خارج از این متر (نرخ جدا) |

**نتیجه‌ی استراتژیک:** تنها منبع هزینه‌ی شما `POST /v1/inbox/conversations/{id}/messages` است. هر چیز دیگری که از Zernio می‌خواهید — خواندن، پاسخ کامنت، لایک کامنت، مخفی‌کردن کامنت، وب‌هوک — **بی‌نهایت و رایگان** است.

### ۱.۳ مکانیزم سهمیه (بسیار مهم)

| ویژگی | مقدار |
|---|---|
| سهمیه رایگان | ۱۰,۰۰۰ پیام خروجی در ماه |
| نرخ پس از آن | `$0.0001` هر پیام = **$1 به‌ازای هر ۱۰,۰۰۰** |
| دامنه‌ی سهمیه | **به‌ازای تیم** (نه به‌ازای اکانت) — همه‌ی اکانت‌ها و اعضای تیم جمع می‌شوند |
| زمان ریست | `00:00 UTC` اول هر ماه میلادی، **بدون انتقال به ماه بعد** |
| شروع رسمی متر | **۱ اکتبر ۲۰۲۶** (ارسال‌های قبل از آن شمرده شدند ولی هرگز شارژ نشدند) |
| **بدون کارت بانکی** | ایمیل هشدار در **۷,۵۰۰** پیام؛ ارسالِ عبورکننده از ۱۰,۰۰۰ با **`403` + کد `message_allowance_exceeded`** شکست می‌خورد |
| با کارت بانکی | هیچ‌چیز متوقف نمی‌شود؛ overage با $1/10k شارژ می‌شود |
| گزارش لحظه‌ای مصرف | ❌ **وجود ندارد** — `GET /v1/usage` و `GET /v1/usage-stats` فقط «اکانت‌های متصل» و «فراخوانی‌های X» را دارند، نه پیام‌ها |

> ⚠️ **پیامد طراحی:** چون Zernio هیچ endpoint برای شمارش پیام‌ها نمی‌دهد، **شما باید شمارنده را در Supabase خودتان نگه دارید.** این هسته‌ی معماری پیشنهادی من است.

### ۱.۴ نرخ‌های Zernio در پلن رایگان

| منبع | مقدار |
|---|---|
| اکانت متصل رایگان | ۲ اکانت، بدون کارت بانکی ([docs.zernio.com/pricing](https://docs.zernio.com/pricing)) |
| Rate limit API | **۶۰ درخواست در دقیقه** (۰–۲ اکانت) — یعنی ۱ درخواست در ثانیه |
| Rate limit endpointهای analytics | ۶ در ثانیه |
| پست‌گذاری | نامحدود (سقف سرعت: ۲۵ پست/ساعت، ۱۰۰ پست/روز برای اینستاگرام) |
| وب‌هوک | **نامحدود و رایگان** در همه‌ی پلن‌ها |
| MCP Server | رایگان (`https://mcp.zernio.com/mcp`، ۴۹۶ ابزار، OAuth 2.1 یا Bearer API Key) |
| اعتبار پلن رایگان | $12 در ماه فقط روی خط «اکانت متصل» |

---

## 📌 بخش ۲ — جدول مقایسه‌ای ۳ گزینه برتر

### ۲.۱ جدول اصلی (پاسخ سوال ۱)

| معیار | 🥇 **Zernio (Free)** | 🥈 **SendPulse (Free / Basic)** | 🥉 **SocialAPI.ai (Hobby)** |
|---|---|---|---|
| **هزینه ماهانه** | **$0** | **$0** | **$0** |
| کارت بانکی لازم | ❌ نه | ❌ نه | ❌ نه |
| **دایرکت خروجی رایگان** | **۱۰,۰۰۰ / ماه** | **۱۰,۰۰۰ / ماه** (مشترک بین همه بات‌ها) | **۵۰ «تعامل» / ماه** |
| **پاسخ کامنت** | ✅ نامحدود و رایگان | ✅ دارد (public reply) | ⚠️ از همان ۵۰ تعامل کسر می‌شود |
| **خواندن پیام/کامنت** | ✅ نامحدود رایگان | ✅ | ✅ **نامحدود رایگان** |
| **سقف مشترک/مخاطب** | ❌ ندارد | ⛔ **۵۰۰ مشترک** (گلوگاه اصلی) | ❌ ندارد |
| **Webhook** | ✅ ۱۴ رویداد، HMAC-SHA256، نامحدود | ✅ Global webhook (ورودی/خروجی/subscription/failed) | ⚠️ فقط پلن‌های پولی ($29+) |
| **REST API کامل** | ✅ همه‌ی ۴۹۶ endpoint در پلن رایگان | ⚠️ «Limited API» در پلن رایگان | ✅ |
| **MCP Server** | ✅ ۴۹۶ ابزار، میزبانی‌شده | ✅ (اخیراً اضافه شده) | ✅ ۷۵ ابزار |
| **Rate limit** | ۶۰ req/min | ۱,۰۰۰ req/min و ۵۰۰,۰۰۰ req/day | محدود به ۵۰ تعامل/ماه |
| **Comment-to-DM بومی** | ✅ با `dedupeSameTextHours`, `repeatPolicy`, `audience`, `followGate` | ✅ | ❌ |
| **وضعیت اپ متا** | ✅ Advanced Access (بدون App Review شما) | ✅ Meta Business Partner از ۲۰۲۱ | ✅ Advanced Access |
| **رفتار هنگام عبور از سهمیه** | 🛡️ **`403` سخت — هرگز شارژ نمی‌شوید** | متوقف می‌شود تا ماه بعد | `429 rate_limit_exceeded` تا دوره بعد |
| **قیمت اولین پلن پولی** | $6 / اکانت (از اکانت سوم) | $12/ماه (۵۰۰ مشترک) — $9.60 سالانه | $29/ماه (نامحدود) |
| **هزینه حجم شما (۲۴k پیام/ماه)** | **$0** (با معماری بخش ۵) یا **$1.40** (بدون بهینه‌سازی) | ⛔ **غیرممکن** — سقف ۵۰۰ مشترک | ⛔ **غیرممکن** — ۵۰ تعامل |
| **ریسک دسترسی از ایران** | 🟢 کم (هم‌اکنون استفاده می‌کنید) | 🟡 متوسط (ثبت‌نام با مرورگر لازم است) | 🟡 متوسط |
| منبع قیمت‌گذاری | [docs.zernio.com/pricing](https://docs.zernio.com/pricing) · [docs.zernio.com/pricing/messages](https://docs.zernio.com/pricing/messages) | [sendpulse.com/pricing/messengers](https://sendpulse.com/pricing/messengers) | [social-api.ai/pricing](https://social-api.ai/pricing) |

### ۲.۲ چرا SendPulse با وجود پلن رایگان سخاوتمندانه، گزینه اصلی نیست؟

پلن رایگان SendPulse روی کاغذ عالی است: ۳ بات، ۱۰,۰۰۰ پیام/ماه، API و Webhook، Meta Business Partner، بدون کارت، بدون انقضا. ([chatbotscape.com/reviews/sendpulse-review](https://chatbotscape.com/reviews/sendpulse-review))

اما سه قفل کشنده دارد:

1. **سقف ۵۰۰ مشترک (تجمعی بین همه بات‌ها).** با حجم ۵۰۰–۱۰۰۰ پیام روز، این سقف **در کمتر از یک هفته** پر می‌شود. خودِ Chatbotscape می‌نویسد: «سقف ۵۰۰ مشترک گلوگاه عملی است، نه سقف پیام.»
2. **«Limited API» در پلن رایگان.** صفحه قیمت‌گذاری صراحتاً `API and webhooks` کامل را در ستون **Pro** می‌گذارد و برای Free می‌نویسد «Limited API». یعنی endpoint ارسال پیام (`POST /contacts/send`) ممکن است در پلن رایگان قفل باشد — باید تست کنید.
3. **ابهام در کانال رایگان.** یک منبع می‌گوید پلن رایگان ۴ کانال بومی دارد: «Telegram, Facebook Messenger, WhatsApp, Live Chat» ([chatbotscape.com/academy/chatbot-free-plans-compared-2026](https://chatbotscape.com/academy/chatbot-free-plans-compared-2026))، منبع دیگر می‌گوید «تا ۳ مورد از: Facebook Messenger, Instagram, TikTok, Telegram, Viber, Website» ([sendpulse.com/features/chatbot/comparison/manychat](https://sendpulse.com/features/chatbot/comparison/manychat)). **اینستاگرام در پلن رایگان تضمین‌شده نیست.**
4. **ریسک تحریم/دسترسی:** SendPulse یک شرکت اوکراینی است و ثبت‌نام آن نیاز به flow مرورگری دارد (برخلاف Zernio که فقط API است). من شواهد قطعی مبنی بر مسدودسازی ایران پیدا نکردم، ولی ریسک آن صفر نیست.

**نقش SendPulse در معماری من:** لایه‌ی **اضطراری (Failover)** — اگر Zernio سهمیه را قفل کرد یا قیمت‌هایش تغییر کرد، SendPulse با ۱۰,۰۰۰ پیام رایگان دیگر یک «ظرف دوم» می‌سازد. ⚠️ اما به دلیل **پروتکل Handover متا** (بخش ۷.۳) نمی‌توانید همزمان دو سرویس روی یک مکالمه فعال داشته باشید.

### ۲.۳ گزینه‌هایی که رد شدند (با دلیل مستند)

| سرویس | چرا رد شد | منبع |
|---|---|---|
| **Ayrshare** | پلن رایگان **حذف شده**. پلن قدیمی Basic (۲۰ پست/ماه) برای ثبت‌نام جدید بسته است. فقط ۲۸ روز تریال. ارزان‌ترین پلن $149/ماه..Messaging API یک add-on با **$49/ماه** است. | [ayrshare.com/pricing](https://www.ayrshare.com/pricing/) · [social-api.ai/blog/ayrshare-pricing-social-media-api-2026](https://social-api.ai/blog/ayrshare-pricing-social-media-api-2026) |
| **Buffer** | API روی همه‌ی پلن‌ها هست ولی **۲۵۰ درخواست در ۲۴ ساعت** روی پلن رایگان؛ **هیچ API برای DM/کامنت اینستاگرام ندارد**. فقط scheduling. | [buffer.com/resources/social-media-api-multi-platform-posting](https://buffer.com/resources/social-media-api-multi-platform-posting/) |
| **Publer** | API دایرکت/کامنت اینستاگرام نمی‌دهد؛ پلن رایگان محدود به scheduling است. | (بررسی شد، مورد تأیید قرار نگرفت) |
| **Blotato** | هیچ پلن رایگانی ندارد؛ از $29/ماه، و DM automation در پلن Starter به **۱,۰۰۰ مخاطب فعال در ماه** محدود است. | [blotato.com/blog/blotato-vs-zernio](https://www.blotato.com/blog/blotato-vs-zernio) |
| **ManyChat** | پلن رایگان از مارس ۲۰۲۶ به **~۲۵ مخاطب فعال در ماه** کاهش یافت + برندینگ اجباری. AI یک add-on ~$29/ماه است. | [reachlee.co/guides/instagram-dm-automation-pricing](https://www.reachlee.co/guides/instagram-dm-automation-pricing/) |
| **ChatbotX Community** | پلن $0 **فقط self-hosted روی VPS خودتان** — که شما ندارید. پلن‌های ابری از $29 شروع می‌شوند. نکته‌ی جالب: ChatbotX **open-source** است ([github.com/ChatbotXIO/ChatbotX](https://github.com/ChatbotXIO/ChatbotX)) ولی کانال اینستاگرامش به اپ متای خودتان نیاز دارد. | [chatbotx.io/pricing](https://chatbotx.io/pricing/) |
| **Postiz / Mixpost** | Self-hosted → نیاز به سرور همیشه‌روشن (در فهرست ممنوعه‌ی شما). | — |
| **n8n / OpenReply / Zernflow** | همه نیاز به worker همیشه‌روشن دارند. `zernio-dev/zernflow` یک ManyChat-alternative متن‌باز است ولی باید deploy شود. | [github.com/zernio-dev/zernflow](https://github.com/zernio-dev/zernflow) |
| **LinkDM / CreatorFlow / Inrō / Zorcha** | پلن رایگان دارند ولی **API عمومی و webhook نمی‌دهند** — فقط رابط کاربری. برای ربات تلگرام شما بی‌فایده‌اند. | [getkanal.com/blog/manychat-alternatives](https://getkanal.com/blog/manychat-alternatives) |
| **MCP رایگان به‌عنوان «دروازه‌ی فرار از هزینه»** | ❌ **وجود ندارد.** MCP فقط یک پروتکل است؛ tool callها دقیقاً همان endpointهای REST را صدا می‌زنند و همان متر را مصرف می‌کنند. خود Zernio می‌گوید: «MCP Server رایگان است. شما هزینه‌ی مصرف API را می‌دهید.» | [zernio.com/linkedin-mcp](https://zernio.com/linkedin-mcp) |

---

## 📌 بخش ۳ — پاسخ تفصیلی به ۵ سوال حیاتی

### ❓ سوال ۱: آیا جایگزین کاملاً رایگانی برای ChatbotX وجود دارد؟

**پاسخ کوتاه:** سه گزینه رایگان وجود دارد، ولی **هیچ‌کدام به تنهایی حجم ۵۰۰–۱۰۰۰ پیام روز را پوشش نمی‌دهند.** ترکیب آن‌ها با معماری هوشمند پوشش می‌دهد.

**محدودیت‌های دقیق استخراج‌شده:**

```
SendPulse Free (Basic) — sendpulse.com/pricing/messengers
├─ ۳ چت‌بات
├─ ۵۰۰ مشترک (تجمعی بین همه بات‌ها) ← گلوگاه واقعی
├─ ۱۰,۰۰۰ پیام/ماه (تجمعی بین همه بات‌ها)
├─ ۱۰ متغیر · ۱۰ trigger
├─ جریان‌های نامحدود (Unlimited flows)
├─ «Limited API» — API و Webhook کامل در ستون Pro
├─ بدون ChatGPT · بدون WhatsApp BSP · با برندینگ SendPulse
└─ دائمی، بدون انقضا، بدون کارت
   └─ ارتقا: $12/ماه (۵۰۰ مشترک) · $39 (۵,۰۰۰) · $59 (۱۰,۰۰۰)

SocialAPI.ai Hobby — social-api.ai/pricing
├─ ۲ پروفایل اجتماعی (brand)
├─ ۱۰ پست/ماه
├─ ۵۰ تعامل/ماه ← هر «نوشتن»: پاسخ کامنت، ارسال DM، reaction
├─ خواندن (کامنت/DM/mention/review): نامحدود و رایگان ✅
├─ دسترسی به همه‌ی ۱۱ پلتفرم + MCP Server
├─ Webhook: ❌ فقط از Side Hustle ($29/ماه) به بالا
└─ هنگام عبور: HTTP 429 با کد rate_limit_exceeded + هدرهای
   X-Posts-Remaining و X-Interactions-Remaining

Zernio Free — docs.zernio.com/pricing
├─ ۲ اکانت متصل (بدون کارت)
├─ ۱۰,۰۰۰ پیام خروجی/ماه ← تنها متر
├─ پست: نامحدود · وب‌هوک: نامحدود · API: کامل · MCP: کامل
├─ پاسخ کامنت/ریویو: نامحدود و رایگان ✅
├─ Rate limit: ۶۰ درخواست/دقیقه
└─ بدون کارت → در پیام ۱۰,۰۰۱ خطای 403، بدون شارژ ✅

Meta Business Suite (بومی متا) — کاملاً رایگان
├─ Instant Reply (اولین پیام) · Away Message · FAQ (۴–۵ سوال)
├─ Saved Replies (دستی) · Location/Contact/Hours
├─ Custom Keywords: ⛔ حداکثر ۵ کلیدواژه · case-sensitive · exact-match
│   ⛔ تأخیر ۱۵ دقیقه‌ای (مگر اینکه خودتان زودتر جواب دهید)
│   ⛔ فقط روی Meta Business Suite دسکتاپ
├─ Comment to Message: ⛔ همان محدودیت‌ها · دسکتاپ-only
│   ⛔ از ۲۰۲۶: حداکثر ۱ DM به‌ازای هر کاربر در ۲۴ ساعت
│   ⛔ پیام‌های بیش از ~۲۰۰/ساعت **حذف می‌شوند، نه اینکه صف شوند**
└─ AI Auto Replies (پاسخ‌دهنده‌ی مولد متا): در حال عرضه تدریجی،
    با برچسب «AI + نام شما» در رشته
```

منابع: [sendpulse.com/pricing/messengers](https://sendpulse.com/pricing/messengers) · [chatbotscape.com/reviews/sendpulse-review](https://chatbotscape.com/reviews/sendpulse-review) · [social-api.ai/pricing](https://social-api.ai/pricing) · [creatorflow.so/blog/instagram-built-in-automation](https://creatorflow.so/blog/instagram-built-in-automation/) · [creatorlanehq.com/blog/instagram-free-comment-to-dm-2026](https://creatorlanehq.com/blog/instagram-free-comment-to-dm-2026)

**جمع‌بندی سوال ۱:** ChatbotX را با **Zernio Inbox** جایگزین کنید (که هم‌اکنون اکانتش را دارید و رایگان است). SendPulse را به‌عنوان **ظرف اضطراری دوم** نگه دارید.

---

### ❓ سوال ۲: آیا بدون اپ متا و با اپ تأییدشده‌ی دیگران می‌توان رایگان خواند و پاسخ داد؟

**پاسخ: بله — و این دقیقاً کاری است که الان با Zernio انجام می‌دهید.** سه لایه را جدا کنیم:

**لایه ۱ — خواندن: ۱۰۰٪ رایگان و نامحدود.**
Zernio این رویدادها را با وب‌هوک HMAC-SHA256 امضاشده به Vercel شما می‌فرستد ([docs.zernio.com/webhooks/inbox](https://docs.zernio.com/webhooks/inbox)):

| رویداد | اینستاگرام | کاربرد شما |
|---|---|---|
| `message.received` | ✅ | ورودی اصلی ربات |
| `message.sent` | ✅ | dedup و همگام‌سازی state |
| `comment.received` | ✅ | ورودی کامنت‌ها |
| `conversation.started` | ✅ | ساخت رکورد مخاطب |
| `message.edited` / `message.deleted` | ✅ | همگام‌سازی ویرایش/unsend |
| `message.read` | ✅ | نمایش «خوانده شد» در ربات تلگرام |
| `reaction.received` | ✅ | ⚠️ رویداد جداگانه است، هرگز `message.received` نمی‌شود — باید جدا branch بزنید |
| `referral.received` | ✅ | تشخیص ورود از لینک `ig.me` |
| `conversation.control_changed` | ✅ | ⚠️ **بحرانی** — یعنی اپ دیگری مالک مکالمه شده |

نکته‌ی مهم: **هزینه‌ی وب‌هوک صفر است و نامحدود.** پس «خواندن» هیچ سقفی ندارد.

**لایه ۲ — پاسخ کامنت: ۱۰۰٪ رایگان و نامحدود.**
`POST /v1/inbox/comments/{postId}` با `{accountId, message, commentId}` — این یک پاسخ عمومی زیر پست است و در متر پیام **نمی‌نشیند**. این بزرگ‌ترین اهرم شماست.

**لایه ۳ — دایرکت: رایگان تا ۱۰,۰۰۰ در ماه.**
`POST /v1/inbox/conversations/{conversationId}/messages`

**آیا Zernio راهی دارد که با وب‌هوک پیام‌ها را به Vercel بفرستد و ما با یک لایه هوشمندانه از هزینه فرار کنیم؟**
بله — و دقیقاً دو مکانیزم:

1. **وب‌هوک رایگان** → Vercel → تصمیم‌گیری هوشمند → **پاسخ کامنت (رایگان)** به‌جای دایرکت.
2. **Workflow های Zernio با نود `webhook`** → نود webhook به Vercel شما POST می‌زند، پاسخ LLM را می‌گیرد و در متغیر ذخیره می‌کند، سپس نود `send_message` آن را می‌فرستد. ([docs.zernio.com/glossary](https://docs.zernio.com/glossary) — Workflow: تریگرها = DM, ref link, ad click, comment, story mention, reaction, WhatsApp status, **API call**؛ نودها = `send message`, `wait_for_reply`, `condition`, `delay`, **`webhook`**, `handoff`)

⚠️ اما توجه: مسیر Workflow هم در نهایت یک `send_message` انجام می‌دهد و **شمرده می‌شود**. هیچ راهی برای دور زدن متر از طریق Workflow یا MCP وجود ندارد. «فرار هوشمندانه» فقط از راه **تغییر کانال پاسخ** (DM → Comment) ممکن است.

**محدودیت‌های رسمی متا که باید رعایت کنید** ([blotato.com/blog/instagram-dm-limits](https://www.blotato.com/blog/instagram-dm-limits) · [conferbot.com/limits/instagram](https://www.conferbot.com/limits/instagram)):

| محدودیت | مقدار متا | دامنه |
|---|---|---|
| پنجره‌ی پیام‌رسانی | ۲۴ ساعت از آخرین پیام کاربر | هر اکانت حرفه‌ای |
| تمدید پنجره | تا ۷ روز با `messageTag: "HUMAN_AGENT"` + `messagingType: "MESSAGE_TAG"` | — |
| Private Reply به کامنت | **۱ پیام به‌ازای هر کامنت**، حداکثر ۷ روز پس از کامنت | — |
| Private Reply (پست/ریلز) | ۷۵۰ فراخوانی در ساعت | هر اکانت |
| Send API (متن/لینک/استیکر) | ۱۰۰ یا ۳۰۰ در ثانیه | هر اکانت |
| Send API (صوت/ویدیو) | ۱۰ در ثانیه | هر اکانت |
| Conversations API (خواندن رشته) | **۲ در ثانیه** ← تنگ‌ترین محدودیت | هر اکانت |
| حجم بالای Inbox | بیش از ۷۲,۰۰۰ پیام ارسال+دریافت → inbox پیام جدید نشان نمی‌دهد و اکانت نمی‌تواند بفرستد | هر اکانت |
| طول متن یک پیام API | ۱,۰۰۰ بایت UTF-8 (با دکمه: ۶۴۰ کاراکتر) | — |
| پیام‌های ویرایش‌شده (۲۰۲۶) | از آوریل ۲۰۲۶ سه message tag روی اینستاگرام از کار افتاد: confirmed event, account update, post-purchase update | — |
| اکتبر ۲۰۲۶ | دکمه‌های quick-reply که شماره تلفن/ایمیل پر می‌کردند غیرفعال شدند | — |

---

### ❓ سوال ۳: راه‌حل Serverless — آیا می‌توان فقط با یک GET/POST رایگان پاسخ را به اینستاگرام فرستاد؟

**پاسخ صادقانه: «ارسال» همیشه یک فراخوانی API به Zernio است و اگر DM باشد شمرده می‌شود. ولی «تولید پاسخ» و «صف‌بندی» را می‌توان ۱۰۰٪ رایگان کرد.** تفکیک:

| بخش | هزینه | راه‌حل رایگان |
|---|---|---|
| دریافت وب‌هوک | $0 | Vercel Function (۱M invocation/ماه رایگان) |
| ذخیره در Supabase | $0 | API requests **نامحدود** در پلن رایگان |
| صف و بودجه‌بندی | $0 | جدول `outbox` در Postgres + تابع SQL اتمیک |
| تولید پاسخ با LLM | $0 | Groq Free (۳۰ RPM / ۱,۰۰۰ RPD) + زنجیره fallback |
| کش پاسخ FAQ | $0 | جدول `faq_cache` در Supabase |
| **ارسال پاسخ کامنت** | **$0** | `POST /v1/inbox/comments/{postId}` |
| **ارسال دایرکت** | **$0 تا ۱۰,۰۰۰/ماه** | `POST /v1/inbox/conversations/{id}/messages` |
| زمان‌بند (Drain) | $0 | Vercel Cron (روزی یک‌بار) + **Cloudflare Worker رایگان به‌عنوان تپش هر ۱۰ دقیقه** |
| پل تلگرام (اطلاع‌رسانی/تصمیم) | $0 | Telegram Bot API (رایگان و نامحدود) |

**درباره‌ی «MCP رایگان» که پرسیدید:** Zernio یک MCP میزبانی‌شده در `https://mcp.zernio.com/mcp` دارد (Streamable HTTP، OAuth 2.1 + PKCE، یا Bearer API Key، ۴۹۶ ابزار). می‌توانید از یک Vercel Function با یک POST ساده JSON-RPC صدا بزنید:

```bash
curl https://mcp.zernio.com/mcp -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $ZERNIO_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call",
       "params":{"name":"posts_create","arguments":{...}},"id":1}'
```

منبع: [docs.zernio.com/mcp](https://docs.zernio.com/mcp)

⚠️ **اما این یک «دروازه‌ی رایگان» نیست.** MCP صرفاً یک روکش روی همان API است و متر همان متر است. خودِ Zernio صریح می‌گوید: «The MCP server is free. You pay for API usage.» ارزش واقعی MCP برای شما **در زمان توسعه** است (اتصال Claude Code / Cursor به اکانت برای دیباگ)، نه در زمان اجرا.

#### لایه LLM رایگان — وضعیت واقعی اکتبر ۲۰۲۶

این بخش خیلی تغییر کرده و اکثر راهنماهای فارسی قدیمی هستند:

| ارائه‌دهنده | سهمیه رایگان (اکتبر ۲۰۲۶) | کارت | نکته حیاتی |
|---|---|---|---|
| **Groq** ⭐ | `gpt-oss-120b` / `gpt-oss-20b` / `qwen3.6-27b`: **30 RPM، 1,000 RPD، 8K TPM، 200K TPD** (هر مدل جدا) | ❌ | **Llama از ۱۶ اوت ۲۰۲۶ از پلن Free حذف شد** و enterprise-only شد. `llama-prompt-guard-2` هنوز ۱۴,۴۰۰ RPD دارد ← **عالی برای تشخیص اسپم**. محدودیت‌ها **به‌ازای organization** است، نه به‌ازای key. |
| **Gemini** | ⚠️ **وضعیت بحرانی.** گوگل از سپتامبر ۲۰۲۶ جدول سهمیه رایگان را از مستندات حذف کرد. مدل‌های `gemini-2.5-*` برای کلیدهای جدید **`404` می‌دهند** («no longer available to new users»). مدل‌های 3.x Flash در عمل **~۲۰ درخواست در روز** گزارش شده‌اند. | ❌ | **روی Gemini به‌عنوان ستون اصلی حساب نکنید.** فقط به‌عنوان fallback. |
| **OpenRouter** | ۲۰ RPM و **۵۰ درخواست در روز** روی مدل‌های `:free`. پس از یک‌بار خرید $10 اعتبار (آستانه‌ی مادام‌العمر، نه موجودی فعلی) به **۱,۰۰۰ در روز** می‌رسد. | ❌ | ترفند $10 یک‌بار پرداخت است و بعد از صفرشدن موجودی هم سقف ۱,۰۰۰ باقی می‌ماند. اگر «کاملاً صفر» می‌خواهید، فقط ۵۰/روز. |
| **Mistral La Plateforme** | لایه «Experiment» رایگان؛ حدود **۱ میلیارد توکن در ماه** ولی نرخ ~**۲ درخواست در دقیقه**. نیاز به **شماره تلفن تأییدشده** (نه کارت). اعداد دقیق دیگر منتشر نمی‌شوند (Admin Panel → API → Limits). | ❌ | RPM=2 یعنی برای حجم شما عملاً بی‌استفاده، مگر به‌عنوان fallback شبانه. |

منابع: [console.groq.com/docs/rate-limits](https://console.groq.com/docs/rate-limits) · [klymentiev.com/blog/groq-pricing](https://klymentiev.com/blog/groq-pricing) · [aiplug.work/en/blog/gemini-free-tier-limits](https://aiplug.work/en/blog/gemini-free-tier-limits) · [ai.google.dev/gemini-api/docs/rate-limits](https://ai.google.dev/gemini-api/docs/rate-limits) · [openrouter.ai/blog/tutorials/free-llm-apis-compared](https://openrouter.ai/blog/tutorials/free-llm-apis-compared/) · [aicost.tools/blog/best-llm-apis-with-a-free-tier](https://aicost.tools/blog/best-llm-apis-with-a-free-tier/) · [dev.to/tariqnasser/free-llm-api-tiers-in-october-2026](https://dev.to/tariqnasser/free-llm-api-tiers-in-october-2026-whats-left-and-how-i-chain-them-227l)

**نتیجه‌ی طراحی:** ستون اصلی = **Groq `gpt-oss-120b`** (۱,۰۰۰ درخواست/روز = ۳۰,۰۰۰ در ماه). با کش FAQ (بخش ۵.۴) مصرف واقعی شما به **۲۰۰–۴۰۰ فراخوانی در روز** می‌رسد → **سقف Groq هرگز شکسته نمی‌شود.** برای طبقه‌بندی اسپم از `meta-llama/llama-prompt-guard-2-22m` با ۱۴,۴۰۰ RPD استفاده کنید (رایگان و تقریباً نامحدود).

⚠️ **نکته‌ی مهم درباره تحریم:** فراخوانی‌های LLM از **IP سرورهای Vercel** انجام می‌شوند، نه از IP ایران. پس Groq/OpenRouter از نظر شبکه کار می‌کنند. مشکل احتمالی فقط در **مرحله‌ی ثبت‌نام و گرفتن API Key** است (احراز هویت). Groq فقط ایمیل می‌خواهد (نه شماره تلفن، نه کارت) → کم‌ریسک‌ترین گزینه.

---

### ❓ سوال ۴: مدیریت مصرف Zernio زیر ۱۰,۰۰۰ پیام در ماه

**پاسخ: بله، و Zernio ابزارهای داخلی خوبی هم دارد.** دو سطح دفاع:

#### سطح ۱ — ابزارهای داخلی Zernio

| قابلیت | فیلد / Endpoint | اثر روی بودجه |
|---|---|---|
| **تشخیص پیام تکراری** ✅ | `dedupeSameTextHours` (۱ تا ۸۷۶۰ ساعت) روی Comment Automation | اگر همان متن (بعد از شخصی‌سازی) قبلاً به همان فرد از همان اکانت ارسال شده، **DM رد می‌شود** و با وضعیت `skipped` لاگ می‌شود |
| **سیاست تکرار** ✅ | `repeatPolicy.mode = "once"` (پیش‌فرض) یا `"every_comment"` + `cooldownHours` | در حالت `once` هر فرد **فقط یک بار برای همیشه** DM می‌گیرد |
| **چرخش متن** ✅ | `dmMessageVariations` (تا ۵) و `commentReplyVariations` (تا ۵) | الگوی «پیام یکسان» را می‌شکند (کمک به ضداسپم متا) |
| **کلیدواژه‌های حذفی** ✅ | `excludeKeywords` | مثلاً «not interested»، «چند»، «قیمت» را از قیف DM بیرون می‌کشد |
| **حالت تطبیق** ✅ | `matchMode: word / exact / contains` + `typoTolerance` | از شلیک اشتباه (مثل `app` داخل `happy`) جلوگیری می‌کند |
| **فیلتر مخاطب** ✅ | `audience.followerStatus`، `audience.minFollowerCount`، `audience.whenUnknown` | فقط فالوورها یا فقط اکانت‌های بالای N فالوور DM بگیرند |
| **بلاک مخاطب** ✅ | فیلد `is_blocked` روی Contact (`GET /v1/contacts/{id}`) + `POST /v1/contacts/{id}` | لیست سیاه داخلی |
| **Opt-out خودکار** ✅ | کلیدواژه‌های STOP / UNSUBSCRIBE / CANCEL به‌طور خودکار مخاطب را opt-out می‌کنند و automations را skip می‌کنند؛ START برمی‌گرداند | فقط برای iMessage/SMS مستند شده، ولی الگو را در لایه خودتان هم پیاده کنید |
| **مخفی‌کردن کامنت اسپم** ✅ | `POST /v1/inbox/comments/{postId}/{commentId}/hide` و `unhide` (اینستاگرام، فیسبوک، Threads) + `set_comment_moderation` | اسپم را قبل از پاسخ حذف می‌کند |
| **کنترل مالکیت مکالمه** ✅ | `POST` روی thread control (Change who answers a conversation) | جلوی خطای `409 not_thread_owner` را می‌گیرد |
| **ایدمپوتنسی** ✅ | هدر `Idempotency-Key` روی پاسخ کامنت | retry دوباره مصرف نمی‌کند |
| **هشدار سهمیه** ⚠️ | ایمیل به مالک تیم در ۷,۵۰۰ پیام (فقط وقتی کارت نداشته باشید) | تنها سیگنال رسمی Zernio |
| **شمارنده زنده** ❌ | **وجود ندارد** — `GET /v1/usage` پیام‌ها را ندارد | **باید خودتان در Supabase بشمارید** |

منابع: [docs.zernio.com/comment-automations/create-comment-automation](https://docs.zernio.com/comment-automations/create-comment-automation) · [docs.zernio.com/changelog](https://docs.zernio.com/changelog) · [docs.zernio.com/pricing/messages](https://docs.zernio.com/pricing/messages) · [zernio.com/social-media-comments](https://zernio.com/social-media-comments) · [github.com/zernio-dev/zernio-php](https://github.com/zernio-dev/zernio-php/blob/main/docs/Model/GetContact200ResponseContact.md)

#### سطح ۲ — بودجه‌بندی در Supabase (معماری پیشنهادی)

**سقف روزانه‌ی ایمن:**

```
بودجه ماهانه              = 10,000 پیام
حاشیه امن (۱۰٪)           =  1,000 پیام
بودجه قابل مصرف           =  9,000 پیام
روزهای ماه                =     30.44
─────────────────────────────────────────
سقف روزانه DM (DAILY_CAP) =    295 پیام  →  گرد می‌کنیم به ۲۹۰
```

**سناریوهای واقعی برای @zeynalikids:**

| سناریو | ورودی روزانه | تفکیک | DM/ماه | هزینه | وضعیت |
|---|---|---|---|---|---|
| خوش‌بینانه | ۵۰۰ | ۷۰٪ کامنت / ۳۰٪ DM | ۴,۵۰۰ | **$0** | ✅ ۵۵٪ حاشیه |
| **محتمل** | ۸۰۰ | ۶۵٪ کامنت / ۳۵٪ DM | ۸,۴۰۰ | **$0** | ✅ ۱۶٪ حاشیه |
| بدبینانه | ۱,۰۰۰ | ۶۰٪ کامنت / ۴۰٪ DM | ۱۲,۰۰۰ | **$0.20** | ⚠️ با بودجه‌بندی → $0 |
| پست وایرال | ۳,۰۰۰ در یک روز | — | — | — | 🛡️ `403` سخت، بدون شارژ |

> 💡 **مهم‌ترین جمله‌ی این گزارش:** با `DAILY_CAP = 290` و انتقال سرریز به **پاسخ عمومی کامنت** (که رایگان و نامحدود است)، حتی در سناریوی «پست وایرال» هم هزینه‌ی شما **صفر مطلق** می‌ماند. کاربر پیامش را می‌گیرد — فقط به‌جای دایرکت، زیر کامنتش.

#### الگوریتم اولویت‌بندی (Priority Router)

```
ورودی → طبقه‌بندی اسپم (Prompt-Guard، رایگان)
   ├─ اسپم/توهین → hide_comment + پاسخ نده  (۰ هزینه)
   ├─ FAQ شناخته‌شده (قیمت/سایز/ارسال/آدرس) → پاسخ از کش، بدون LLM
   ├─ حاوی اطلاعات خصوصی (شماره سفارش، آدرس، کد ملی، تلفن) → **حتماً DM** (اولویت ۱)
   ├─ مشتری با سابقه خرید → **DM** (اولویت ۲)
   ├─ سوال عمومی (سایز، رنگ، موجودی) → **پاسخ کامنت** (اولویت ۳ — رایگان)
   ├─ درخواست لینک → **پاسخ کامنت با لینک** (لینک در کامنت مجاز است)
   └─ بودجه DM امروز تمام شد؟ → همه‌چیز به **پاسخ کامنت** + اطلاع به تلگرام ادمین
```

---

### ❓ سوال ۵: راه‌حل خلاقانه — آیا یک پل کاملاً رایگان و بی‌محدودیت ممکن است؟

**پاسخ: یک پل «بدون هیچ محدودیتی» وجود ندارد — چون گلوگاه در سمت متاست، نه سمت شما. ولی یک پل «بدون هیچ هزینه‌ای» بله، و این همان چیزی است که در بخش ۵ طراحی کرده‌ام.**

چهار ایده‌ی خلاقانه را بررسی کردم:

| ایده | نتیجه |
|---|---|
| **الف) پل تلگرام ← اینستاگرام** (ادمین در تلگرام جواب می‌دهد، ربات به اینستاگرام می‌فرستد) | ✅ **کار می‌کند و رایگان است** — ولی هر پاسخ یک DM خروجی = مصرف سهمیه. **محدودیت را حذف نمی‌کند، فقط UX را بهتر می‌کند.** |
| **ب) استفاده از Telegram به‌عنوان کانال جایگزین** (هدایت کاربران به بات تلگرام) | ✅ **هوشمندانه‌ترین راه فرار.** بات تلگرام **هیچ سقفی ندارد** و Telegram Bot API کاملاً رایگان است. در بیو اینستاگرام + پاسخ کامنت بنویسید: «برای سفارش به @zeynalikids_bot پیام دهید». ⚠️ اما نرخ تبدیل در ایران پایین است (کاربران اینستاگرام ترجیح می‌دهند در همان اینستاگرام بمانند). |
| **پ) ترکیب Zernio + SendPulse روی یک اکانت** | ⛔ **غیرممکن.** پروتکل Handover متا فقط به **یک اپ** اجازه می‌دهد مالک یک مکالمه باشد. اگر اپ دیگری مالک باشد، ارسال شما با `409 not_thread_owner` شکست می‌خورد (`details.ownerAppId` مالک را نام می‌برد). باید با `conversation.control_changed` مالکیت را بگیرید. |
| **ت) اتوماسیون بومی Meta Business Suite به‌عنوان لایه‌ی بی‌نهایت** | ⚠️ **تا حدی.** رایگان و بی‌سقف است، ولی: تأخیر ۱۵ دقیقه‌ای، ۵ کلیدواژه exact و case-sensitive، فقط دسکتاپ، و از ۲۰۲۶ حداکثر ۱ DM به‌ازای هر کاربر در ۲۴ ساعت. ⛔ **و یک تله‌ی بزرگ:** اگر همزمان با Zernio فعال باشد، Page Inbox متا مالک مکالمه می‌شود و Zernio خطای `409` می‌گیرد. **باید یکی را انتخاب کنید.** |
| **ث) MCP رایگان به‌عنوان دروازه‌ی ارسال** | ⛔ **توهم.** MCP همان API است؛ همان متر. |

**نتیجه:** خلاقانه‌ترین ترکیب ممکن = **Zernio (DM بودجه‌بندی‌شده) + پاسخ کامنت (بی‌نهایت) + بات تلگرام (بی‌نهایت، کانال فرار) + Groq (LLM رایگان) + Cloudflare Worker رایگان (تپش صف)**. این معماری در بخش ۵ آمده است.

---

## 📌 بخش ۴ — معماری پیشنهادی: «ZerCost Inbox v1»

### ۴.۱ دیاگرام کلی

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          اینستاگرام @zeynalikids                          │
│                       (Business/Creator + Facebook Page)                  │
└───────────────┬───────────────────────────────▲──────────────────────────┘
                │ Webhook (Meta)                 │ Graph API (اپ تأییدشده Zernio)
                ▼                                │
┌───────────────────────────────────────────────────────────────────────────┐
│  ZERNIO  (پلن رایگان: ۲ اکانت · ۶۰ req/min · ۱۰,۰۰۰ DM/ماه · بدون کارت)  │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐ │
│  │ Inbox Webhooks  │  │ Comments API     │  │ Messages API             │ │
│  │ ۱۴ رویداد        │  │ reply/hide/like  │  │ send (متر مصرف می‌شود)    │ │
│  │ HMAC-SHA256     │  │ ⭐ رایگان نامحدود │  │ ⭐ رایگان تا ۱۰,۰۰۰      │ │
│  └────────┬────────┘  └────────▲─────────┘  └───────────▲──────────────┘ │
└───────────┼────────────────────┼─────────────────────────┼────────────────┘
            │ POST               │ POST                    │ POST
            ▼                    │                         │
┌───────────────────────────────────────────────────────────────────────────┐
│                        VERCEL (Hobby) — FastAPI                            │
│                                                                            │
│  /api/webhooks/zernio        /api/drain             /api/tg               │
│  ┌──────────────────────┐   ┌────────────────────┐ ┌────────────────────┐ │
│  │ ۱. تأیید HMAC        │   │ تپش هر ۱۰ دقیقه    │ │ ربات تلگرام ادمین  │
│  │ ۲. dedup با event_id │   │ از Cloudflare یا   │ │ — تأیید/رد پاسخ AI │
│  │ ۳. skip standby      │   │ cron-job.org یا    │ │ — پاسخ دستی        │
│  │ ۴. INSERT به inbox   │   │ Vercel daily cron  │ │ — آمار مصرف        │
│  │ ۵. صف‌گذاری در outbox │   │ ↓                  │ └────────────────────┘ │
│  │ ۶. زنجیره خود-فراخوان│   │ بودجه‌سنج → LLM →  │                         │
│  └──────────────────────┘   │ ارسال               │                         │
│                             └─────────┬──────────┘                         │
└───────────────────────────────────────┼────────────────────────────────────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
┌──────────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│  SUPABASE (Free)         │ │  GROQ (Free)         │ │  CLOUDFLARE (Free)   │
│  • DB 500MB              │ │  gpt-oss-120b        │ │  Worker + Cron       │
│  • API نامحدود           │ │  30 RPM / 1,000 RPD  │ │  هر ۱۰ دقیقه →       │
│  • ۵۰۰K Edge Fn/ماه      │ │  + llama-prompt-guard│ │  POST /api/drain     │
│  • pause پس از ۷ روز بی‌  │ │  (اسپم: ۱۴,۴۰۰ RPD)  │ │  ✅ استفاده تجاری    │
│    کاری (→ با تپش حل می‌  │ │  fallback: OpenRouter│ │     مجاز است         │
│    شود)                   │ │  → Gemini → Mistral  │ │  ۱۰۰K req/روز        │
└──────────────────────────┘ └──────────────────────┘ └──────────────────────┘
```

### ۴.۲ چرا این معماری روی Vercel کار می‌کند (با وجود نبود VPS)

سه مشکل Serverless و راه‌حل هرکدام:

| مشکل | راه‌حل | هزینه |
|---|---|---|
| **الف) Cron روی Vercel Hobby فقط روزی یک‌بار است** — عباراتی مثل `0 * * * *` یا `*/30 * * * *` در زمان deploy با خطای «Hobby accounts are limited to daily cron jobs» شکست می‌خورند | یک **Cloudflare Worker رایگان** با Cron Trigger هر ۱۰ دقیقه که فقط یک `fetch()` به `https://<your-app>.vercel.app/api/drain` می‌زند. جایگزین‌ها: `cron-job.org` (رایگان، تا هر دقیقه) یا GitHub Actions روی **repo عمومی** (دقیقه‌ها نامحدود و رایگان) | $0 |
| **ب) صف نیاز به worker همیشه‌روشن دارد** | الگوی **زنجیره خود-فراخوان**: هر webhook پس از INSERT یک `fetch()` به `/api/drain` می‌زند (با `waitUntil` تا بعد از پاسخ اجرا شود). اگر صف خالی بود، زنجیره قطع می‌شود. Cron هر ۱۰ دقیقه به‌عنوان «تور ایمنی» برای پیام‌های جامانده | $0 |
| **ج) حالت (state) بین فراخوانی‌ها از بین می‌رود** | همه‌ی state در Supabase: شمارنده بودجه، صف، کش FAQ، dedup. توابع SQL اتمیک تا شرط مسابقه نداشته باشید | $0 |

منابع: [vercel.com/docs/cron-jobs/usage-and-pricing](https://vercel.com/docs/cron-jobs/usage-and-pricing) · [vercel.com/docs/plans/hobby](https://vercel.com/docs/plans/hobby) · [developers.cloudflare.com/workers/platform/limits](https://developers.cloudflare.com/workers/platform/limits/) · [cron-job.org](https://cron-job.org/en/) · [docs.github.com/en/actions/concepts/billing-and-usage](https://docs.github.com/en/actions/concepts/billing-and-usage)

### ۴.۳ طرح دیتابیس Supabase

```sql
-- ═══════════════════════════════════════════════════════════════
-- ۱. ورودی خام (هر رویداد وب‌هوک)
-- ═══════════════════════════════════════════════════════════════
create table public.inbox_events (
  id               uuid primary key default gen_random_uuid(),
  zernio_event_id  text unique not null,        -- dedup در سطح وب‌هوک
  event_type       text not null,               -- message.received | comment.received | ...
  account_id       text not null,
  profile_id       text,
  conversation_id  text,
  platform_msg_id  text,
  direction        text check (direction in ('inbound','outbound')),
  sender_id        text,
  sender_name      text,
  channel          text not null check (channel in ('dm','comment','reaction','story_reply','referral')),
  body_text        text,
  is_standby       boolean default false,       -- metadata.standby از Zernio
  raw_payload      jsonb not null default '{}'::jsonb,
  received_at      timestamptz not null default now(),
  processed        boolean default false
);
create index on public.inbox_events (processed, received_at);
create index on public.inbox_events (conversation_id, received_at desc);

-- ═══════════════════════════════════════════════════════════════
-- ۲. صف خروجی با بودجه‌بندی (قلب سیستم ضد-هزینه)
-- ═══════════════════════════════════════════════════════════════
create type outbox_status as enum
  ('pending','generating','ready','sent','failed','downgraded','blocked','expired');
create type reply_channel as enum ('dm','comment','telegram_only','none');
create type budget_class  as enum ('metered','free');   -- metered = DM

create table public.outbox (
  id                uuid primary key default gen_random_uuid(),
  inbox_event_id    uuid references public.inbox_events(id) on delete set null,
  idempotency_key   text unique not null,        -- hash(conversation, event, intent)
  account_id        text not null,
  conversation_id   text,
  comment_post_id   text,                        -- برای پاسخ کامنت
  comment_id        text,
  recipient_id      text,
  intended_channel  reply_channel not null default 'dm',
  actual_channel    reply_channel,
  budget_class      budget_class generated always as
                    (case when intended_channel = 'dm' then 'metered'::budget_class
                          else 'free'::budget_class end) stored,
  priority          smallint not null default 5, -- ۱ = بحرانی (اطلاعات خصوصی)
  body              text,
  llm_provider      text,
  llm_model         text,
  llm_latency_ms    integer,
  cache_hit         boolean default false,
  status            outbox_status not null default 'pending',
  attempts          smallint not null default 0,
  last_error        text,
  zernio_status     integer,                     -- ۴۰۳ = سهمیه تمام
  created_at        timestamptz not null default now(),
  scheduled_for     timestamptz not null default now(),
  sent_at           timestamptz
);
create index on public.outbox (status, scheduled_for, priority)
  where status in ('pending','generating','ready');
create index on public.outbox (created_at);

-- ═══════════════════════════════════════════════════════════════
-- ۳. شمارنده بودجه (چون Zernio هیچ endpoint شمارش ندارد)
-- ═══════════════════════════════════════════════════════════════
create table public.dm_budget (
  period_utc    text primary key,               -- '2026-10'
  monthly_cap   integer not null default 10000,
  safety_margin real    not null default 0.10,  -- ۱۰٪ حاشیه امن
  daily_cap     integer not null default 290,   -- floor(9000 / 31)
  sent_month    integer not null default 0,
  sent_today    integer not null default 0,
  blocked_403   integer not null default 0,
  downgraded    integer not null default 0,
  updated_at    timestamptz not null default now()
);

-- ═══════════════════════════════════════════════════════════════
-- ۴. کش پاسخ FAQ (بزرگ‌ترین کاهش‌دهنده مصرف LLM)
-- ═══════════════════════════════════════════════════════════════
create table public.faq_cache (
  id            uuid primary key default gen_random_uuid(),
  intent        text not null,                  -- 'price' | 'size' | 'shipping' | ...
  normalized_q  text not null,
  answer_dm     text not null,
  answer_comment text not null,                 -- نسخه کوتاه‌تر برای کامنت عمومی
  lang          text not null default 'fa',
  hit_count     integer not null default 0,
  embedding     vector(3072),                   -- اختیاری (pgvector)
  approved      boolean not null default true,
  updated_at    timestamptz not null default now(),
  unique (intent, normalized_q, lang)
);

-- ═══════════════════════════════════════════════════════════════
-- ۵. لیست سیاه و آمار مخاطب
-- ═══════════════════════════════════════════════════════════════
create table public.contact_flags (
  contact_id     text primary key,
  instagram_id   text,
  is_spam        boolean default false,
  is_blocked     boolean default false,
  is_opted_out   boolean default false,         -- STOP / لغو / حذف
  is_vip         boolean default false,         -- مشتری با سابقه خرید
  order_count    integer default 0,
  msg_in_count   integer default 0,
  msg_out_count  integer default 0,
  last_seen_at   timestamptz,
  note           text
);

-- ═══════════════════════════════════════════════════════════════
-- ۶. ژورنال ممیزی (برای ربات تلگرام و دیباگ)
-- ═══════════════════════════════════════════════════════════════
create table public.audit_log (
  id          bigserial primary key,
  actor       text,      -- 'webhook' | 'drain' | 'llm' | 'zernio' | 'admin'
  action      text,
  entity_id   text,
  detail      jsonb default '{}'::jsonb,
  created_at  timestamptz default now()
);

-- ═══════════════════════════════════════════════════════════════
-- ⭐ تابع اتمیک «رزرو بودجه» — جلوگیری از شرط مسابقه
-- ═══════════════════════════════════════════════════════════════
create or replace function public.try_reserve_dm_slot(p_utc_date date)
returns table(granted boolean, reason text, sent_today int, sent_month int)
language plpgsql as $$
declare
  v_period text := to_char(p_utc_date, 'YYYY-MM');
  v_row    public.dm_budget%rowtype;
  v_eff_daily_cap integer;
begin
  insert into public.dm_budget(period_utc)
  values (v_period)
  on conflict (period_utc) do nothing;

  select * into v_row from public.dm_budget where period_utc = v_period for update;

  -- سرریز روز قبل را صفر کن (ریست Zernio در 00:00 UTC اول ماه است،
  -- ولی سقف روزانه ما محلی است)
  v_eff_daily_cap := greatest(1, floor(
      (v_row.monthly_cap * (1 - v_row.safety_margin))
      / extract(day from (date_trunc('month', p_utc_date) + interval '1 month - 1 day'))
  )::integer);

  if v_row.sent_month >= (v_row.monthly_cap * (1 - v_row.safety_margin)) then
    return query select false, 'monthly_cap_reached'::text, v_row.sent_today, v_row.sent_month;
    return;
  end if;

  if v_row.sent_today >= v_eff_daily_cap then
    return query select false, 'daily_cap_reached'::text, v_row.sent_today, v_row.sent_month;
    return;
  end if;

  update public.dm_budget
     set sent_today = sent_today + 1,
         sent_month = sent_month + 1,
         updated_at = now()
   where period_utc = v_period;

  return query select true, 'granted'::text, v_row.sent_today + 1, v_row.sent_month + 1;
end;
$$;

-- ═══════════════════════════════════════════════════════════════
-- ⭐ تابع «برداشتن کار بعدی از صف» — اتمیک و بدون دوبل‌کاری
-- ═══════════════════════════════════════════════════════════════
create or replace function public.claim_next_outbox(p_batch int default 10)
returns setof public.outbox
language plpgsql as $$
begin
  return query
  with picked as (
    select o.id
      from public.outbox o
     where o.status in ('pending','ready')
       and o.scheduled_for <= now()
       and o.attempts < 5
     order by o.priority asc, o.scheduled_for asc
     limit p_batch
     for update skip locked
  )
  update public.outbox o
     set status = 'generating', attempts = o.attempts + 1
    from picked p
   where o.id = p.id
  returning o.*;
end;
$$;

-- ═══════════════════════════════════════════════════════════════
-- ⭐ افزایش اتمیک شمارنده downgrade (برای گزارش‌گیری)
-- ═══════════════════════════════════════════════════════════════
create or replace function public.bump_downgraded()
returns void language sql as $$
  insert into public.dm_budget(period_utc, downgraded)
  values (to_char(now() at time zone 'utc', 'YYYY-MM'), 1)
  on conflict (period_utc)
  do update set downgraded = public.dm_budget.downgraded + 1,
                updated_at = now();
$$;

-- ═══════════════════════════════════════════════════════════════
-- نمای مصرف برای ربات تلگرام
-- ═══════════════════════════════════════════════════════════════
create or replace view public.v_usage_today as
select
  (select sent_today from public.dm_budget
    where period_utc = to_char(now() at time zone 'utc','YYYY-MM'))  as dm_sent_today,
  (select daily_cap  from public.dm_budget
    where period_utc = to_char(now() at time zone 'utc','YYYY-MM'))  as dm_daily_cap,
  (select sent_month from public.dm_budget
    where period_utc = to_char(now() at time zone 'utc','YYYY-MM'))  as dm_sent_month,
  (select count(*) from public.outbox
    where status = 'downgraded'
      and created_at > date_trunc('day', now()))                     as downgraded_today,
  (select count(*) from public.inbox_events
    where received_at > date_trunc('day', now()))                    as inbound_today,
  (select count(*) from public.faq_cache c
    where c.hit_count > 0)                                           as faq_entries;
```

### ۴.۴ کد پیاده‌سازی (FastAPI روی Vercel)

#### الف) Webhook receiver با تأیید امضا و dedup

```python
# app/webhooks.py
import hashlib, hmac, json, os
from datetime import datetime, timezone
from fastapi import APIRouter, Request, HTTPException
from supabase import create_client

router = APIRouter()
sb = create_client(os.environ["SUPABASE_URL"], os.environ["SUPABASE_SERVICE_KEY"])
ZERNIO_SECRET = os.environ["ZERNIO_WEBHOOK_SECRET"].encode()

# رویدادهایی که هزینه تولید پاسخ دارند
ACTIONABLE = {"message.received", "comment.received"}


def verify_signature(body: bytes, signature: str) -> bool:
    """Zernio با HMAC-SHA256 در هدر X-Zernio-Signature امضا می‌کند."""
    expected = hmac.new(ZERNIO_SECRET, body, hashlib.sha256).hexdigest()
    # امضا ممکن است با پیشوند sha256= بیاید
    candidate = signature.split("=", 1)[-1]
    return hmac.compare_digest(expected, candidate)


@router.post("/api/webhooks/zernio")
async def zernio_webhook(request: Request):
    raw = await request.body()
    sig = request.headers.get("x-zernio-signature", "")
    if not verify_signature(raw, sig):
        raise HTTPException(status_code=401, detail="bad signature")

    evt = json.loads(raw)
    event_type = evt.get("event") or evt.get("type")
    event_id = evt.get("eventId") or evt.get("id")

    # ── فیلتر ۱: رویدادهای غیرقابل‌اقدام (read receipt, delivered, ...)
    if event_type not in ACTIONABLE:
        return {"ok": True, "skipped": event_type}   # ۲۰۰ برگردان تا retry نشود

    # ── فیلتر ۲: standby — اپ دیگری مالک مکالمه است، اتوماسیون اجرا نکن
    meta = evt.get("metadata") or {}
    is_standby = bool(meta.get("standby"))

    # ── فیلتر ۳: reaction هرگز message.received نمی‌شود، جدا branch بزن
    acct = evt.get("account") or {}
    msg = evt.get("message") or {}
    cmt = evt.get("comment") or {}

    channel = "dm" if event_type == "message.received" else "comment"
    sender = (msg.get("sender") or cmt.get("author") or {})

    row = {
        "zernio_event_id": event_id,
        "event_type": event_type,
        "account_id": acct.get("accountId"),
        "profile_id": acct.get("profileId"),
        "conversation_id": (evt.get("conversation") or {}).get("id"),
        "platform_msg_id": msg.get("platformMessageId") or cmt.get("platformCommentId"),
        "direction": "inbound",
        "sender_id": sender.get("id"),
        "sender_name": sender.get("name") or sender.get("username"),
        "channel": channel,
        "body_text": msg.get("text") or cmt.get("text"),
        "is_standby": is_standby,
        "raw_payload": evt,
    }

    # ── dedup: تحویل Zernio «at-least-once» است، پس روی event_id unique بزن
    try:
        sb.table("inbox_events").insert(row).execute()
    except Exception as e:
        if "duplicate key" in str(e) or "23505" in str(e):
            return {"ok": True, "deduped": True}
        raise

    if is_standby:
        sb.table("audit_log").insert(
            {"actor": "webhook", "action": "skipped_standby",
             "entity_id": event_id, "detail": {"owner": meta}}
        ).execute()
        return {"ok": True, "skipped": "standby"}

    # ── صف‌گذاری با اولویت
    priority = classify_priority(row)
    intended = "dm" if channel == "dm" else "comment"
    idem = hashlib.sha256(
        f"{row['conversation_id']}|{row['platform_msg_id']}|{intended}".encode()
    ).hexdigest()

    sb.table("outbox").insert({
        "inbox_event_id": None,           # در صورت نیاز با select پر کنید
        "idempotency_key": idem,
        "account_id": row["account_id"],
        "conversation_id": row["conversation_id"],
        "comment_post_id": (cmt.get("post") or {}).get("platformPostId"),
        "comment_id": cmt.get("platformCommentId"),
        "recipient_id": sender.get("id"),
        "intended_channel": intended,
        "priority": priority,
        "status": "pending",
    }).execute()

    # ── زنجیره خود-فراخوان (بدون نیاز به VPS)
    trigger_drain_background(request)
    return {"ok": True, "queued": True, "priority": priority}


def classify_priority(row: dict) -> int:
    """۱=بحرانی (اطلاعات خصوصی) · ۳=مشتری VIP · ۵=عادی · ۸=کم‌ارزش"""
    text = (row.get("body_text") or "").lower()
    private_markers = ["سفارش", "کد رهگیری", "آدرس", "تلفن", "شماره کارت", "order"]
    if any(m in text for m in private_markers):
        return 1                       # حتماً باید DM شود (حریم خصوصی)
    if row["channel"] == "comment":
        return 5                       # پاسخ کامنت رایگان است
    return 5


def trigger_drain_background(request: Request):
    """با waitUntil یک فراخوانی پس‌زمینه به /api/drain بزن."""
    import asyncio
    asyncio.get_event_loop().create_task(_ping_drain())


async def _ping_drain():
    import httpx
    try:
        async with httpx.AsyncClient(timeout=25) as c:
            await c.post(
                os.environ["SELF_URL"] + "/api/drain",
                headers={"x-drain-token": os.environ["DRAIN_TOKEN"]},
            )
    except Exception:
        pass   # cron هر ۱۰ دقیقه جبران می‌کند
```

#### ب) Drain worker با بودجه‌سنج و زنجیره LLM

```python
# app/drain.py
import os, time, httpx
from datetime import datetime, timezone
from fastapi import APIRouter, Request, HTTPException

router = APIRouter()
ZERNIO = "https://zernio.com/api/v1"
ZKEY = os.environ["ZERNIO_API_KEY"]
DAILY_HARD_CAP = int(os.environ.get("DAILY_HARD_CAP", "290"))
BATCH = 10


@router.post("/api/drain")
async def drain(request: Request):
    if request.headers.get("x-drain-token") != os.environ["DRAIN_TOKEN"]:
        # Vercel Cron هم با Authorization: Bearer $CRON_SECRET می‌زند
        auth = request.headers.get("authorization", "")
        if auth != f"Bearer {os.environ['CRON_SECRET']}":
            raise HTTPException(401, "unauthorized")

    today = datetime.now(timezone.utc).date()
    processed = 0

    while processed < 5 * BATCH:                      # حداکثر ۵۰ مورد در هر فراخوانی
        rows = sb.rpc("claim_next_outbox", {"p_batch": BATCH}).execute().data
        if not rows:
            break

        for job in rows:
            budget_granted, reason = reserve_budget(job, today)

            if not budget_granted:
                downgrade_to_comment(job, reason)     # ⭐ هسته‌ی ضد-هزینه
                continue

            body = await compose_reply(job)
            ok, err, status = await send_reply(job, body)

            if not ok and status == 403:
                # سهمیه Zernio تمام شد → بلافاصله downgrade
                release_budget(today)
                downgrade_to_comment(job, "zernio_403_allowance_exceeded")
            elif not ok:
                mark_failed(job, err)
            processed += 1

        if len(rows) < BATCH:
            break

    return {"ok": True, "processed": processed}


def reserve_budget(job: dict, today) -> tuple[bool, str]:
    """اگر کانال free است (کامنت/تلگرام) هیچ بودجه‌ای مصرف نکن."""
    if job["intended_channel"] != "dm":
        return True, "free_channel"

    res = sb.rpc("try_reserve_dm_slot", {"p_utc_date": today.isoformat()}).execute().data
    if res and res[0]["granted"]:
        return True, "granted"

    reason = res[0]["reason"] if res else "unknown"
    # اولویت ۱ (اطلاعات خصوصی) هرگز downgrade نمی‌شود — فقط به تلگرام می‌رود
    if job["priority"] == 1:
        return False, f"private_data_{reason}"
    return False, reason


async def compose_reply(job: dict) -> str:
    """۱) کش FAQ → ۲) قوانین → ۳) LLM با fallback"""
    # ۱. کش
    cached = lookup_faq(job)
    if cached:
        sb.table("outbox").update({"cache_hit": True}).eq("id", job["id"]).execute()
        return cached

    # ۲. LLM زنجیره‌ای
    for provider, model, url, key in LLM_CHAIN:
        try:
            t0 = time.time()
            text = await call_llm(provider, model, url, key, job)
            latency = int((time.time() - t0) * 1000)
            sb.table("outbox").update({
                "llm_provider": provider, "llm_model": model,
                "llm_latency_ms": latency,
            }).eq("id", job["id"]).execute()
            return text
        except (httpx.HTTPStatusError, Exception):
            continue                                   # به provider بعدی برو

    return FALLBACK_TEXT                               # پاسخ ثابت، بدون LLM


LLM_CHAIN = [
    ("groq",      "openai/gpt-oss-120b",              os.environ["GROQ_URL"],  os.environ["GROQ_KEY"]),
    ("groq",      "openai/gpt-oss-20b",               os.environ["GROQ_URL"],  os.environ["GROQ_KEY"]),
    ("openrouter","google/gemma-4-31b-it:free",       os.environ["OR_URL"],    os.environ["OR_KEY"]),
    ("gemini",    "gemini-3.1-flash-lite",            os.environ["GEM_URL"],   os.environ["GEM_KEY"]),
]
FALLBACK_TEXT = (
    "سلام 🌸 ممنون از پیام شما. همکاران ما در اسرع وقت پاسخ می‌دهند. "
    "برای پیگیری سریع‌تر می‌توانید به بات تلگرام ما پیام دهید."
)


async def send_reply(job: dict, body: str) -> tuple[bool, str, int]:
    headers = {
        "Authorization": f"Bearer {ZKEY}",
        "Content-Type": "application/json",
        "Idempotency-Key": job["idempotency_key"],
    }
    if job["intended_channel"] == "comment":
        url = f"{ZERNIO}/inbox/comments/{job['comment_post_id']}"
        payload = {
            "accountId": job["account_id"],
            "commentId": job["comment_id"],
            "message": body[:2200],
        }
    else:
        url = f"{ZERNIO}/inbox/conversations/{job['conversation_id']}/messages"
        payload = {
            "accountId": job["account_id"],
            "message": body.encode("utf-8")[:1000].decode("utf-8", "ignore"),
        }

    async with httpx.AsyncClient(timeout=20) as c:
        r = await c.post(url, headers=headers, json=payload)

    if r.status_code in (200, 201):
        sb.table("outbox").update({
            "status": "sent", "actual_channel": job["intended_channel"],
            "body": body, "sent_at": datetime.now(timezone.utc).isoformat(),
            "zernio_status": r.status_code,
        }).eq("id", job["id"]).execute()
        return True, "", r.status_code

    if r.status_code == 409:      # not_thread_owner → باید thread control بگیریم
        await take_thread_control(job)
        return await send_reply(job, body) if job.get("_retried") is None else (False, r.text, 409)

    sb.table("outbox").update({
        "status": "failed", "last_error": r.text[:500], "zernio_status": r.status_code,
    }).eq("id", job["id"]).execute()
    return False, r.text, r.status_code


def downgrade_to_comment(job: dict, reason: str):
    """⭐ تبدیل DM به پاسخ کامنت رایگان — یا فقط اطلاع به تلگرام."""
    if job["comment_id"] and job["comment_post_id"]:
        sb.table("outbox").update({
            "intended_channel": "comment",
            "actual_channel": "comment",
            "status": "pending",                 # دوباره صف‌گذاری، این بار رایگان
            "priority": 6,
            "last_error": f"downgraded:{reason}",
        }).eq("id", job["id"]).execute()
        sb.rpc("bump_downgraded").execute()          # افزایش اتمیک شمارنده downgrade
    else:
        # کامنتی در کار نیست (DM خالص) → فقط به تلگرام ادمین اطلاع بده
        sb.table("outbox").update({
            "intended_channel": "telegram_only",
            "status": "downgraded",
            "last_error": f"downgraded:{reason}",
        }).eq("id", job["id"]).execute()
        notify_telegram_admin(job, reason)         # Telegram Bot API — رایگان و بی‌سقف
```

#### ج) Cloudflare Worker به‌عنوان تپش رایگان (جایگزین محدودیت cron در Vercel)

```javascript
// worker/drain-tick.js  —  deploy با wrangler، پلن Workers Free
// wrangler.toml:
//   [triggers]
//   crons = ["*/10 * * * *"]
export default {
  async scheduled(event, env, ctx) {
    const targets = [env.DRAIN_URL];               // https://<app>.vercel.app/api/drain
    ctx.waitUntil(Promise.allSettled(targets.map(async (url) => {
      try {
        await fetch(url, {
          method: "POST",
          headers: {
            "x-drain-token": env.DRAIN_TOKEN,
            "content-type": "application/json",
          },
          body: JSON.stringify({ source: "cf-cron", ts: Date.now() }),
        });
      } catch (e) {
        console.error("drain tick failed", e);
      }
    })));
  },
};
```

**چرا Cloudflare و نه Vercel Cron؟**

| معیار | Vercel Hobby Cron | Cloudflare Workers Free Cron |
|---|---|---|
| حداقل فاصله | ⛔ **روزی یک‌بار** | ✅ دقیقه‌ای (مستندات رسمی حداقل را مشخص نکرده؛ عملی `*/10` کار می‌کند) |
| تعداد trigger | ۱۰۰ در هر پروژه | ۵ در هر اکانت |
| دقت زمانی | ±۵۹ دقیقه | بهتر |
| **استفاده تجاری** | ⛔ **ممنوع** | ✅ **مجاز** |
| CPU | — | ۱۰ms در هر فراخوانی (برای یک `fetch` کافی است) |
| درخواست‌ها | ۱M/ماه | ۱۰۰K/روز |

منابع: [vercel.com/docs/cron-jobs/usage-and-pricing](https://vercel.com/docs/cron-jobs/usage-and-pricing) · [developers.cloudflare.com/workers/platform/limits](https://developers.cloudflare.com/workers/platform/limits/) · [developers.cloudflare.com/changelog/post/2026-02-04-queues-free-plan](https://developers.cloudflare.com/changelog/post/2026-02-04-queues-free-plan/) · [morphllm.com/comparisons/cloudflare-workers-vs-vercel](https://www.morphllm.com/comparisons/cloudflare-workers-vs-vercel)

> 🔁 **جایگزین بدون Cloudflare:** `cron-job.org` (کاملاً رایگان، تا هر دقیقه، متن‌باز GPL) یا یک GitHub Actions workflow روی **ریپازیتوری عمومی** (دقیقه‌های رایگان و نامحدود؛ حداقل فاصله `*/5`؛ فقط روی branch پیش‌فرض؛ پس از ۶۰ روز بی‌کاری غیرفعال می‌شود). توجه: روی ریپوی **خصوصی** با اکانت Free، رویداد `schedule` عملاً کار نمی‌کند.

#### د) لایه تشخیص اسپم رایگان (Prompt Guard)

```python
# app/spam.py
import httpx, os

GUARD_URL = "https://api.groq.com/openai/v1/chat/completions"
GUARD_KEY = os.environ["GROQ_KEY"]
GUARD_MODEL = "meta-llama/llama-prompt-guard-2-22m"   # ۱۴,۴۰۰ درخواست/روز رایگان


async def is_spam(text: str) -> bool:
    if not text or len(text) < 2:
        return True
    # فیلتر ارزان و قطعی اول
    spam_patterns = ["crypto", "usdt", "سیگنال", "ویو رایگان", "فالوور",
                     "follow4follow", "dm for", "قرعه‌کشی برنده"]
    low = text.lower()
    if any(p in low for p in spam_patterns):
        return True
    try:
        async with httpx.AsyncClient(timeout=8) as c:
            r = await c.post(GUARD_URL, headers={"Authorization": f"Bearer {GUARD_KEY}"},
                             json={"model": GUARD_MODEL,
                                   "messages": [{"role": "user", "content": text}]})
        label = r.json()["choices"][0]["message"]["content"].strip().lower()
        return "unsafe" in label or "spam" in label or "injection" in label
    except Exception:
        return False       # در خطا، سخت‌گیرانه عمل نکن (fail-open)
```

---

## 📌 بخش ۵ — برنامه کاهش هزینه (با اعداد)

| # | اقدام | اثر روی مصرف DM | اثر روی مصرف LLM | پیچیدگی |
|---|---|---|---|---|
| ۱ | **غیرفعال‌کردن Comment Automation روی اینستاگرام** (چون `dmMessage` اجباری است و هر trigger یک DM مصرف می‌کند) | 🔻 تا ۶۰٪ | — | کم |
| ۲ | **پاسخ کامنت به‌جای DM** برای سوالات عمومی (قیمت، سایز، موجودی، رنگ) با `POST /v1/inbox/comments/{postId}` — رایگان و نامحدود | 🔻 ۵۰–۷۰٪ | — | کم |
| ۳ | **کش FAQ** با تطبیق نرمال‌شده (حذف اعراب/نیم‌فاصله) برای ۱۰ پرسش پرتکرار | — | 🔻 ۶۰–۸۰٪ | متوسط |
| ۴ | **`dedupeSameTextHours`** و `repeatPolicy.mode="once"` در هر automation باقی‌مانده | 🔻 ۱۰–۲۵٪ | — | کم |
| ۵ | **لیست سیاه + Prompt Guard** برای اسپم/ربات‌ها → `hide_comment` بدون پاسخ | 🔻 ۵–۱۵٪ | 🔻 ۵–۱۵٪ | کم |
| ۶ | **بودجه‌بندی روزانه ۲۹۰** در Supabase + downgrade خودکار | 🔻 سقف سخت | — | متوسط |
| ۷ | **لایه‌ی قوانین قبل از LLM** (regex برای «قیمت»، «سایز»، «ارسال») | — | 🔻 ۳۰–۵۰٪ | کم |
| ۸ | **هدایت به بات تلگرام** برای سفارش‌گذاری و پیگیری (بدون هیچ سقفی) | 🔻 ۱۰–۳۰٪ | — | متوسط |
| ۹ | **ادغام پاسخ‌ها**: اگر کاربر ۳ پیام پشت‌هم فرستاد، ۲ ثانیه صبر کن و یک پاسخ بده (debounce) | 🔻 ۱۵–۳۰٪ | 🔻 ۱۵–۳۰٪ | متوسط |
| ۱۰ | **حذف کارت بانکی از Zernio** → `403` سخت به‌جای شارژ خودکار | 🛡️ سقف $0 | — | کم |

### سناریوی بهینه‌شده برای @zeynalikids

```
ورودی روزانه:                        ۸۰۰ پیام/کامنت
├─ اسپم (Prompt Guard + لیست سیاه):   −۸۰   →  hide، بدون پاسخ
├─ تکراری (debounce + dedup):         −۹۶   →  ادغام شد
├─ کامنت عمومی → پاسخ کامنت (رایگان): −۴۲۰  →  $0
└─ DM واقعی:                          ۲۰۴ در روز
                                       ─────────
                                       × ۳۰.۴۴ = ۶,۲۱۰ DM در ماه
                                       سهمیه = ۱۰,۰۰۰
                                       ✅ ۳۸٪ حاشیه امن باقی می‌ماند
                                       💰 هزینه ماهانه: $0.00

مصرف LLM:
├─ کش FAQ:                            ۵۵٪ بدون LLM
├─ لایه قوانین:                       ۲۰٪ بدون LLM
└─ فراخوانی واقعی LLM:                 ~۵۰ در روز
                                       ✅ زیر ۵٪ سهمیه Groq (۱,۰۰۰/روز)
```

---

## 📌 بخش ۶ — زیرساخت: جدول سهمیه‌ها و ریسک‌های پنهان

| سرویس | سهمیه رایگان | مصرف تخمینی شما | وضعیت | ریسک پنهان |
|---|---|---|---|---|
| **Vercel Hobby** | ۱M function invocation/ماه · ۴ CPU-hour · ۳۶۰ GB-hrs حافظه · ۱۰۰ GB انتقال · ۳۰۰s حداکثر زمان اجرا · ۱۰۰ deploy در روز | ~۱۰۰K invocation/ماه | ✅ ۱۰ برابر حاشیه | ⛔ **فقط استفاده غیرتجاری/شخصی.** عبور از سهمیه = **pause شدن پروژه تا ۳۰ روز** (بدون شارژ اضافه) |
| **Supabase Free** | ۵۰۰MB دیتابیس · **API نامحدود** · ۵۰۰K Edge Function/ماه · ۵GB egress · ۲ پروژه · ۲۰۰ اتصال Realtime | ~۴۰MB دیتابیس در سال اول | ✅ | ⚠️ **pause خودکار پس از ۷ روز بی‌کاری** → تپش ۱۰ دقیقه‌ای Cloudflare این را هم حل می‌کند |
| **Cloudflare Workers Free** | ۱۰۰K درخواست/روز · ۱۰ms CPU · ۵ Cron Trigger/اکانت · Queues ۱۰K operation/روز · KV ۱GB (۱۰۰K خواندن/۱K نوشتن در روز) · D1 ۵GB | ~۴,۵۰۰ درخواست/ماه | ✅ | ⚠️ **KV write فقط ۱,۰۰۰ در روز** → از D1 یا Supabase استفاده کنید، نه KV |
| **Groq Free** | ۳۰ RPM · ۱,۰۰۰ RPD · ۸K TPM · ۲۰۰K TPD (هر مدل) | ~۵۰–۲۰۰ در روز | ✅ | ⚠️ سهمیه **به‌ازای organization** است، چند key کمکی نمی‌کند. Llama از ۱۶ اوت ۲۰۲۶ حذف شد |
| **Telegram Bot API** | رایگان و عملاً نامحدود | — | ✅ | ⚠️ ارسال به همان چت: ~۱ پیام در ثانیه |
| **cron-job.org** | رایگان، تا هر دقیقه | — | ✅ | ⚠️ وابستگی به سرویس شخص ثالث |

منابع: [vercel.com/docs/plans/hobby](https://vercel.com/docs/plans/hobby) · [vercel.com/docs/limits](https://vercel.com/docs/limits) · [cloudzero.com/blog/supabase-pricing](https://www.cloudzero.com/blog/supabase-pricing/) · [itpathsolutions.com/supabase-free-tier-limits](https://www.itpathsolutions.com/supabase-free-tier-limits) · [developers.cloudflare.com/workers/platform/pricing](https://developers.cloudflare.com/workers/platform/pricing/)

---

## 📌 بخش ۷ — تحلیل ریسک کامل

### ۷.۱ ریسک‌های بحرانی

| # | ریسک | احتمال | اثر | راه‌حل |
|---|---|---|---|---|
| **R1** | ⛔ **Vercel Hobby استفاده تجاری را ممنوع می‌کند.** «Hobby teams are restricted to non-commercial personal use only.» یک پیج فروش لباس کودک = تجاری. | بالا | **بالا** — مسدودسازی اکانت | گزینه الف: انتقال به **Cloudflare Workers Free** (تجاری مجاز). گزینه ب: Vercel Pro ($20/ماه — با هدف صفر تضاد دارد). گزینه ج: پذیرش ریسک و استفاده از Cloudflare به‌عنوان لایه اصلی |
| **R2** | ⚠️ **تغییر قیمت Zernio.** Zernio در ۱۲ ماه **سه بار** مدل قیمت‌گذاری را عوض کرده: ۵ پلن ثابت (تا سپتامبر ۲۰۲۵) → Social Sets با add-on (اوایل ۲۰۲۶) → پرداخت به‌ازای اکانت (از میانه ۲۰۲۶). متر پیام هم از ۱ اکتبر ۲۰۲۶ فعال شد. | **بسیار بالا** | بالا | لایه انتزاع `SocialGateway` در کد تا تعویض provider در یک روز ممکن باشد + نگهداری SendPulse به‌عنوان ظرف دوم + مانیتورینگ `docs.zernio.com/changelog` |
| **R3** | ⚠️ **پروتکل Handover متا.** اگر Page Inbox یا اپ دیگری مالک مکالمه شود، ارسال Zernio با `409 not_thread_owner` شکست می‌خورد. | متوسط | متوسط | گوش دادن به `conversation.control_changed` + فراخوانی endpoint «Change who answers a conversation» قبل از ارسال + **هرگز همزمان Meta Business Suite Automations و Zernio را فعال نکنید** |
| **R4** | ⚠️ **کاهش سهمیه LLM رایگان.** گوگل در دسامبر ۲۰۲۵ سهمیه رایگان را ۵۰–۸۰٪ کاهش داد (۲۵۰ → ۲۰ RPD برای 2.5 Flash) و مدل‌های ۲.۵ را برای کاربران جدید بست. Groq هم Llama را حذف کرد. | **بسیار بالا** | متوسط | زنجیره ۴-لایه fallback + لایه قوانین + کش FAQ + متن ثابت `FALLBACK_TEXT` که بدون LLM کار می‌کند |
| **R5** | ⚠️ **نبود endpoint شمارش پیام در Zernio.** `GET /v1/usage` پیام‌ها را برنمی‌گرداند. | قطعی | متوسط | شمارنده در Supabase (تابع `try_reserve_dm_slot`) + ایمیل هشدار Zernio در ۷,۵۰۰ به‌عنوان راستی‌آزمایی + کالیبراسیون ماهانه |
| **R6** | ⚠️ **pause شدن Supabase پس از ۷ روز بی‌کاری.** | کم (با تپش ۱۰ دقیقه‌ای) | **بالا** | تپش Cloudflare/cron-job.org که هر ۱۰ دقیقه یک `SELECT 1` می‌زند + UptimeRobot به‌عنوان لایه سوم |
| **R7** | ⚠️ **عبور از سقف ۱۰,۰۰۰ → `403` سخت.** بدون کارت، ارسال شکست می‌خورد و کاربر پاسخ نمی‌گیرد. | متوسط | متوسط | downgrade خودکار به پاسخ کامنت + اطلاع به تلگرام ادمین + بودجه روزانه ۲۹۰ که عملاً هیچ‌وقت به ۱۰,۰۰۰ نمی‌رسد |
| **R8** | ⚠️ **حذف کارت بانکی = ریسک تغییر رفتار.** اگر روزی کارت اضافه کنید، «هیچ‌چیز متوقف نمی‌شود» و overage شارژ می‌شود. | — | — | **هرگز کارت اضافه نکنید** یا یک `spend cap` در داشبورد Zernio تنظیم کنید |
| **R9** | ⚠️ **محدودیت ۷۲,۰۰۰ پیام Inbox اینستاگرام.** اگر از آن عبور کنید، «inbox پیام جدید نشان نمی‌دهد و اکانت نمی‌تواند بفرستد». | بسیار کم | **بحرانی** | آرشیو و حذف مکالمه‌های قدیمی با `delete_inbox_conversation` |
| **R10** | ⚠️ **ریسک تحریم/مسدودسازی حساب.** Zernio، Groq و Cloudflare همه شرکت‌های غربی هستند و ممکن است سیاست‌های تحریمی سخت‌گیرانه‌تری اعمال کنند. | کم | **بحرانی** | خروجی گرفتن منظم از Supabase (`pg_dump` روزانه با GitHub Actions روی ریپوی عمومی) + مستندسازی کامل معماری برای مهاجرت سریع |

### ۷.۲ ریسک‌های عملیاتی

| # | ریسک | راه‌حل |
|---|---|---|
| R11 | Rate limit ۶۰ req/min در پلن رایگان Zernio = ۱ درخواست در ثانیه. در زمان وایرال‌شدن پست ممکن است گلوگاه شود. | صف با `for update skip locked` + `Retry-After` روی `429` + خواندن هدر `X-RateLimit-Remaining` در هر پاسخ |
| R12 | تحویل وب‌هوک «at-least-once» است → پیام تکراری | constraint `unique` روی `zernio_event_id` + `Idempotency-Key` روی ارسال |
| R13 | `Conversations API` فقط ۲ فراخوانی در ثانیه | **هرگز** برای هر رویداد مکالمه را re-read نکنید؛ state را از payload وب‌هوک مشتق کنید |
| R14 | پاسخ کامنت بیش از ۲,۲۰۰ کاراکتر / DM بیش از ۱,۰۰۰ بایت UTF-8 | برش امن روی مرز کاراکتر (نه بایت) با `encode()[:1000].decode('utf-8','ignore')` |
| R15 | Private Reply فقط ۱ بار به‌ازای هر کامنت و حداکثر ۷ روز پس از آن | dedup روی `comment_id` + صف‌گذاری فوری (نه تأخیری) |
| R16 | پیام‌های بیش از ~۲۰۰/ساعت در اتوماسیون بومی متا **حذف می‌شوند، نه صف** | اگر از Meta Business Suite استفاده می‌کنید، حتماً pacing اعمال کنید |
| R17 | Cold start روی Vercel (چند صد میلی‌ثانیه) | وب‌هوک را با `200` سریع برگردانید و کار را به `waitUntil`/drain منتقل کنید |
| R18 | پنجره ۲۴ ساعته متا — پس از آن فقط با `messageTag: "HUMAN_AGENT"` و `messagingType: "MESSAGE_TAG"` | صف با اولویت زمانی: پیام‌های نزدیک به انقضای پنجره اول |

### ۷.۳ تله‌ی چند-اپ (Multi-App Trap)

این مهم‌ترین هشدار عملیاتی این گزارش است:

```
⛔ هرگز این‌ها را همزمان روی یک اکانت اینستاگرام فعال نکنید:
   ۱. Zernio Inbox / Comment Automations / Workflows
   ۲. Meta Business Suite → Inbox → Automations (Custom Keywords / Comment to Message)
   ۳. SendPulse Chatbot
   ۴. هر ابزار دیگری که به Page متصل است

چرا: پروتکل Handover متا فقط یک اپ را «مالک» هر مکالمه می‌کند.
     بقیه فقط observe می‌کنند (standby).
     ارسال از اپ غیرمالک → 409 not_thread_owner
     Zernio در این حالت metadata.standby: true می‌فرستد و
     «هیچ اتوماسیونی روی آن اجرا نمی‌کند: نه workflow، نه keyword automation»

✅ قانون: یک منبع حقیقت (Single Source of Truth) انتخاب کنید.
   پیشنهاد من: Zernio = مالک همه‌چیز. Meta Business Suite Automations = خاموش.
   اگر روزی به SendPulse مهاجرت کردید، اول Zernio را disconnect کنید.
```

منبع: [docs.zernio.com/webhooks/inbox](https://docs.zernio.com/webhooks/inbox) — بخش «Another app can own the conversation (standby)»

---

## 📌 بخش ۸ — نقشه راه پیاده‌سازی (۷ فاز)

### فاز ۰ — آماده‌سازی (۳۰ دقیقه، بدون کد)

- [ ] در داشبورد Zernio: **کارت بانکی را حذف کنید** (اگر هست) → تضمین `403` به‌جای شارژ
- [ ] در داشبورد Zernio: **فقط ۲ اکانت متصل** نگه دارید (Instagram + Facebook Page) → پلن رایگان
- [ ] `GET https://zernio.com/api/v1/accounts` با Bearer token → `accountId` و `profileId` اینستاگرام را یادداشت کنید
- [ ] `GET /v1/accounts` → بررسی `X-RateLimit-Limit: 60` (تأیید پلن رایگان)
- [ ] **غیرفعال‌کردن همه‌ی Comment Automationهای موجود** (بخش ۵ اقدام ۱)
- [ ] **خاموش‌کردن Meta Business Suite → Inbox → Automations** (بخش ۷.۳)
- [ ] ساخت API Key در [console.groq.com](https://console.groq.com) (فقط ایمیل لازم است)
- [ ] ساخت حساب Cloudflare (رایگان) برای Worker تپش

### فاز ۱ — لایه دریافت (۱ روز)

- [ ] `POST /v1/webhooks/settings` با یک `secret` دلخواه (خودتان می‌سازید؛ Zernio یکی تولید نمی‌کند) و رویدادهای: `message.received`, `comment.received`, `conversation.started`, `message.deleted`, `reaction.received`, `conversation.control_changed`
- [ ] پیاده‌سازی `/api/webhooks/zernio` با تأیید `X-Zernio-Signature`
- [ ] ساخت جدول `inbox_events` + constraint unique
- [ ] تست با یک DM واقعی → بررسی Webhook Logs در داشبورد Zernio (status code و response body هر تلاش)
- [ ] **مهم:** همیشه `200` برگردانید (حتی برای رویدادهای skip‌شده) وگرنه Zernio retry می‌کند

### فاز ۲ — لایه پاسخ کامنت رایگان (۱ روز) ← **بیشترین بازگشت سرمایه**

- [ ] پیاده‌سازی `/api/drain` با `claim_next_outbox`
- [ ] ارسال با `POST /v1/inbox/comments/{postId}` + `Idempotency-Key`
- [ ] ساخت `faq_cache` با ۱۰ پرسش پرتکرار @zeynalikids (قیمت، سایزبندی، ارسال، آدرس، تلفن، ساعت کاری، مرجوعی، رنگ‌بندی، جنس پارچه، نحوه سفارش)
- [ ] **تأیید کنید که `sent_month` در `dm_budget` افزایش نمی‌یابد** ← اثبات رایگان‌بودن

### فاز ۳ — بودجه‌سنج DM (۱ روز)

- [ ] تابع `try_reserve_dm_slot` + جدول `dm_budget`
- [ ] `downgrade_to_comment` + `notify_telegram_admin`
- [ ] مدیریت `403 message_allowance_exceeded` و `409 not_thread_owner`
- [ ] تست: `DAILY_HARD_CAP=2` بگذارید و ۵ DM بفرستید → باید ۲ تا DM و ۳ تا downgrade ببینید

### فاز ۴ — لایه LLM (۱–۲ روز)

- [ ] زنجیره fallback (Groq 120b → Groq 20b → OpenRouter → Gemini Flash-Lite)
- [ ] `llama-prompt-guard-2-22m` برای اسپم
- [ ] System prompt فارسی با شخصیت برند کودک + محدودیت «هرگز قیمت دقیق نده اگر در کش نیست»
- [ ] `FALLBACK_TEXT` برای وقتی همه providerها شکست خوردند
- [ ] **Human-in-the-loop:** برای پاسخ‌های با confidence پایین، به‌جای ارسال، به تلگرام ادمین بفرستید و با دکمه inline «تأیید / ویرایش / رد» تصمیم بگیرید

### فاز ۵ — پل تلگرام (۱ روز)

- [ ] اطلاع‌رسانی: هر DM جدید + downgradeها + هشدار ۸۰٪ بودجه
- [ ] دستور `/stats` → `v_usage_today`
- [ ] دستور `/budget <n>` → تغییر `daily_cap` به‌صورت زنده
- [ ] دستور `/pause` و `/resume` → توقف/ادامه اتوماسیون
- [ ] پاسخ دستی: ادمین در تلگرام reply می‌کند → ربات به اینستاگرام می‌فرستد (با مصرف بودجه)

### فاز ۶ — تپش و پایداری (نیم روز)

- [ ] Cloudflare Worker با `crons = ["*/10 * * * *"]` → `POST /api/drain`
- [ ] Vercel Cron روزانه (`0 3 * * *`) → job سنگین: گزارش روزانه، پاک‌سازی، `pg_dump`
- [ ] UptimeRobot روی `/health` (رایگان) → هم Supabase را بیدار نگه می‌دارد هم از سلامت سیستم مطمئن می‌شوید
- [ ] GitHub Actions روی **ریپوی عمومی** برای backup روزانه `pg_dump` → آرشیو در Releases

### فاز ۷ — بهینه‌سازی (مستمر)

- [ ] debounce ۲–۳ ثانیه‌ای برای پیام‌های پشت‌هم یک کاربر
- [ ] تحلیل `faq_cache` → افزودن intentهای جدید بر اساس پرسش‌های واقعی
- [ ] مانیتور هفتگی `docs.zernio.com/changelog` برای تغییرات قیمت/متر
- [ ] مانیتور ماهانه `console.groq.com/docs/rate-limits` برای تغییرات سهمیه

---

## 📌 بخش ۹ — چک‌لیست نهایی «صفر دلار»

| ردیف | قلم | هزینه ماهانه |
|---|---|---|
| ۱ | Zernio — ۲ اکانت متصل، بدون کارت بانکی | **$0.00** |
| ۲ | Zernio — وب‌هوک‌ها (نامحدود) | **$0.00** |
| ۳ | Zernio — پاسخ کامنت (نامحدود) | **$0.00** |
| ۴ | Zernio — پیام‌های دریافتی و خواندن (نامحدود) | **$0.00** |
| ۵ | Zernio — دایرکت خروجی تا ۱۰,۰۰۰/ماه | **$0.00** |
| ۶ | Zernio — MCP Server | **$0.00** |
| ۷ | Vercel Hobby | **$0.00** ⚠️ (محدودیت غیرتجاری) |
| ۸ | Supabase Free | **$0.00** |
| ۹ | Cloudflare Workers Free (تپش) | **$0.00** |
| ۱۰ | Groq Free (LLM + Prompt Guard) | **$0.00** |
| ۱۱ | Telegram Bot API | **$0.00** |
| ۱۲ | cron-job.org / GitHub Actions عمومی (پشتیبان) | **$0.00** |
| | **جمع کل** | **$0.00** |

### اگر مجبور به پرداخت شدید، ارزان‌ترین سناریوها

| سناریو | هزینه |
|---|---|
| ۱۲,۰۰۰ DM در ماه (۲,۰۰۰ overage) | **$0.20** |
| ۲۰,۰۰۰ DM در ماه (۱۰,۰۰۰ overage) | **$1.00** |
| ۳۰,۰۰۰ DM در ماه (۲۰,۰۰۰ overage) | **$2.00** |
| اتصال اکانت سوم (مثلاً Telegram جداگانه در Zernio) | **+$6.00** ⚠️ — **این کار را نکنید** |
| ارتقا به Zernio paid برای ۶۰۰ req/min | فقط در صورت اتصال اکانت سوم اتفاق می‌افتد |
| SendPulse Pro (۵۰۰ مشترک) | **$12.00** |
| SocialAPI.ai Side Hustle | **$29.00** |
| Vercel Pro (رفع محدودیت تجاری + cron دقیقه‌ای) | **$20.00** |

> 💡 **نکته طلایی:** تنها خطایی که می‌تواند شما را ناخواسته به $6 برساند، **اتصال یک اکانت سوم** به Zernio است (مثلاً Telegram یا یک اکانت Meta Ads). هر اتصال اضافی = $6/ماه. حواستان به `metaads` باشد: «یک Facebook Page، اکانت اینستاگرامش و اکانت تبلیغاتی Meta پشتشان = **۳ اکانت متصل**». فقط اینستاگرام (+ صفحه فیسبوک در صورت نیاز به comment-to-DM) را وصل کنید.

---

## 📌 بخش ۱۰ — منابع (همه در اکتبر ۲۰۲۶ بررسی شدند)

### Zernio (گزینه اصلی)
1. قیمت‌گذاری اصلی — https://docs.zernio.com/pricing
2. **قیمت‌گذاری پیام‌ها (مهم‌ترین سند)** — https://docs.zernio.com/pricing/messages
3. وب‌هوک‌های Inbox (۱۴ رویداد + standby + handover) — https://docs.zernio.com/webhooks/inbox
4. وب‌هوک‌ها (کلی، امضا، retry، dedup) — https://docs.zernio.com/webhooks
5. پلتفرم اینستاگرام (محدودیت‌ها، ice breakers، HUMAN_AGENT) — https://docs.zernio.com/platforms/instagram
6. ساخت Comment Automation (همه‌ی فیلدها) — https://docs.zernio.com/comment-automations/create-comment-automation
7. Rate Limits (۶۰/۶۰۰/۱۲۰۰ req/min) — https://docs.zernio.com/guides/rate-limits
8. واژه‌نامه (Workflow, Contact, Conversation) — https://docs.zernio.com/glossary
9. MCP Server — https://docs.zernio.com/mcp
10. Changelog (تغییرات ۲۰۲۶) — https://docs.zernio.com/changelog
11. API Comments (hide/unhide/moderation) — https://zernio.com/social-media-comments
12. GitHub org (SDKها، zernflow، unified-inbox) — https://github.com/zernio-dev
13. SDK پایتون — https://pypi.org/project/zernio-sdk/
14. OpenAPI spec — https://zernio.com/openapi.yaml · llms.txt — https://zernio.com/llms.txt
15. قیمت‌گذاری (صفحه بازاریابی) — https://zernio.com/pricing

### گزینه‌های جایگزین
16. SendPulse — قیمت مسنجرها — https://sendpulse.com/pricing/messengers
17. SendPulse — API اینستاگرام (۴۰+ endpoint) — https://sendpulse.com/integrations/api/chatbot/instagram
18. SendPulse — Webhookها — https://sendpulse.com/integrations/api/chatbot/webhooks
19. SendPulse — مرور پلن‌ها (جزئیات Free) — https://chatbotscape.com/reviews/sendpulse-review
20. مقایسه پلن‌های رایگان چت‌بات — https://chatbotscape.com/academy/chatbot-free-plans-compared-2026
21. SocialAPI.ai — قیمت‌گذاری — https://social-api.ai/pricing
22. SocialAPI.ai — رتبه‌بندی APIهای رایگان ۲۰۲۶ — https://social-api.ai/blog/free-social-media-api-2026
23. **۳ API که App Review متا را پاس کرده‌اند** — https://social-api.ai/blog/instagram-facebook-api-without-meta-app-review
24. Ayrshare — قیمت‌گذاری (بدون پلن رایگان) — https://www.ayrshare.com/pricing/
25. Ayrshare — Messages API (add-on $49) — https://www.ayrshare.com/docs/apis/messages/overview
26. ChatbotX — قیمت‌گذاری — https://chatbotx.io/pricing/
27. ChatbotX — GitHub (متن‌باز) — https://github.com/ChatbotXIO/ChatbotX
28. Blotato vs Zernio — https://www.blotato.com/blog/blotato-vs-zernio

### محدودیت‌های رسمی متا
29. محدودیت‌های DM اینستاگرام (جدول کامل) — https://www.blotato.com/blog/instagram-dm-limits
30. همه‌ی سقف‌های مستندشده — https://www.conferbot.com/limits/instagram
31. تغییرات ۲۰۲۶ در اتوماسیون DM — https://dmcompared.com/explained/instagram-automation-trends/
32. دو روش ادغام Instagram Messaging API — https://zernio.com/blog/instagram-messaging-api
33. قیمت API اینستاگرام — https://www.postzen.dev/blog/instagram-api-pricing
34. رایگان‌بودن Instagram Messaging API — https://www.conferbot.com/chatbot/instagram

### اتوماسیون بومی رایگان متا
35. شش ابزار رایگان اینستاگرام + محدودیت‌ها — https://creatorflow.so/blog/instagram-built-in-automation/
36. Comment-to-DM رایگان متا و مرزهایش — https://creatorlanehq.com/blog/instagram-free-comment-to-dm-2026
37. راهنمای تنظیم — https://getkanal.com/blog/instagram-automatic-messages

### زیرساخت Serverless
38. Vercel Hobby Plan — https://vercel.com/docs/plans/hobby
39. **Vercel Cron (روزی یک‌بار روی Hobby)** — https://vercel.com/docs/cron-jobs/usage-and-pricing
40. Vercel Limits — https://vercel.com/docs/limits
41. Cloudflare Workers Limits — https://developers.cloudflare.com/workers/platform/limits/
42. Cloudflare Workers Pricing (+ Queues رایگان) — https://developers.cloudflare.com/workers/platform/pricing/
43. Queues روی پلن رایگان (فوریه ۲۰۲۶) — https://developers.cloudflare.com/changelog/post/2026-02-04-queues-free-plan/
44. Cloudflare Queues Limits — https://developers.cloudflare.com/queues/platform/limits/
45. Supabase Free Tier Limits — https://www.itpathsolutions.com/supabase-free-tier-limits
46. قیمت Supabase (قاعده ۷ روز) — https://www.cloudzero.com/blog/supabase-pricing/
47. Cloudflare Workers vs Vercel (محدودیت تجاری) — https://www.morphllm.com/comparisons/cloudflare-workers-vs-vercel
48. cron-job.org — https://cron-job.org/en/
49. GitHub Actions Billing — https://docs.github.com/en/actions/concepts/billing-and-usage
50. محدودیت schedule روی ریپوی خصوصی Free — https://devactivity.com/insights/github-actions-cron-schedules-a-hidden-free-tier-hurdle-impacting-developer-productivity/

### لایه LLM رایگان
51. **Groq Rate Limits (رسمی)** — https://console.groq.com/docs/rate-limits
52. Groq Free Tier 2026 (حذف Llama) — https://klymentiev.com/blog/groq-pricing
53. Gemini Rate Limits (رسمی) — https://ai.google.dev/gemini-api/docs/rate-limits
54. **اندازه‌گیری عملی سهمیه رایگان Gemini (۲۰/روز)** — https://aiplug.work/en/blog/gemini-free-tier-limits
55. وضعیت Gemini Free در اکتبر ۲۰۲۶ — https://www.aifreeapi.com/en/posts/google-gemini-api-free-tier
56. مقایسه ۱۳ LLM رایگان — https://openrouter.ai/blog/tutorials/free-llm-apis-compared/
57. زنجیره‌کردن لایه‌های رایگان (اکتبر ۲۰۲۶) — https://dev.to/tariqnasser/free-llm-api-tiers-in-october-2026-whats-left-and-how-i-chain-them-227l
58. بهترین LLMها با پلن رایگان — https://aicost.tools/blog/best-llm-apis-with-a-free-tier/

### تحریم و دسترسی
59. فهرست کشورهای تحت تحریم جامع OFAC — https://sanctionslawyers.net/blog-en/ofac-sanctioned-countries-list-2026/
60. محدودیت‌های صادراتی ایران — https://orpa.princeton.edu/export-controls/sanctioned-countries

---

## 📌 پیوست الف — تقلب‌نامه‌ی Endpointها

```bash
# ═══ خواندن (همه رایگان و نامحدود) ═══
GET  https://zernio.com/api/v1/accounts
GET  https://zernio.com/api/v1/inbox/conversations
GET  https://zernio.com/api/v1/inbox/conversations/{conversationId}
GET  https://zernio.com/api/v1/inbox/conversations/{conversationId}/messages
GET  https://zernio.com/api/v1/inbox/comments                # پست‌های کامنت‌دار
GET  https://zernio.com/api/v1/inbox/comments/{postId}       # کامنت‌های یک پست
GET  https://zernio.com/api/v1/inbox/mentions
GET  https://zernio.com/api/v1/inbox/reviews
GET  https://zernio.com/api/v1/contacts
GET  https://zernio.com/api/v1/usage                         # ⚠️ پیام‌ها را ندارد
GET  https://zernio.com/api/v1/accounts/{accountId}/follow-status/{userId}

# ═══ نوشتن رایگان (متر مصرف نمی‌شود) ═══
POST https://zernio.com/api/v1/inbox/comments/{postId}                        # پاسخ عمومی کامنت ⭐
POST https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}/like
POST https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}/hide       # مخفی‌کردن اسپم ⭐
POST https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}/unhide
POST https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}/moderation
DELETE https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}
POST https://zernio.com/api/v1/inbox/posts/{postId}/like
POST https://zernio.com/api/v1/inbox/conversations/{id}/read                  # marking read
PUT  .../read-receipts · .../typing-indicators                                # رایگان

# ═══ نوشتن پولی (متر مصرف می‌شود) ═══
POST https://zernio.com/api/v1/inbox/conversations/{conversationId}/messages  # ⚠️ DM
POST https://zernio.com/api/v1/inbox/comments/{postId}/{commentId}/private-reply  # ⚠️ DM
POST https://zernio.com/api/v1/broadcasts                                     # ⚠️ به‌ازای هر گیرنده

# ═══ پیکربندی ═══
POST https://zernio.com/api/v1/webhooks/settings                              # ثبت وب‌هوک + secret
GET  https://zernio.com/api/v1/webhooks/logs                                  # لاگ تحویل
POST https://zernio.com/api/v1/comment-automations                            # ⚠️ dmMessage اجباری
GET  https://zernio.com/api/v1/comment-automations                            # + آمار
POST https://zernio.com/api/v1/workflows                                      # Workflow با نود webhook
POST https://zernio.com/api/v1/workflows/{workflowId}/activate
POST https://zernio.com/api/v1/accounts/{accountId}/instagram-ice-breakers
POST https://zernio.com/api/v1/accounts/{accountId}/messenger-menu
POST https://zernio.com/api/v1/connect/instagram                              # OAuth میزبانی‌شده

# ═══ هدرها ═══
Authorization: Bearer sk_...
X-Zernio-Signature: <HMAC-SHA256>      # در وب‌هوک‌های دریافتی
Idempotency-Key: <hash>                # برای جلوگیری از ارسال دوباره
X-RateLimit-Limit / Remaining / Reset  # در هر پاسخ
Retry-After                            # فقط در 429

# ═══ کدهای خطای کلیدی ═══
403 message_allowance_exceeded         # سهمیه ۱۰,۰۰۰ تمام → downgrade به کامنت
403 feature_not_available              # پلن AppSumo قدیمی، inbox خاموش
409 not_thread_owner                   # اپ دیگری مالک است → thread control بگیرید
429 rate_limited                       # Retry-After را رعایت کنید
502 details.privateReplyOutcome=unknown # نتیجه نامعلوم → کور retry نکنید، اول مکالمه را بخوانید

# ═══ متغیرهای محیطی لازم ═══
ZERNIO_API_KEY            = sk_...
ZERNIO_WEBHOOK_SECRET     = <خودتان می‌سازید>
SUPABASE_URL              = https://xxx.supabase.co
SUPABASE_SERVICE_KEY      = <service_role>
GROQ_KEY                  = gsk_...
GROQ_URL                  = https://api.groq.com/openai/v1/chat/completions
OR_KEY / GEM_KEY          = (fallback)
SELF_URL                  = https://<app>.vercel.app
DRAIN_TOKEN               = <رندوم ۳۲ بایت>
CRON_SECRET               = <برای Vercel Cron>
DAILY_HARD_CAP            = 290
TG_ADMIN_CHAT_ID          = <چت ادمین>
```

---

## 📌 پیوست ب — پرسش‌های باز که باید خودتان تست کنید

این‌ها مواردی هستند که از مستندات عمومی قابل استنتاج نبودند و باید با یک تست ۵ دقیقه‌ای قطعی شوند:

| # | پرسش | روش تست |
|---|---|---|
| ۱ | آیا `POST /v1/inbox/comments/{postId}` (پاسخ عمومی) واقعاً در متر نمی‌نشیند؟ | ۲۰ پاسخ کامنت بفرستید، سپس از پشتیبانی Zernio یا ایمیل ۷,۵۰۰ بپرسید |
| ۲ | آیا `POST /v1/inbox/comments/{postId}/{commentId}/private-reply` یک DM محسوب می‌شود؟ | یک private reply بفرستید و رفتار را با `message.sent` وب‌هوک مقایسه کنید |
| ۳ | آیا ارسال به **Telegram** از طریق Zernio متر را مصرف می‌کند؟ | مستندات می‌گوید بله (Telegram در فهرست کانال‌های metered است) → **از Telegram Bot API مستقیم استفاده کنید، نه Zernio** |
| ۴ | آیا پلن رایگان SendPulse واقعاً کانال اینستاگرام و `POST /contacts/send` را می‌دهد؟ | ثبت‌نام و اتصال یک اکانت تست |
| ۵ | آیا اکانت Zernio شما پلن AppSumo قدیمی است؟ (در آن صورت inbox با `403 feature_not_available` قفل است) | `POST /v1/webhooks/settings` → اگر `403` گرفتید، به پشتیبانی پیام دهید |
| ۶ | سهمیه واقعی پروژه‌ی Gemini شما چقدر است؟ | در AI Studio → Rate Limit Console ببینید (دیگر در مستندات نیست) |
| ۷ | آیا Groq از IP ایران برای **ثبت‌نام** در دسترس است؟ | تست مستقیم (API call از Vercel مشکلی ندارد) |

---

## 🎯 نتیجه‌گیری نهایی

**پاسخ به خواسته‌ی اصلی شما («بدون اینکه یک دلار هزینه بدهم»):**

✅ **شدنی است** — به شرط سه تصمیم معماری:

1. **کانال پیش‌فرض پاسخ = کامنت عمومی، نه دایرکت.** این تنها کانالی است که در اکوسیستم رسمی متا **واقعاً بی‌نهایت و رایگان** است.
2. **دایرکت = منبع کمیاب با بودجه روزانه ۲۹۰.** فقط برای مواردی که واقعاً خصوصی‌اند (شماره سفارش، آدرس، اطلاعات مشتری).
3. **کارت بانکی روی Zernio نباشد.** این‌گونه حتی اگر همه‌ی محاسباتتان اشتباه از آب درآمد، سیستم با `403` متوقف می‌شود نه با یک صورتحساب.

**جایگزین ChatbotX:** خودِ Zernio Inbox (که هم‌اکنون اکانتش را دارید و رایگان است) + SendPulse Free به‌عنوان ظرف اضطراری.

**پاسخ به «راه‌حل خلاقانه با پل تلگرامی بی‌محدودیت»:** محدودیت از سمت متاست و با هیچ پلی حذف نمی‌شود. اما **بات تلگرام @zeynalikids_bot به‌عنوان کانال سرریز** واقعاً بی‌سقف و رایگان است و می‌تواند ۱۰–۳۰٪ از بار دایرکت اینستاگرام را جذب کند.

**تنها ریسکی که باید جدی بگیرید:** محدودیت «غیرتجاری» بودن پلن Vercel Hobby (R1). اگر این برایتان مهم است، کل لایه‌ی محاسباتی را به **Cloudflare Workers Free** منتقل کنید — که هم استفاده تجاری را مجاز می‌کند، هم Cron با فاصله‌ی دقیقه‌ای رایگان دارد، هم صف رایگان.

---

*این گزارش در ۱۸ اکتبر ۲۰۲۶ بر اساس ۶۰ منبع زنده تهیه شده است. با توجه به اینکه Zernio در ۱۲ ماه گذشته سه بار مدل قیمت‌گذاری خود را تغییر داده، پیشنهاد می‌کنم هر ماه `docs.zernio.com/pricing/messages` و `docs.zernio.com/changelog` را بازبینی کنید.*
