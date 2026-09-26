# خطة مشروع: نظام إدارة المدارس ورياض الأطفال (SaaS)

> توثيق تخطيطي شامل — جاهز ليُعطى كسياق لـ GitHub Copilot / أي أداة AI للبدء بالتنفيذ

---

## 1. نظرة عامة على المشروع

نظام **SaaS متعدد المستأجرين (Multi-tenant)** لإدارة المدارس ورياض الأطفال، يُباع كاشتراك لعدة مدارس (عملاء). كل مدرسة تدير بياناتها الخاصة بشكل معزول تمامًا عن باقي المدارس.

### المراحل (بالترتيب المطلوب)

| المرحلة | الوصف | الحالة |
|---|---|---|
| **1** | تسجيل الأطفال + إدارة الرسوم (دفع يدوي) | **الحالية — جاري التصميم** |
| **2** | الحضور والغياب | لاحقًا |
| **3** | التواصل مع الأهالي (إشعارات/رسائل) | لاحقًا |
| **4 (مستقبلي)** | بوابة دفع إلكتروني | مستقبلي |

### فريق العمل
مهندس Team Lead بخبرة أنظمة ERP + فريق صغير.

---

## 2. القرارات التقنية المعتمدة (نهائية)

| البند | القرار |
|---|---|
| قاعدة البيانات | **PostgreSQL** |
| استراتيجية Multi-tenancy | **قاعدة بيانات منفصلة لكل مدرسة (DB per Tenant)** |
| توزيع قواعد البيانات | **كل قواعد بيانات المدارس على نفس السيرفر (نفس PostgreSQL instance)**، اسم DB مختلف لكل مدرسة |
| نشر التطبيق | **Codebase/نسخة واحدة موحدة** تخدم كل المدارس (Single Deployment)، فقط الاتصال بقاعدة البيانات يتغير ديناميكيًا حسب المدرسة |
| آلية تحديد المدرسة (Tenant Identification) | **Subdomain** (مثال: `schoolA.yourapp.com`) |
| نظام الدفع (المرحلة 1) | تسجيل يدوي فقط (الأدمن يسجل أن الأهل دفعوا) — دفع إلكتروني يُضاف لاحقًا |
| استراتيجية Migrations | **يدوية عبر سكريبت منظم** بالبداية (تشغيل عند الحاجة من الـ terminal)، تُؤتمت لاحقًا مع نمو عدد المدارس |

---

## 3. البنية المعمارية (Architecture)

```
┌───────────────────────────┐
│      Master / Central DB      │
│  (بيانات المدارس، الاشتراكات، │
│   platform_admins، التوجيه)   │
└───────────────────────────┘
              │
              │  الطلب يوصل عبر subdomain
              │  (schoolA.yourapp.com)
              ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │ school_1 │   │ school_2 │   │ school_3 │  ← نفس السيرفر، نفس الـ schema، DB منفصلة
   └──────────┘   └──────────┘   └──────────┘
```

### مكونات رئيسية يجب بناؤها

1. **Tenant Resolver / Routing Middleware** — يقرأ الـ subdomain من الطلب، يستعلم Master DB عن معلومات المدرسة، ويحقن معلومات الاتصال بالـ request context
2. **Tenant Connection Service** — يدير Connection Pool لكل مدرسة (لا يفتح اتصال جديد من الصفر بكل طلب — يحتفظ بـ pool بالذاكرة لكل مدرسة مستخدمة)
3. **Migration Runner** — سكريبت يُطبّق أي تعديل schema على *كل* قواعد بيانات المدارس، ويحدّث `schema_version` بجدول `schools` لكل مدرسة تم تحديثها بنجاح
4. **Onboarding Service** — عند اشتراك مدرسة جديدة: إنشاء DB جديد + تشغيل كل الـ migrations عليه من الصفر
5. **Cross-tenant Reporting Service** (لاحقًا) — لتجميع تقارير على مستوى كل المدارس للوحة Super Admin

### تدفق الطلب (Request Flow) بالتفصيل

```
1. المستخدم يفتح: schoolA.yourapp.com/api/children
2. TenantMiddleware:
   - يستخرج "schoolA" من الـ Host header
   - يستعلم: SELECT * FROM schools WHERE subdomain = 'schoolA'
   - إذا غير موجودة → 404
   - إذا status = 'suspended' → 403 (رسالة اشتراك موقوف)
   - إذا كل شيء تمام → يحقن معلومات DB بالـ request context
3. TenantConnectionService:
   - يتحقق من وجود connection pool مفتوح مسبقًا لـ schoolA بالذاكرة
   - إذا لا، يفتح واحدًا جديدًا (db_name, db_host, db_port من جدول schools)
4. باقي الـ Controllers/Services تستخدم هذا الاتصال المحقون تلقائيًا (Request-scoped Provider في NestJS)
```

### التحديات المعروفة لهذا النهج (يجب الانتباه لها أثناء التنفيذ)

- الـ Migrations يجب أتمتتها تدريجيًا — تطبيق تعديل schema على عدد كبير من المدارس يدويًا غير عملي على المدى الطويل
- Onboarding مدرسة جديدة يحتاج سكريبت آلي منذ البداية لتفادي الأخطاء اليدوية
- استعلامات تقارير شاملة (عبر كل المدارس) أعقد من نظام الـ shared DB — تحتاج تجميع بيانات من كل DB على حدة

---

## 4. الـ Stack المقترح

### Backend
- **NestJS (Node.js + TypeScript)** — بنية منظمة (Modules/Controllers/Services)، مناسبة لمشروع Enterprise-style
  - بديل: Spring Boot (Java) إذا كان الفريق أقرب لخبرة Java/ERP التقليدية

### قاعدة البيانات
- **PostgreSQL** (معتمد)
- ORM: **Prisma** أو **TypeORM** — يحتاج تهيئة خاصة لدعم Multi-database ديناميكي (اتصال مختلف حسب الطلب)
- **Row-Level Security (RLS)** كطبقة حماية إضافية إذا احتجنا فصل صلاحيات إضافي داخل نفس الـ DB

### Frontend
- **React + TypeScript**
- **Next.js** (لو فيه حاجة لصفحة تسويقية/SEO) أو **Vite** (SPA بحت للوحات التحكم)
- **Tailwind CSS** + مكتبة UI (shadcn/ui أو Ant Design)

### الأمان
- Authentication: **JWT + Refresh Tokens** (أو Auth0/Clerk لتوفير وقت)
- Authorization: **RBAC** — أدوار: Super Admin / School Admin / Teacher / Parent
- تشفير الحقول الحساسة (هواتف، عناوين) في قاعدة البيانات
- **Audit Logs** لكل عملية حساسة (خصوصًا المالية)
- الالتزام بمعايير حماية بيانات الأطفال المعمول بها محليًا

### البنية التحتية
- **Docker** لكل الخدمات
- **Redis** للـ caching والـ sessions
- **PgBouncer** لإدارة اتصالات PostgreSQL المتعددة بكفاءة
- استضافة: AWS أو DigitalOcean + Nginx كـ reverse proxy
- CI/CD: GitHub Actions (بداية بسيطة)

---

## 5. تصميم Master DB (تفصيلي)

### جدول `schools`

```sql
CREATE TABLE schools (
    id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name              VARCHAR(255) NOT NULL,             -- اسم المدرسة/الروضة
    subdomain         VARCHAR(63) UNIQUE NOT NULL,        -- schoolA (بدون النطاق الكامل)
    db_name           VARCHAR(63) UNIQUE NOT NULL,        -- اسم قاعدة البيانات الفعلي: school_a
    db_host           VARCHAR(255) NOT NULL DEFAULT 'localhost',
    db_port           INTEGER NOT NULL DEFAULT 5432,
    status            VARCHAR(20) NOT NULL DEFAULT 'active',  -- active / suspended / trial / cancelled
    schema_version    VARCHAR(20),                         -- آخر migration اتطبقت (مهم جداً لتتبع حالة كل مدرسة)
    created_at        TIMESTAMP NOT NULL DEFAULT now(),
    updated_at        TIMESTAMP NOT NULL DEFAULT now()
);

CREATE INDEX idx_schools_subdomain ON schools(subdomain);
```

**شرح الأعمدة المهمة:**
- `subdomain` — هوية المدرسة بالـ URL، فريدة (unique)
- `db_name` — منفصل عمدًا عن subdomain، حتى لو تغيّر subdomain لاحقًا ما بتحتاج تلمس الـ DB
- `db_host` / `db_port` — configurable من اليوم الأول، حتى لو الكل حاليًا بنفس السيرفر (يسهّل التوسع مستقبلًا لأكثر من سيرفر)
- `status` — يتحقق منه الـ middleware قبل السماح بالدخول (suspended = رسالة اشتراك موقوف)
- `schema_version` — يعرف أي مدرسة لسا ما اتطبق عليها آخر تحديث، مهم جدًا لو صار خطأ بمنتصف عملية تحديث جماعي

### جدول `platform_admins`

```sql
CREATE TABLE platform_admins (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email         VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name     VARCHAR(255),
    created_at    TIMESTAMP NOT NULL DEFAULT now()
);
```
هذا الجدول لحسابات صاحب المنصة وفريقه (Super Admin) — يشوفون كل المدارس والاشتراكات. منفصل تمامًا عن مستخدمي كل مدرسة.

### جدول `subscriptions` (مبدئي، يحتاج تفصيل لاحق)

```sql
CREATE TABLE subscriptions (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id     UUID NOT NULL REFERENCES schools(id),
    plan          VARCHAR(50) NOT NULL,       -- اسم الباقة
    status        VARCHAR(20) NOT NULL,       -- active / past_due / cancelled
    starts_at     TIMESTAMP NOT NULL,
    ends_at       TIMESTAMP,
    created_at    TIMESTAMP NOT NULL DEFAULT now()
);
```

---

## 6. مخطط قاعدة بيانات المدرسة (School DB) — مبدئي للمرحلة 1

> ملاحظة: هذا مخطط أولي — يحتاج جلسة تصميم منفصلة لتحديد كل الحقول والعلاقات بدقة (ERD كامل) قبل التنفيذ الفعلي.

### الجداول المتوقعة
- **`children`** — اسم الطفل، تاريخ الميلاد، الصف/الفئة العمرية، حالة التسجيل
- **`guardians`** — بيانات ولي الأمر (اسم، هاتف، بريد إلكتروني)
- **`child_guardian`** — جدول ربط (طفل واحد ممكن يكون له أكثر من ولي أمر)
- **`fee_structures`** — أنواع الرسوم (تسجيل، شهري، إضافي مثل الباص/الأكل)
- **`payments`** — سجل الدفعات (تاريخ، طريقة الدفع اليدوية، الحالة، من سجّلها)
- **`staff` / `users`** — المعلمون والموظفون وصلاحياتهم داخل المدرسة (RBAC محلي)

---

## 7. خطوات العمل التالية (Next Steps) — بالترتيب

- [ ] تثبيت بيئة العمل: Node.js, PostgreSQL, Docker
- [ ] إعداد GitHub Copilot Free (أو Codeium/Windsurf) في VS Code
- [ ] إنشاء هيكل مشروع NestJS الأساسي
- [ ] بناء Master DB: جداول `schools`, `platform_admins`, `subscriptions`
- [ ] بناء `TenantMiddleware` (استخراج subdomain + التحقق من حالة المدرسة)
- [ ] بناء `TenantConnectionService` (إدارة Connection Pools الديناميكية)
- [ ] بناء سكريبت Migration Runner (تطبيق على كل قواعد بيانات المدارس + تحديث `schema_version`)
- [ ] بناء سكريبت Onboarding (إنشاء DB جديد لمدرسة جديدة + تشغيل migrations)
- [ ] تصميم ERD كامل ومفصّل لقاعدة بيانات المدرسة (children, guardians, fees, payments)
- [ ] بناء API تسجيل الأطفال (CRUD)
- [ ] بناء API إدارة الرسوم والدفعات (تسجيل يدوي)
- [ ] بناء واجهة الإدمن (React) للمرحلة 1: تسجيل الأطفال + الرسوم

---

## 8. نقاط قرار مفتوحة (لم تُحسم بعد — تحتاج نقاش لاحق)

- تفاصيل صلاحيات كل دور (Super Admin / School Admin / Teacher / Parent) بدقة
- تفاصيل نظام الاشتراكات والفوترة الخاص بمزوّد الخدمة (الأسعار، الباقات)
- استراتيجية النسخ الاحتياطي لعدد كبير من قواعد البيانات على نفس السيرفر
- متى تنتقل أتمتة الـ Migrations من يدوية إلى تلقائية بالكامل (عند أي عدد من المدارس؟)
- الحقول الدقيقة والعلاقات الكاملة لـ ERD قاعدة بيانات المدرسة

---

## 9. ملاحظة للعمل مع GitHub Copilot

هذا الملف مصمم ليُعطى كسياق كامل لـ Copilot Chat أو أي أداة AI مساعدة عند بدء الكتابة الفعلية للكود. يُنصح بـ:
1. وضع هذا الملف في جذر المشروع (مثلاً `PROJECT_PLAN.md`)
2. الرجوع إليه عند طلب أي كود جديد ("بناءً على PROJECT_PLAN.md، أنشئ...")
3. تحديثه كلما تغيّر قرار معماري أو تمت إضافة تفاصيل جديدة (خصوصًا قسم 8 كلما حُسمت نقطة)
