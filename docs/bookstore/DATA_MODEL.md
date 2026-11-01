# مخطط بيانات متجر الروايات المقترح — PostgreSQL + Eloquent

## الكيانات
- **users**: identity/email verified, auth session. لا مشاركة حسابات Fenix.
- **series**: id, slug unique, status, original_locale, visibility.
- **series_translations**: series_id+locale unique, title, synopsis, SEO.
- **books**: id, series_id FK, volume_number, slug unique, status, ordered release_date.
- **editions**: id, book_id FK, locale, format_set, version, source_commit, manuscript_digest, rights_confirmed, editorial_status, publication_status, published_at. Unique(book_id, locale, version).
- **edition_translations**: عند الحاجة لوصف الواجهة في locale مختلفة عن لغة الكتاب؛ لا تعني ترجمة نص الكتاب نفسه.
- **edition_assets**: edition_id, format enum epub/pdf-digital/pdf-print/cover/sample, storage_disk private, object_key, checksum, size, approval_state, content_type.
- **prices**: edition_id (or offer_id), currency, amount_minor, active_from, active_to; unique edition+currency+date window enforced by service layer/exclusion constraints when relevant.
- **orders**: id ULID, user_id FK nullable if guest supported, status, currency, subtotal_minor, tax_minor, total_minor, client_reference unique, placed_at.
- **order_items**: order_id, edition_id, price_snapshot_minor, currency, title_snapshot, quantity (ebooks 1).
- **payments**: order_id, provider enum, provider_reference unique per provider, state, gross_minor, currency, signature_verified_at, settled_at.
- **payment_events**: provider_event_id unique per provider, payload_hash, received_at, processed_at, state; event body redacted/encrypted as needed.
- **entitlements**: user_id, edition_id, order_item_id, activated_at, revoked_at; unique entitlement per purchase according to business model.
- **download_grants**: entitlement_id, asset_id, nonce hash, expires_at, used_at, max_uses; no public key.
- **audit_events**: actor, type, object, correlation_id, created_at, privacy-preserving metadata.
- **waitlist_subscribers**: verified consent and unsubscribe token, no purchase assumption.

## Relationships
- Series hasMany Books and Translations.
- Book belongsTo Series; hasMany Editions.
- Edition belongsTo Book; hasMany EditionAssets and Prices.
- Order hasMany OrderItems and Payments; belongsTo User when linked.
- Entitlement belongsTo User, Edition, OrderItem; hasMany DownloadGrants.

## Indexes/constraints
- unique(series.slug), unique(books.slug), unique(series_id, volume_number)
- unique(editions.book_id, locale, version); index(book_id, publication_status, published_at)
- index(orders.user_id, placed_at DESC); unique(orders.client_reference)
- unique(payments.provider, provider_reference)
- unique(payment_events.provider, provider_event_id)
- index(entitlements.user_id, edition_id, revoked_at)
- index(download_grants.entitlement_id, expires_at)
- partial index of active published editions for catalog; do not build query that scans all translations.
- enforce monetary amounts in integer minor units with currency precision rules from PSP; no float.
- private asset keys never returned to public API.

## Invariants
- Customer access must be authorized server-side, independent of UI button.
- Repeated verified webhooks do not create duplicate entitlement.
- Price/currency from server snapshot; never trust posted amount.
- Payment not confirmed by browser redirect alone.
- Refund/revocation policy explicit, audited, legally compliant.
- Unpublished edition cannot be checkout item.
- Source commit, SHA256 digest and approved visual Asset IDs must match release manifest.
- Any rights/author/cover approval failure keeps edition as private draft.

## Sample schema detail (implementation guidance)
Orders are immutable for price/item snapshots after payment initiation. Reconciliation creates event records rather than overwriting history; use DB transaction and pessimistic lock around state transitions. Use queue retry and dead-letter handling for provider timeouts, with idempotent handlers. Only the payment adapter reads credentials from secure config/secrets. Never store raw card data.
