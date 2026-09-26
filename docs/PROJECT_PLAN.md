# خطة المشروع

## نظرة عامة

نظام SaaS لإدارة المدارس ورياض الأطفال، متعدد المستأجرين.

## المرحلة الحالية

- تسجيل الأطفال
- إدارة الرسوم
- تسجيل المدفوعات يدويًا
- إدارة معلومات أولياء الأمور

## البنية المقترحة

- apps/api: NestJS backend
- apps/web: React + Next.js أو Vite frontend
- packages/shared: models and validation shared logic
- docker: services configuration

## التحديات الرئيسية

- Tenant routing via subdomain
- DB per school
- Dynamic connection per tenant
- Migration runner
- Onboarding for new schools
