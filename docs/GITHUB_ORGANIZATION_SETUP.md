# إعداد ربط المشروع مع GitHub Organization

## 1. الحالة الحالية

تم تجهيز المشروع داخل المجلد الحالي بشكل تنظيمي، لكن الربط الفعلي مع GitHub لا يمكن تنفيذه من هنا لأن Git غير موجود في البيئة الحالية. تم التحقق من ذلك عبر محاولة تشغيل أمر git، ونتيجة الأمر تشير إلى أن Git غير مثبت أو غير متاح في هذا الجهاز.

## 2. الخطوات المطلوبة على جهازك

### الخطوة 1: إنشاء Organization على GitHub
- افتح GitHub
- اضغط على New organization
- اختر النوع المناسب
- املأ اسم المنظمة، مثل:
  - school-saas
  - school-management-platform
- قم بإنشاء المنظمة

### الخطوة 2: إنشاء Repository داخل Organization
- داخل المنظمة، اختر New repository
- اسم Repo مثال:
  - school-saas-platform
  - school-management-system
- اختر:
  - Public أو Private حسب سياسة الفريق
  - Add a README: لا نحتاجه هنا لأن المشروع موجود محليًا
- إنشاء الـ Repository

### الخطوة 3: إعداد الأعضاء والفرق
- داخل Organization → People
- أضف أعضاء الفريق
- أنشئ Teams مثل:
  - Tech Lead
  - Backend Team
  - Frontend Team
  - QA Team
  - DevOps Team
- خصّص صلاحيات لكل فريق حسب الحاجة

### الخطوة 4: إعداد الحماية على الفرع الرئيسي
- Go to Settings → Branches
- Add rule
- اختر الفرع الرئيسي، مثل main
- فعّل:
  - Require pull request reviews before merging
  - Require status checks to pass before merging
  - Restrict who can push to matching branches

## 3. ربط المشروع المحلي مع Repository الخاص بالمنظمة

بعد تثبيت Git على الجهاز، نفّذ هذه الأوامر داخل مجلد المشروع:

```bash
git init
git branch -M main
git add .
git commit -m "chore: initialize project structure"
git remote add origin https://github.com/<ORG_NAME>/<REPO_NAME>.git
git push -u origin main
```

مثال:

```bash
git init
git branch -M main
git add .
git commit -m "chore: initialize project structure"
git remote add origin https://github.com/school-saas/school-saas-platform.git
git push -u origin main
```

## 4. تنظيم العمل داخل GitHub

### إنشاء Projects
- Create Project
- اختر Board
- أضف Columns:
  - Backlog
  - To Do
  - In Progress
  - Review
  - Done

### إنشاء Issues
- كل مهمة كبيرة أو صغيرة تُنشأ كـ Issue
- كل Issue يجب أن يحتوي على:
  - العنوان
  - الوصف
  - القيم/النتيجة المطلوبة
  - المسؤول
  - القيم المتعلقة بالمرحلة

### ربط المهام مع PR
- عند تنفيذ مهمة، يتم إنشاء branch جديد
- مثال:
  - feature/children-module
  - feature/fees-module
  - fix/tenant-routing
- ثم فتح Pull Request مرتبط بالـ Issue

## 5. القواعد الموصى بها للفريق

- كل تطوير يفتح في branch منفصل
- لا يُسمح بالدفع المباشر إلى main
- كل PR يحتاج review من عضو آخر
- كل مهمة مرتبطة بـ Issue أو Task
- كل مرحلة لها owner و acceptance criteria

## 6. التوصية العملية

الأفضل أن يكون هناك:
- Organization باسم المشروع
- Repository واحد رئيسي
- Projects للحالة
- Teams للأدوار
- Branch protection على main

## 7. الخطوة التالية

بعد أن يتم إنشاء Organización و Repository على GitHub، يمكنني مساعدتك في:
- كتابة اسم المنظمة واسم الـ Repository المناسبين
- تجهيز جدول المهام داخل GitHub Projects
- تنظيم الـ branches و naming conventions
- إعداد ستيب-by-step كامل لربط المشروع فعليًا
