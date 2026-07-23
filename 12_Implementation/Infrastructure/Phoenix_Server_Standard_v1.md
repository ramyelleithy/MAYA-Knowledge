> **Provenance:** Recovered from `ramyelleithy/PBI-Sales-Agent` on 2026-07-23. Infrastructure standards, unrelated to MAYA's conversational behavior — kept for reference when adding new services to Phoenix Server.

# Phoenix Server Standard v1

الوثيقة المرجعية الرسمية لأي `Service` جديدة تُضاف على `Phoenix Server`. أي `Service` جديدة (`Chatwoot`, `Uptime Kuma`, أو أي `Tool` بعد كده) لازم تُطابق هذه القواعد قبل ما تدخل السيرفر — بدون استثناء.

---

## 1. البنية والتنظيم (Structure)

- **Rule 1:** كل `Service` لها فولدر مستقل تحت `/opt/phoenix/compose/<service>/`.
- **Rule 2:** مفيش `docker-compose` واحد يجمع أكتر من خدمة (No Monolithic Compose). كل `Stack` مستقل بالكامل ويُدار (`start`/`stop`/`restart`) بدون التأثير على غيره.
- **Rule 3:** كل `Service` من يوم واحد لازم يكون معاها 3 ملفات: `docker-compose.yml`, `.env`, `README.md` (حتى لو الـ`README` سطرين بس).

## 2. الشبكة (Networking)

- **Rule 4:** كل `Stack` ينضم لشبكة `external` واحدة مشتركة اسمها `phoenix_net`. الاستقلالية معمولة على مستوى ملفات الـ`Compose`، مش على مستوى الشبكة — التواصل بين الخدمات (زي الاتصال بـ`Postgres` المشترك) بيتم عبرها.
- **Rule 5:** مفيش فتح `ports` مباشرة على الـ`Host` إلا للخدمة اللي بتتكلم مع `Nginx` (`Reverse Proxy`) فقط. أي خدمة داخلية (`Postgres`, `Redis`, إلخ) بتتواصل فقط جوه `phoenix_net`.

## 3. البيانات والتخزين (Data & Storage)

- **Rule 6:** كل `Service` لها `Bind Mount` واضح تحت `/opt/phoenix/data/<service>`.
- **Rule 7:** أي مجلد تحت `/opt/phoenix/data/` يُعتبر تلقائياً جزء من سكريبت الـ`Backup` الموجود — بدون الحاجة لإضافة يدوية مع كل `Service` جديدة.

## 4. الاستقرار والمراقبة (Stability & Monitoring)

- **Rule 8:** كل `Service` لازم يكون لها `Healthcheck` معرّف في الـ`Compose`.
- **Rule 9:** `restart: unless-stopped` هي السياسة الافتراضية لكل `Container` — مش `always` — عشان أي إيقاف يدوي للصيانة يفضل زي ما هو لحد ما حد يشغّله بنفسه.
- **Rule 10:** كل `Container` اسمه يبدأ بـ `phoenix-` (مثال: `phoenix-chatwoot-web`) — عشان الوضوح في `Portainer` ووقت الـ`debugging`.
- **Rule 11:** `Logging Rotation` إجباري على كل `Service` (`max-size` + `max-file` محددين) — عشان منع امتلاء الـ`Disk` (75GB `NVMe`) بملفات `logs` غير محدودة.

## 5. الموارد (Resources)

- **Rule 12:** `Resource Limits` (`mem_limit` وغيرها) بتتضاف فقط بعد ما الخدمة تثبت استقرارها ويبقى عندنا استهلاك فعلي مقاس (عبر `Portainer` أو `Netdata`) — مش تقدير من أول يوم. المبدأ مسجل هنا، لكن التنفيذ الفعلي مؤجل لحد ما تتوفر بيانات حقيقية (نفس منطق قرار ترقية الـ`VPS`: 85%+ لمدة أسبوعين مستمرين).

## 6. الإصدارات (Version Pinning)

- **Rule 13:** ممنوع استخدام `latest` في أي `Docker Image`. كل صورة لازم يكون لها `Tag` صريح ومحدد، يُتحقق منه على `Docker Hub`/`GitHub` وقت التنفيذ الفعلي (مش وقت كتابة الوثيقة دي).
- **Rule 14:** مبدأ الـ`Pinning` يمتد لأي عنصر يؤثر على إعادة بناء السيرفر بشكل مطابق بعد فترة: إصدار `Docker Compose`, إصدار `Ubuntu LTS` (24.04)، وأي `Scripts`/`Dependencies` رئيسية — كلها تُوثّق بأرقامها الدقيقة.
- **Rule 15:** أي ترقية (`Upgrade`) لأي `Image` أو `Dependency` تتم **يدوياً وبمراجعة**، مش تلقائياً — القرار دايماً بإيد رامي، مش بإيد `Docker`.

## 7. استراتيجية الـ`Reverse Proxy` (Nginx)

- **الوضع الحالي (Current State):** `Nginx` شغال مباشرة على مستوى الـ`Host` (مش `Container`)، وبيعمل `Reverse Proxy` لكل الخدمات (`n8n`, `Chatwoot`, وأي خدمة جديدة بعد كده) عن طريق ربطها بـ`127.0.0.1:<port>` على الـ`Host` نفسه. هذا القرار مستمر طالما السيرفر مستقر، ومفيش داعي لتغييره بدون سبب حقيقي (Rule: لا نضيف تعقيد قبل الحاجة الفعلية).
- **الخيار المستقبلي (Future Option):** لو احتجنا لاحقاً نوحّد إدارة كل الخدمات جوه `Docker` بالكامل (مثلاً عشان سهولة النقل بين السيرفرات، أو التوسع لأكتر من `Instance`)، ممكن نحوّل `Nginx` نفسه لـ`Container` منضم لـ`phoenix_net`، ويتواصل مع باقي الخدمات مباشرة عن طريق أسماء الـ`Containers` بدل `127.0.0.1`.
- **الأثر على القاعدة الحالية:** طالما `Nginx` على الـ`Host`، أي خدمة محتاجة `Reverse Proxy` لازم تعمل `Exception` واحدة لـ`Rule 5` (مفيش `Ports` على الـ`Host`) وتربط نفسها بـ`127.0.0.1:<port>` فقط — زي `n8n` (`5678`) و`Chatwoot` (`3000`).

---

### كيفية الاستخدام

قبل تركيب أي `Service` جديدة على `Phoenix Server`، السؤال الأول دايماً:

> هل الـ`Compose` بتاعها مطابق لـ`Phoenix Server Standard v1`؟

- لو **نعم** → نكمل التنفيذ.
- لو **لأ** → نعدلها قبل ما تدخل السيرفر.

هذه الوثيقة هي المرجع الوحيد لأي `Service` جديدة من هنا فصاعداً.
