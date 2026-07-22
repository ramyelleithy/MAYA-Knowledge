# MAYA v2 — Adversarial Runtime Test Plan

**النطاق:** `MAYA - WhatsApp Sales Agent copy` (`jI4meYNr11hP6nbJ`) فقط، Dev فقط.
**الغرض:** كسر النظام، مش تأكيده. لا تنفيذ في هذه المرحلة — Plan فقط، بانتظار الموافقة على الترتيب قبل التنفيذ.
**حالة التنفيذ:** Batch 1 (9 اختبارات: ST-08, ST-09, PC-02, PC-03, PC-04, RG-01 إلى RG-04) اتنفذ ونجح بالكامل — صفر Bugs جديدة. التفاصيل والنتائج في `Release_Notes_MAYA_v2_Beta.md`. ST-09 اتأجل (يحتاج تعديل مباشر على الـ Data Table، مفيش أداة Update متاحة). باقي الخطة (52 اختبار) لسه Backlog — مستني تحديد Batch تانية.
**كيف اتبنى الترتيب:** حسب الأثر الفعلي المحتمل (State خاطئة يظل عالقة للعميل = أعلى من رسالة مش مثالية الصياغة)، وحسب أننا وجدنا Bug حقيقي في نفس بالضبط منطقة الـ State transitions و الـ Credentials مرتين متتاليتين — يعني المناطق دي فيها احتمال أعلى لأخطاء تانية.

---

## ترتيب الأولوية العام (Tier)

| Tier | الفئة | السبب |
|---|---|---|
| **P0** | Sales State — كل الانتقالات المسموحة والممنوعة | فيها Bug اتلقى فعلاً مرتين (New عالقة، Credentials). أعلى احتمال لخطأ تالت. خطأ هنا = بيانات خاطئة تتخزن وتتزامن مع Chatwoot. |
| **P0** | Runtime Failures (خصوصًا Empty Result / Missing Credential-adjacent) | نفس فئة الـ Bug اللي لقيناها. لسه فيه عُقد تانية معتمدة على نفس النمط (Google Drive empty result تحديدًا، مش بس missing credential). |
| **P0** | Regression — كل المسارات اللي سبق إصلاحها | لازم تتأكد إن أي تعديل جديد ما كسرش حاجة قديمة. |
| **P1** | Project Context (مشروع واحد/اتنين/تلاتة) | مباشرة مبني على نفس الكود اللي فيه الـ Bug التاني (Sales State Reader). |
| **P1** | Customer Journey (end-to-end) | بتغطي تكامل كل الأجزاء مع بعض، بتكشف أخطاء تكامل مش منطق منعزل. |
| **P2** | Conversation Memory | مهم لكن أثره على المبيعات غير مباشر (تجربة مستخدم أكتر من قرار بيعي خاطئ). |
| **P2** | Attachments | مسار معروف اتأكد إنه شغال (Bug 1 fix)، لكن فيه أنواع ملفات تانية لسه ما اتاختبرتش. |
| **P3** | أنواع رسائل غريبة (صوت، أزرار، ستيكر، موقع) | نادر الحدوث نسبيًا، لكن لو انكسر ممكن يوقف المحادثة كاملة. |
| **P3** | Concurrency / Duplicate Webhook | صعب تنفيذه يدويًا، لكن خطر حقيقي في Production لو حصل. مطروح كفحص استكشافي مش كـ Test Case قياسي. |

---

## 1) Sales State — كل الانتقالات (P0)

### 1.1 الانتقالات المسموحة (Allowed Transitions)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | ممكن ينكسر إيه | Manual/Auto |
|---|---|---|---|---|---|
| ST-01 | `New → Qualifying` عند الرسالة التانية بدون كود مشروع | رقم جديد: رسالة 1 "مرحبا"، رسالة 2 "عايز اعرف اكتر" (بدون referral) | `sales_state = Qualifying` بعد الرسالة التانية بالظبط، مش الأولى | لو اتحسبت من الرسالة الأولى غلط، أو لو ما اتحسبتش خالص | Auto (webhook calls) |
| ST-02 | `New → Exploring` مباشرة لو أول رسالة فيها كود مشروع (Bug 2 fix) | رقم جديد + referral `[MRG-SHERATON]` في أول رسالة | `sales_state = Exploring` من أول Turn | Regression للـ Bug اللي اتصلح | Auto |
| ST-03 | `Qualifying → Exploring` لو المشروع ظهر في رسالة تالتة+ | رقم: رسالة1 عادية، رسالة2 عادية (Qualifying)، رسالة3 فيها كود مشروع (بدون referral — لازم نتأكد إزاي بيتكشف الكود من غير referral، هل فيه استخراج من النص نفسه ولا الكود بيتكشف من الـ Referral بس؟) | لو الاكتشاف referral-only: يفضل Qualifying لأن مفيش referral تاني. **هذا اختبار لازم يتأكد أولاً هل النظام أصلاً بيقدر يكتشف مشروع من نص عادي، أو الـ Referral هو الطريقة الوحيدة** | فرضية أساسية غلط لو مفيش طريقة تانية لتحديد مشروع غير أول Referral | Manual (يحتاج فحص كود Extract Project Code الأول) |
| ST-04 | `Exploring → Interested` عند Interest Score ≥ 4 | رسالة فيها كلمتين إشارة (مثلاً ميزانية + تقسيط) على مشروع فعّال | `sales_state = Interested`، `interest_score ≥ 4` في `project_history` | لو الـ Regex مش بيتطابق مع صياغات عامية تانية (زي "5 مليون" من غير كلمة جنيه) | Auto |
| ST-05 | `Exploring → Awaiting Callback` مباشرة (تخطي Interested) | رسالة فيها طلب رقم تواصل صريح وهي أول إشارة اهتمام (Score < 4) | `sales_state = Awaiting Callback` رغم إن Score متوصلش 4 | لو الأولوية اتقلبت وقفت عند Interested بدل ما تكمل لـ Awaiting Callback | Auto |
| ST-06 | Resume من `Follow-up`/`Dormant` لـ `previous_active_state` | يحتاج تعديل يدوي في Data Table (`sales_state=Follow-up`, `previous_active_state=Interested`) ثم رسالة جديدة | `sales_state` يرجع `Interested` فورًا، `previous_active_state` يترجع `null` | لو فضلت Follow-up عالقة، أو لو previous_active_state ماتصفرش (تسرب بيانات قديمة لدورة تالية) | Manual (يحتاج تعديل مباشر في Data Table لأن الدخول التلقائي لسه مش موجود) |
| ST-07 | `→ Closed Lost` من أي حالة غير-Terminal | رسالة فيها كلمة رفض صريحة ("مش هكمل معاكم") من حالة `Interested` | `sales_state = Closed Lost` | لو الانتقال اتوقف بسبب حالة وسيطة (زي Awaiting Callback) لسبب مش متوقع | Auto |

### 1.2 الانتقالات الممنوعة (Forbidden — لازم تتأكد إنها **ما بتحصلش**)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | ممكن ينكسر إيه | Manual/Auto |
|---|---|---|---|---|---|
| ST-08 | `Closed Lost` لازم يفضل عالق (Sticky) — مفيش رجوع تلقائي | من حالة `Closed Lost`، ابعت رسالة فيها كود مشروع جديد + كلام اهتمام قوي | `sales_state` يفضل `Closed Lost` رغم وجود مشروع/اهتمام جديد | لو ده Bug حقيقي، معناه عميل رافض بيرجع "مهتم" تلقائيًا من غير تدخل بشري — خطر عالي على تقارير المبيعات | Auto |
| ST-09 | `Closed Won` لازم يفضل عالق | نفس ST-08 لكن يبدأ من `Closed Won` (يحتاج تعديل يدوي أولاً لأنه Manual-only) | يفضل `Closed Won` | نفس منطق ST-08 | Manual (تعديل يدوي أول) |
| ST-10 | `Interested → Exploring` (رجوع للخلف) ما يحصلش تلقائيًا لمجرد رسالة عادية | من `Interested`، ابعت رسالة عادية بدون كلمات اهتمام أو رفض | `sales_state` يفضل `Interested` (مفيش تراجع تلقائي للخلف في التصميم الحالي) | لو الكود بيعيد حساب الحالة من الصفر بدل ما ياخد أعلى حالة وصلها العميل | Auto |
| ST-11 | `Awaiting Callback` ما يتغيرش بمجرد ذكر مشروع جديد | من `Awaiting Callback`، اذكر كود مشروع مختلف تاني | `sales_state` يفضل `Awaiting Callback`، لكن `active_project` يتحدث للمشروع الجديد | ده Edge Case مثير للاهتمام: هل فعلاً المفروض الـ State يفضل زي ما هي؟ ولا لازم القاعدة تتراجع لو العميل بدأ موضوع تاني خالص؟ **يحتاج قرار عمل قبل ما نحكم عليه Bug أو سلوك مقصود** | Auto + قرار بشري |
| ST-12 | `New`/`Qualifying` + `callback_requested=true` — هل بتقفز لـ `Awaiting Callback`؟ | من `New` أو `Qualifying` (بدون مشروع)، اطلب رقم تواصل مباشرة | حسب الكود الحالي: **لا تقفز** لأن `engagedStates` بتستثني New/Qualifying — لازم تتأكد إن ده مقصود مش سهو، لأن طلب Callback من عميل لسه في New معناه Lead ساخن ومهم | لو ده فعلاً Gap — عميل بيطلب مكالمة من أول رسالة بيتسجل Sales State غلط (فاضل Qualifying رغم إنه طلب تواصل) | Auto + قرار عمل |

---

## 2) Runtime Failures (P0)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | ممكن ينكسر إيه | Manual/Auto |
|---|---|---|---|---|---|
| RF-01 | Missing Project (كود مش موجود في Google Sheet) | Referral بكود وهمي `[XYZ-NOTREAL]` | يمشي في مسار Project Not Found بدون error، رد افتراضي، `sales_state` يتحدث بشكل صحيح | تم اختباره جزئيًا قبل كده، لكن لازم يتأكد إنه لسه شغال بعد كل التعديلات الأخيرة | Auto |
| RF-02 | **Google Drive Folder Search بيرجع نتيجة فاضية** (كود المشروع موجود في الـ Sheet، لكن مفيش فولدر بنفس الاسم في Drive) | كود مشروع جديد نضيفه للـ Sheet مؤقتًا بدون فولدر مطابق، أو نستخدم اسم فولدر متغير عمدًا | **هذا أهم اختبار في الفئة دي.** لازم نتأكد هل `Check Project Folder Exists` بيتعامل مع النتيجة الفاضية زي ما `Project Mapping Found?` بيتعامل مع الـ Sheet الفاضي (Finding #15)، ولا هيرمي Error زي الـ Bug اللي اتصلح في Credentials | لو مفيش IF-node مكافئ بعد `Search Project Folder`، ده نفس فئة Bug 1 لكن Empty-Result مش Missing-Credential — **احتمال كبير إنه موجود ولسه ما انلقاش** | Manual (يحتاج تجهيز بيانات وهمية في Sheet/Drive) |
| RF-03 | Missing File — الفولدر موجود بس ملف `Project_Bible.md` أو `Sales_Playbook.md` مش موجود جواه | مشروع حقيقي بس نمسح الملف مؤقتًا (أو نستخدم مشروع تجريبي بفولدر فاضي) | نفس منطق RF-02 لكن على مستوى الملف مش الفولدر — لازم رد لطيف بدل Error | خطر التوقف الكامل للمحادثة | Manual |
| RF-04 | Missing Client File عند طلب بروشور/Master Plan لمشروع معندوش الملف ده | اطلب `master_plan` لمشروع عنده Brochure بس مفيش Master Plan فيه | رد واضح إن الملف مش متاح، بدون تعليق الـ Workflow | لو الـ `Find Matching Client File` مالوش IF واضح لحالة "لا يوجد نتيجة" | Auto/Manual |
| RF-05 | Missing Chatwoot Custom Attribute (Sync يفشل) | صعب نفتعله بدون تعديل الـ Credential مؤقتًا — بديل: نراقب لو `Sync Sales State to Chatwoot` فشل هل باقي الـ Pipeline (خصوصًا Send WhatsApp Reply) اتأثر؟ | التوثيق بيقول `onError: continueRegularOutput` — لازم تأكيد حي إن الرسالة اتبعتت للعميل حتى لو الـ Sync فشل | لو في الواقع الفشل بيوقف حاجة تانية بعده رغم `continueRegularOutput` | Manual (يحتاج تعطيل مؤقت ومقصود للـ Credential، خطر — يحتاج إذن صريح قبل التنفيذ) |
| RF-06 | Duplicate Webhook Delivery (نفس `wamid` يوصل مرتين) | نفس الـ payload بالظبط (نفس `id`/`wamid`) يتبعت مرتين متتاليتين | المفروض معالجة واحدة بس، أو على الأقل مفيش Double-reply للعميل ولا Double turn_count | WhatsApp Cloud API فعليًا بيعيد المحاولة لو الـ webhook ما ردش 200 بسرعة — لو مفيش Idempotency check، العميل ممكن ياخد نفس الرد مرتين وturn_count يتضاعف غلط | Manual/استكشافي |
| RF-07 | رسالتين من نفس الرقم في وقت متقارب جدًا (Race Condition) | تنفيذ Execution لنفس الرقم مرتين بفارق ثواني قليلة جدًا (شبه متزامن) | لازم الحالة النهائية تعكس آخر رسالة صح، من غير Overwrite لبيانات وسطى (زي `project_history`) | نمط Read-Modify-Write بدون Lock — احتمال حقيقي لفقدان تحديثات | Manual/استكشافي، صعب التحكم فيه بدقة عبر Manual execution |

---

## 3) Regression — إعادة تشغيل كل المسارات اللي اتصلحت (P0)

| # | المسار | بيانات الإدخال | النتيجة المتوقعة | Manual/Auto |
|---|---|---|---|---|
| RG-01 | Project Not Found | Referral بكود وهمي | رد افتراضي صحيح، Sales State يتحدث، Chatwoot Sync ناجح | Auto |
| RG-02 | Project Found (كامل) | Referral بكود حقيقي (MRG-SHERATON) + سؤال | Project Bible + Sales Playbook يتحملوا، رد من MAYA، Sales State صحيح | Auto |
| RG-03 | Callback Request | "محتاج رقم للتواصل" | رد الـ Callback المحدد، `callback_requested=true`، الانتقال لـ Awaiting Callback لو الحالة Engaged | Auto |
| RG-04 | File Request (Brochure) | "ابعتلي البروشور" على مشروع فيه ملف | الملف يتحمل ويترفع وينبعت فعليًا عبر WhatsApp | Auto |
| RG-05 | Financing (مسار محجوب) | سؤال عن "التمويل البنكي" | رد محجوب زي ما التصميم بيقول، من غير ما يوصل لـ Ask MAYA | Auto |
| RG-06 | Spam / رسائل غير عقارية | رسالة عرض خدمة غير متعلقة (زي "عندي عرض تسويق ليك") | رد محجوب مشابه، من غير تفعيل باقي الـ Pipeline | Auto |
| RG-07 | First Message + Project Code (Bug 2) | referral بكود حقيقي في أول رسالة | `Exploring` من أول Turn (تأكيد إضافي بعد أي تعديل لاحق) | Auto |
| RG-08 | كل المسارات التلاتة (Financing/Spam/Direct Callback) بتوصل لنقطة الالتقاء الجديدة (Writer/Persist/Sync) من غير Error | تنفيذ الثلاثة بالتتابع | كل واحدة تكمل لـ `Sales State Writer` → `Persist Sales State` → `Sync Sales State to Chatwoot` بنجاح | Auto |

---

## 4) Project Context (P1)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | ممكن ينكسر إيه | Manual/Auto |
|---|---|---|---|---|---|
| PC-01 | مشروع واحد — Turn متعددة | 4 رسائل بنفس كود المشروع | `turn_count` يوصل 4، `interest_score` فيه +1 بالظبط عند الـ Turn التالت مش غيره | تكرار الـ Bonus على كل Turn بدل مرة واحدة فقط | Auto |
| PC-02 | مشروعان — تبديل | مشروع A (رسالتين) ثم مشروع B (رسالتين) | `active_project=B`، `project_history` فيه Entry لـ A و B منفصلين، `sales_state` **ما بترجعش** لـ New/Qualifying | فقدان تاريخ A، أو Reset غير متوقع لـ sales_state | Auto |
| PC-03 | ثلاثة مشاريع | A ثم B ثم C، كل واحد برسالتين | `project_history` فيه 3 Entries، كل واحد بـ `turn_count=2` مستقل، `active_project=C` | تداخل الـ turn_count بين المشاريع (زيادة خطأ لمشروع تاني) | Auto |
| PC-04 | الرجوع للمشروع الأول بعد التنقل | A → B → A تاني | `active_project=A` مرة تانية، الـ Entry الأصلي لـ A يتحدّث (`turn_count` يزيد من قيمته السابقة مش من الصفر)، `last_seen_at` يتحدث | لو النظام بينشئ Entry جديد لـ A بدل ما يحدث القديم (فقدان الـ turn_count التراكمي) | Auto |
| PC-05 | Interest Score تراكمي عبر مشروعين مختلفين لا يتداخل | اهتمام قوي بمشروع A (Score يوصل 4)، بعدين تبديل لمشروع B بدون أي إشارة اهتمام | Score الخاص بـ A يفضل زي ما هو، B يبدأ من 0 — و`sales_state` (لو Interested بسبب A) هل يفضل Interested رغم التبديل لمشروع B؟ | ده تحديدًا نقطة غامضة زي ST-11 — القرار الحالي في الكود إن الـ Score محفوظ بس الـ sales_state عام مش لكل مشروع | Auto + قرار عمل |

---

## 5) Customer Journey — End to End (P1)

| # | السيناريو | بيانات الإدخال | النتيجة المتوقعة | Manual/Auto |
|---|---|---|---|---|
| CJ-01 | أول رسالة بدون Referral | "السلام عليكم" فقط | `New`، مفيش `active_project`، رد ترحيبي عام | Auto |
| CJ-02 | أول رسالة مع Referral لمشروع حقيقي | Referral MRG-SHERATON | `Exploring` مباشرة (تأكيد Bug 2) | Auto |
| CJ-03 | أول رسالة مع Referral لمشروع غير موجود | Referral بكود وهمي | مسار Project Not Found، Sales State لا يزال يتحدث بشكل صحيح (New أو حسب المنطق) | Auto |
| CJ-04 | تغيير المشروع منتصف المحادثة | يبدأ بمشروع A، يسأل بعدين عن مشروع B صراحة | `active_project` يتحدث، الرد يتكلم عن B مش A (تأكد إن الـ Context مش فيه تلوث من مشروع A في رد B) | Auto |
| CJ-05 | العودة لنفس المشروع | (نفس PC-04 لكن من منظور تجربة المحادثة) | الرد يعكس إدراك إن ده مشروع اتكلمنا فيه قبل كده (لو الـ Summary بيوصل صح لـ Ask MAYA) | Auto |
| CJ-06 | مقارنة بين مشروعين صراحة | "إيه الفرق بين [مشروع أ] و[مشروع ب]؟" في نفس الرسالة | هل الـ Regex بتاخد أول Referral بس؟ وهل الرد بيغطي الاتنين ولا بس المشروع النشط؟ | Manual (يحتاج صياغة دقيقة وملاحظة الرد) |
| CJ-07 | طلب بروشور | "ابعتلي البروشور" | ملف PDF يوصل فعليًا، `file_request=brochure` | Auto (تأكد إضافي بعد Bug 1 fix) |
| CJ-08 | طلب مكالمة صريح | "عايز حد يكلمني" | رد الـ Callback القياسي، الانتقال الصحيح لـ Sales State | Auto |
| CJ-09 | رفض صريح | "مش هكمل معاكم خالص" | `Closed Lost`، ووقف أي محاولة بيع بعد كده | Auto |
| CJ-10 | رفض غير صريح (False Positive check) | جملة فيها "مش" بس مش رفض فعلي، زي "مش عايز تفاصيل زيادة دلوقتي، بس هرجعلك بعدين" | Sales State **ما ينزلش** لـ Closed Lost غلط | Auto — هذا اختبار حرج للـ Precision |
| CJ-11 | عودة بعد انقطاع طويل (محاكاة، مش انتظار فعلي) | تعديل `sales_state_updated_at` يدويًا لتاريخ قديم (>30 يوم) ثم رسالة جديدة | تأكيد إن الفجوة المعروفة (لا يوجد دخول تلقائي لـ Follow-up/Dormant) بتتصرف زي ما هو موثق، مش بسلوك غير متوقع تاني | Manual |
| CJ-12 | رسالة غامضة تمامًا لا علاقة لها بالعقارات ولا Spam واضح | "إزيك عامل إيه النهاردة؟" (دردشة عادية) | رد طبيعي بدون Business Rule Block، بدون فرض أسئلة تأهيل قسرية | Auto |

---

## 6) Conversation Memory (P2)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | Manual/Auto |
|---|---|---|---|---|
| CM-01 | رسائل قصيرة جدًا متتالية | "تمام"، "ok"، "👍" على التوالي | الردود متكيفة مع القصر، من غير تكرار ترحيب | Auto |
| CM-02 | رسالة طويلة جدًا (فقرة كاملة بأسئلة متعددة) | رسالة من 5-6 أسطر فيها أكتر من سؤال في نفس الوقت | الرد يغطي على الأقل السؤال الأهم، ما ينهارش أو يتجاهل الرسالة كلها | Manual (يحتاج قراءة الرد بعناية) |
| CM-03 | إعادة نفس السؤال حرفيًا مرتين | نفس الرسالة بالظبط مرتين متتاليتين | الرد الثاني ما يكررش نفس الصياغة الحرفية (المفروض فيه قاعدة لتفادي التكرار زي حالة الـ Callback) | Manual |
| CM-04 | تغيير الموضوع فجأة | من سؤال عن السعر لسؤال عن المدارس القريبة لنفس المشروع | الرد يتابع الموضوع الجديد من غير التصاق بالسؤال القديم | Auto |
| CM-05 | الرجوع لموضوع سابق بعد تغييره | سؤال عن الموقع، بعدين عن التمويل (محجوب)، بعدين رجوع للموقع تاني | الرد يرجع للسياق الصح، `conversation_summary` ما تلخبطش | Manual |
| CM-06 | `conversation_summary` بعد محادثة طويلة (10+ رسائل) | محادثة طويلة متنوعة المواضيع | الملخص يفضل مركّز وما يكبرش بشكل غير منضبط، ولا يفقد معلومة حرجة (زي المشروع النشط) | Manual |

---

## 7) Attachments (P2)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | Manual/Auto |
|---|---|---|---|---|
| AT-01 | طلب Brochure | "ابعتلي البروشور" | PDF يوصل (تم التأكد جزئيًا في Bug 1 fix — يحتاج تكرار على مشروع تاني غير Sheraton) | Auto |
| AT-02 | طلب Master Plan | "عايز أشوف الـ Master Plan" | ملف الـ Master Plan يوصل، أو رد واضح لو مش متاح | Auto |
| AT-03 | طلب Price List (لو موجود كفئة) | "ابعتلي قائمة الأسعار" | حسب الـ Schema: `file_request` مفهوش قيمة `price_list` صراحة — لازم تتأكد الرد بيتعامل صح مع طلب مش موجود في الـ enum (يرجع فاضي ويوضح إنه مش متاح دلوقتي) | Manual |
| AT-04 | طلب صور (`images`) | "عندك صور للمشروع؟" | صور تتبعت أو رد واضح لو مفيش | Auto |
| AT-05 | طلب ملف على مشروع Project Not Found | نفس AT-01 لكن على كود مشروع وهمي | لازم رد يوضح المشروع مش موجود، من غير محاولة تحميل ملف من فولدر مش موجود أصلاً | Manual |
| AT-06 | طلب نوع ملف مش مدعوم (فيديو مثلاً) | "ابعتلي فيديو عن المشروع" | حسب التصميم الموثق: `file_request` يفضل فاضي والرد يوضح إنه مش متاح | Auto |

---

## 8) أنواع رسائل غير نصية (P3)

| # | الهدف | بيانات الإدخال | النتيجة المتوقعة | ممكن ينكسر إيه | Manual/Auto |
|---|---|---|---|---|
| MT-01 | رسالة صوتية (Voice Note) | `type: audio` بدل `text` | النظام يتعامل معاها بشكل معقول (تحويل/تجاهل موضح للعميل)، مش Crash | فيه Node مخصص لـ audio (`Get Media URL`/`Download Media`) — لازم تتأكد الاستخدام الفعلي منها سليم بعد إصلاح الـ Credentials | Manual |
| MT-02 | رد بزرار (Interactive/List Reply) | `type: interactive`, `list_reply.id/title` معبى | يتفهم كاختيار وقت Callback مثلاً، مش كنص خام غامض | لو `Resolve Customer Message` مش بيقرا `interactive.list_reply.title` صح | Manual |
| MT-03 | رسالة فاضية تمامًا (نص فاضي أو مسافة بس) | `text.body: " "` | رد لطيف يطلب توضيح، مش Crash ولا رد فاضي يترجع للعميل | Auto |
| MT-04 | رسالة Sticker/Location/Contact Card | `type` مختلف تمامًا زي `sticker` أو `location` | رد يوضح إن النوع ده مش مدعوم، بدون كسر الـ Pipeline | Manual |
| MT-05 | Referral Headline بصيغة غريبة | `[MRG-SHERATON][EXTRA]`, أو `MRG-SHERATON` من غير أقواس, أو `[mrg-sheraton]` بحروف صغيرة, أو `[ MRG-SHERATON ]` بمسافات | تأكيد سلوك الـ Regex الفعلي مقابل كل صيغة — أي منها بينجح وأي منها بيرجع `projectCode: null` | Auto |

---

## ملخص عدد الاختبارات

| Tier | عدد الاختبارات |
|---|---|
| P0 | 7 (Sales State المسموح) + 5 (Sales State الممنوع) + 7 (Runtime Failures) + 8 (Regression) = **27** |
| P1 | 5 (Project Context) + 12 (Customer Journey) = **17** |
| P2 | 6 (Memory) + 6 (Attachments) = **12** |
| P3 | 5 (أنواع رسائل) = **5** |
| **الإجمالي** | **61 Test Case** |

---

## نقاط تحتاج قرار عمل قبل التنفيذ (مش أخطاء، أسئلة تصميم)

هذول مش Bugs — دول أسئلة سلوك لازم تتحدد الإجابة الصح ليها قبل ما نحكم على نتيجة الاختبار إنها Pass أو Fail:

1. **ST-11 / PC-05:** لو العميل في `Awaiting Callback` أو `Interested` بسبب مشروع معين، وبعدين ذكر مشروع تاني تمامًا — هل `sales_state` المفروض يفضل زي ما هو (تصميم حالي) ولا يتأثر بالمشروع الجديد؟
2. **ST-12:** طلب Callback من عميل في `New`/`Qualifying` (قبل أي مشروع) — مقصود إنه ما يتحولش لـ `Awaiting Callback` ولا ده فجوة؟
3. **AT-03:** لو عميل طلب "قائمة أسعار" صراحة وهي مش موجودة كفئة في الـ Schema — هل المتوقع إنها تتصنف كـ Brochure (لو فيها أسعار) ولا ترفض؟

---

## اقتراح ترتيب التنفيذ الفعلي (الدفعة الأولى المقترحة)

بما إننا لقينا Bug حقيقي مرتين في نفس المنطقتين (State transitions + Credentials/Empty-results)، أقترح نبدأ بالدفعة دي تحديدًا لأنها الأعلى احتمال لكشف Bug تالت:

1. **RF-02** (Google Drive Empty Folder Result) — نفس فئة الـ Bug اللي لقيناها، لسه ما اتاختبرتش
2. **ST-08, ST-09** (Closed states لازم تفضل عالقة)
3. **ST-11, ST-12, PC-05** (نقاط القرار — نحتاج نحسمها الأول عشان نعرف نحكم على باقي الاختبارات صح)
4. **PC-02, PC-03, PC-04** (Multi-project correctness)
5. باقي RG-* (Regression الكامل)
6. باقي P1/P2/P3 حسب الوقت المتاح

في انتظار توجيهك لبدء التنفيذ.
