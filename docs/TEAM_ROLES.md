# توزيع الأدوار داخل الفريق

## 1. Team Lead / Tech Lead
**المسؤوليات:**
- متابعة التنفيذ العام للمشروع
- اتخاذ قرارات البنية التقنية
- مراجعة القواعد والمعمارية
- إدارة أولويات المرحلة الحالية
- حل التعارضات الفنية

**الجاهزية المطلوبة:**
- خبرة في ERP أو أنظمة المؤسسات
- معرفة متقدمة بـ NestJS / PostgreSQL / System Design

## 2. Backend Developer
**المسؤوليات:**
- بناء APIs
- إدارة قاعدة البيانات
- كتابة services و business logic
- تصميم Tenant-aware architecture
- إعداد migrations

**المهام المقترحة في المرحلة الأولى:**
- Master DB
- School DB
- Children Module
- Guardians Module
- Fees and Payments

## 3. Frontend Developer
**المسؤوليات:**
- تصميم واجهة الإدارة
- بناء صفحات تسجيل الأطفال والرسوم
- توصيل الواجهة مع الـ API
- تحسين تجربة المستخدم

**المهام المقترحة في المرحلة الأولى:**
- Login and dashboard
- Children registration form
- Fees and payment management page
- Parent data entry UI

## 4. QA / Tester
**المسؤوليات:**
- كتابة test cases
- اختبار السيناريوهات الأساسية
- التحقق من القبول
- مراجعة الـ bugs والارتجاعيات

**المهام المقترحة:**
- Manual QA for registration flow
- Fee payment validation
- Access permissions verification

## 5. DevOps / Infrastructure
**المسؤوليات:**
- إعداد Docker
- PostgreSQL container config
- CI/CD setup
- Environment variables and secrets
- Deployment and monitoring

## 6. Product Owner / Stakeholder
**المسؤوليات:**
- تحديد المتطلبات
- مراجعة النتائج
- إعطاء الأولوية للمزايا
- قبول أو رفض الـ outcomes

## 7. Distribution of Work by Phase

### Phase 1: Foundation
- Tech Lead: architecture and review
- Backend Developer: master db + tenant routing
- DevOps: postgres + docker setup
- QA: check environment setup and test flow

### Phase 2: Children & Fees
- Backend Developer: child APIs
- Frontend Developer: registration UI
- QA: test forms and validation

### Phase 3: School Admin / Reporting
- Backend Developer: analytics + reports
- Frontend Developer: dashboard views
- QA: regression and usability checks

## 8. Communication Rules

- كل عضو يلتزم بتحديث حالة المهمة بشكل دوري.
- أي مشكلة تؤثر على timeline يجب إخطار Tech Lead فورًا.
- أي طلب جديد يحتاج موافقة Product Owner قبل التنفيذ.

## 9. Team Working Principles

- الشفافية قبل التنفيذ
- التواصل المستمر
- التوثيق قبل الإغلاق
- الالتزام بالمواعيد المتفق عليها
- مراجعة الكود قبل دمجه
