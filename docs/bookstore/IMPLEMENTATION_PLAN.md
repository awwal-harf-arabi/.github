# تنفيذ موقع بيع «أول حرف عربي» — مراحل MVP

> هذا مشروع مخطط، وليس ادعاء بوجود متجر حي أو معالجة مدفوعات ناجحة. أنشئ مستودعاً خاصاً اسمه bookstore تحت المؤسسة عند بدء التطوير.

## Phase 0 — معرض وقراءة عينة (قابل للنشر)
- Laravel + Livewire + Tailwind + Filament admin.
- كتالوج Series -> Book -> Edition مع حالة editorial / publish، Locale ومفهوم direction.
- أغلفة معتمدة؛ عينة مجانية من فصول approved فقط مع خيار إخفاء الحرق.
- صفحات عربية وإنجليزية وSEO وOpenGraph وsitemap من المنتجات المنشورة فقط.
- صفحة Coming Soon للكتب غير المكتملة بلا زر شراء أو سعر كاذب.
- بريد قائمة انتظار Opt-in مؤكد وunsubscribe، analytics privacy-first.
- اختبارات: unpublished never listed as purchasable; locales; sample contains only approved paragraphs.
- يكتمل حين يمكن نشر موقع تعريفي دون بيع.

## Phase 1 — بيع إلكتروني آمن (قابل للنشر)
- Catalog Pricing مع snapshot في OrderItems، دعم عملة واحدة أولاً ثم متعددة.
- Checkout + provider adapter + signed webhook + reconciliation + refund handling.
- Orders/Payments/Entitlements/DownloadGrants، تأكيد البريد، My Library.
- تخزين رقمي خاص، signed short-lived URLs، rate limits، event audit.
- حماية: policy على order/download، webhook idempotency، replay prevention، CSRF، escaping، signed order references، retry queues.
- نظام publish manifest: إصدار معتمد من repo خاص -> artefacts private -> digest & rights verified -> Filament approval -> publish.
- اختبارات تكامل على webhook success/failed/duplicate/refund, price tamper, unauthorized download, expired link.
- لا يكتمل قبل موافقة مزود الدفع وتشغيل webhook في وضع التجربة ثم إثبات التسوية وفق شروط المزود.

## Phase 2 — نسخ مطبوعة وحزم
- تحديد Printer Profiles حسب trim/bleed/spine/paper/caliper، cover preview، print proof confirmation.
- طلبات منتجات ورقية مخزون/طلب مسبق مشروع + تكاليف شحن وتتبّع وعناوين وعوائد.
- حزم أجزاء ولغات، كوبونات محدودة ومقننة، معالجة refund/license revocation المتناسبة مع القانون.
- اختبار دقة وإنتاج PDF print وcover final عبر publisher-engine؛ لا تدّع طباعة جاهزة بلا فحص مطبعة.

## Phase 3 — التوسع العالمي
- ترجمة معتمدة لكل لغة وBook Edition مستقل، BCP47 + RTL/LTR.
- checkout localization والعملات حسب المزود، شروط بيع دولية وضرائب رقمية.
- تقارير مالية وحقوق مؤلفين ومترجمين، رسائل إطلاق، نسخ صوتية مستقلة.
- تنزيلات إقليمية وكاش/CDN لملفات مرخصة فقط؛ مفاتيح النسخ الخاصة لا تتسرب للعموم.

## مقاييس تشغيل تستحق المتابعة
- visit -> sample -> checkout -> paid entitlement
- payment failure by provider/currency
- download delivery success and expired grants
- refunds and support response
- editorial throughput per book/locale, translation freshness
- release preflight pass rate, asset approval

## شروط الإطلاق
- Book status approved/locked بالكامل، no editorial callouts.
- Written rights/translator rights/cover/image/font licenses.
- Cover approved and correct format validated.
- ISBN/metadata and imprint addressed for chosen channels.
- Payment onboarding/terms verified (لا تخمين مدخلات API).
- Terms/privacy/refund/consumer compliance reviewed.
- Digital fulfillment private path with test purchases and rollback.
- Backups, uptime/alerts, incident/reconciliation playbook.

## أول عرض للبيع
«جرس الرَّفْد» غير قابل للبيع الآن؛ وضعه draft/writing حتى يُكتب ويُحرَّر ويُعتمد ويجتاز بوابات الحقوق والجودة. يسمح المتجر بعينة مجانية فقط عند موافقة المؤلف.
