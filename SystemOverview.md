# Sistem Genel Bakış Dokümantasyonu

Bu dokümantasyon, mevcut sistemin tüm bileşenlerini, modellerini, controller'larını, API endpoint'lerini ve permission sistemini içerir.

**Son Güncelleme:** 2025-11-04 22:22

---

## 📋 İçindekiler

1. [Proje Yapısı](#proje-yapısı)
2. [Modeller](#modeller)
3. [Controller'lar](#controllerlar)
4. [API Endpoint'leri](#api-endpointleri)
5. [Permission Sistemi](#permission-sistemi)
6. [Rol Sistemi](#rol-sistemi)
7. [Observer'lar](#observerlar)
8. [Migration'lar](#migrationlar)
9. [Önemli Özellikler](#önemli-özellikler)

---

## 📁 Proje Yapısı

```
laravel/example-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Api/          # API Controller'ları (17 controller)
│   │   │       ├── ApiKeyController.php
│   │   │       ├── AuditController.php
│   │   │       ├── AuthController.php
│   │   │       ├── ContentController.php
│   │   │       ├── IntegrationController.php
│   │   │       ├── LocaleController.php
│   │   │       ├── NotificationController.php
│   │   │       ├── OrganizationController.php
│   │   │       ├── PublicUserController.php
│   │   │       ├── ReportController.php
│   │   │       ├── RolePermissionController.php
│   │   │       ├── SessionController.php
│   │   │       ├── SystemController.php
│   │   │       ├── TranslationController.php
│   │   │       ├── UserController.php
│   │   │       ├── UserDetailController.php
│   │   │       └── UserSettingController.php
│   │   └── Middleware/
│   │       ├── PermissionMiddleware.php
│   │       └── SetUserLocale.php
│   ├── Models/                # Eloquent Modeller (14 model)
│   │   ├── ApiKey.php
│   │   ├── AuditLog.php
│   │   ├── Content.php
│   │   ├── Integration.php
│   │   ├── Locale.php
│   │   ├── Notification.php
│   │   ├── Organization.php
│   │   ├── Permission.php
│   │   ├── Report.php
│   │   ├── Role.php
│   │   ├── Translation.php
│   │   ├── User.php
│   │   ├── UserDetail.php
│   │   └── UserSetting.php
│   ├── Observers/             # Model Observer'ları (9 observer)
│   │   ├── ApiKeyObserver.php
│   │   ├── ContentObserver.php
│   │   ├── IntegrationObserver.php
│   │   ├── LocaleObserver.php
│   │   ├── NotificationObserver.php
│   │   ├── OrganizationObserver.php
│   │   ├── ReportObserver.php
│   │   ├── UserObserver.php
│   │   └── UserSettingObserver.php
│   ├── Helpers/
│   │   ├── LocaleHelper.php   # Dil ve yerelleştirme helper'ı
│   │   └── TranslationHelper.php  # Çeviri yönetimi helper'ı
│   └── Traits/
│       └── HasPermissions.php  # Permission yönetimi trait'i
├── database/
│   ├── migrations/            # Veritabanı migration'ları
│   └── seeders/
│       ├── DatabaseSeeder.php # Ana seeder (tüm seeder'ları çağırır)
│       ├── LocaleSeeder.php   # Dil seeder'ı
│       ├── RoleSeeder.php     # Rol ve Permission seeder'ı
│       └── TranslationSeeder.php # Çeviri seeder'ı
├── routes/
│   ├── api.php                # Ana API route dosyası (tüm modül route'larını yükler)
│   └── api/                   # Modüler API route dosyaları (alfabetik sırada)
│       ├── api-keys.php
│       ├── audit.php
│       ├── auth.php
│       ├── content.php
│       ├── integrations.php
│       ├── locales.php
│       ├── misc.php
│       ├── notifications.php
│       ├── organizations.php
│       ├── rbac.php
│       ├── reports.php
│       ├── sessions.php
│       ├── system.php
│       ├── translations.php
│       ├── user.php
│       └── user-settings.php
└── docs/
    ├── ApiResources.md
    ├── DevelopmentGuide.md
    ├── SystemOverview.md
    └── SystemOverview-*.md    # Versiyonlanmış SystemOverview dosyaları
```

---

## 🗄️ Modeller

### 1. User
**Dosya:** `app/Models/User.php`

**Özellikler:**
- `SoftDeletes` trait
- `HasApiTokens` trait (Sanctum)
- `uuid` kolonu (string, 36) - Public identifier
- `is_active` boolean alanı
- `email`, `password` alanları
- `name` alanı kaldırıldı (UserDetail'e taşındı)

**İlişkiler:**
- `detail()` - HasOne (UserDetail)
- `setting()` - HasOne (UserSetting)
- `roles()` - BelongsToMany (Role)

**Özel Metodlar:**
- `hasPermission(string $permissionSlug): bool` - Permission kontrolü
- `getHighestPriorityRole(): ?Role` - En yüksek öncelikli rol
- `getHighestPriorityLevel(): int` - En yüksek öncelik seviyesi
- `canManageRole(Role $targetRole): bool` - Rol yönetebilme kontrolü
- `canAssignRoleToUser(User $targetUser, Role $targetRole): bool` - Rol atayabilme kontrolü
- `hasPermissionOrOwn(string $permission, ?User $resourceOwner = null): bool` - Genel veya `.own` permission kontrolü
- `canManageOwn(string $basePermission, ?User $resourceOwner = null): bool` - Kendi kaynağını yönetebilme
- `canManageResource(string $basePermission, ?User $resourceOwner = null): bool` - Kaynak yönetebilme kontrolü

**Sistem Rolleri:**
- `system.toor` ve `server.root` rolleri tüm permission kontrollerini otomatik bypass eder

---

### 2. UserDetail
**Dosya:** `app/Models/UserDetail.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key

**İlişkiler:**
- `user()` - BelongsTo (User)

**Not:** User oluşturulduğunda otomatik olarak boş bir UserDetail kaydı oluşturulur (Observer ile)

---

### 3. UserSetting
**Dosya:** `app/Models/UserSetting.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key (unique)
- `locale` alanı (string, 10) - Dil kodu (tr, en, de, fr, es, it, ru, ar, zh, ja)
- `timezone` alanı (string, 50) - Saat dilimi (Europe/Istanbul, vb.)
- `date_format` alanı (string, 20) - Tarih formatı (d/m/Y, Y-m-d, vb.)
- `time_format` alanı (string, 20) - Saat formatı (H:i, h:i A, vb.)
- `currency` alanı (string, 3, nullable) - Para birimi (TRY, USD, EUR, vb.)
- `country_code` alanı (string, 2, nullable) - Ülke kodu (TR, US, DE, vb.)
- `notifications_email`, `notifications_push`, `notifications_sms` boolean alanları
- `theme` alanı (string, 20) - Tema (light, dark, auto)
- `items_per_page` alanı (integer) - Sayfa başına öğe sayısı
- `custom_settings` alanı (JSON, nullable) - Özel ayarlar

**İlişkiler:**
- `user()` - BelongsTo (User)

**Özel Metodlar:**
- `getLocale(): string` - Kullanıcının dil tercihini döndürür
- `getTimezone(): string` - Kullanıcının saat dilimini döndürür
- `getDateFormat(): string` - Kullanıcının tarih formatını döndürür
- `getTimeFormat(): string` - Kullanıcının saat formatını döndürür
- `getCurrency(): ?string` - Kullanıcının para birimini döndürür
- `getCountryCode(): ?string` - Kullanıcının ülke kodunu döndürür

**Not:** User oluşturulduğunda otomatik olarak varsayılan değerlerle bir UserSetting kaydı oluşturulur (Observer ile)

---

### 4. Role
**Dosya:** `app/Models/Role.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `slug` alanı (namespace'li: `system.toor`, `mgmt.superadmin`, vb.)
- `priority` alanı (integer) - Düşük sayı = Yüksek yetki

**İlişkiler:**
- `permissions()` - BelongsToMany (Permission)
- `users()` - BelongsToMany (User)

**Özel Metodlar:**
- `isSystemRole(): bool` - Sistem rolü kontrolü (`system.toor`, `server.root`)
- `hasHigherPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması
- `hasLowerPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması
- `hasHigherOrEqualPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması

**Rol Slug'ları:**
- `system.toor` (priority: 1) - Sistem Seviyesinde erişim
- `server.root` (priority: 2) - Sunucu Seviyesinde Erişim
- `mgmt.superadmin` (priority: 10) - Tam Yetkili Erişim
- `mgmt.admin` (priority: 20) - Yönetici erişimi
- `mgmt.moderator` (priority: 30) - Moderasyon
- `mgmt.editor` (priority: 40) - Editör
- `mgmt.user` (priority: 50) - Kullanıcı
- `mgmt.anonymous` (priority: 60) - Anonim

---

### 5. Permission
**Dosya:** `app/Models/Permission.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `slug` alanı (örn: `users.view`, `content.create.own`)
- `name` alanı (Türkçe açıklama)

**İlişkiler:**
- `roles()` - BelongsToMany (Role)

**Not:** Permission'lar controller'larda tanımlanır ve `RoleSeeder` tarafından otomatik toplanır

---

### 6. Organization
**Dosya:** `app/Models/Organization.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `is_active` boolean alanı
- Kapsamlı alanlar: `organization_code`, `organization_name`, `legal_name`, `tax_number`, vb.

**İlişkiler:**
- Yok (şimdilik)

---

### 7. Content
**Dosya:** `app/Models/Content.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `slug` alanı (otomatik oluşturulur)
- `type` enum: `post`, `page`, `article`, `news`, `document`
- `status` enum: `draft`, `published`, `archived`, `pending`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 8. Report
**Dosya:** `app/Models/Report.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `report_type` alanı
- `status` enum: `pending`, `generating`, `completed`, `failed`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 9. AuditLog
**Dosya:** `app/Models/AuditLog.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key (nullable)
- `action` alanı
- `model_type` ve `model_id` alanları (polymorphic)
- `ip_address`, `user_agent` alanları
- `old_values`, `new_values` JSON alanları

**İlişkiler:**
- `user()` - BelongsTo (User)
- `model()` - MorphTo (polymorphic)

---

### 10. Notification
**Dosya:** `app/Models/Notification.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `sender_id` foreign key (nullable)
- `is_active` boolean alanı
- `is_read` boolean alanı
- `type` alanı: `info`, `success`, `warning`, `error`, `system`

**İlişkiler:**
- `user()` - BelongsTo (User)
- `sender()` - BelongsTo (User)

---

### 11. ApiKey
**Dosya:** `app/Models/ApiKey.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `key` alanı (hash'lenmiş, 64 karakter)
- `key_prefix` alanı (ilk 8 karakter)
- `expires_at` timestamp (nullable)

**İlişkiler:**
- `user()` - BelongsTo (User)

**Not:** API anahtarı oluşturulduğunda otomatik olarak hash'lenir ve key_prefix kaydedilir

---

### 12. Integration
**Dosya:** `app/Models/Integration.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `type` alanı: `webhook`, `oauth`, `api`, `custom`
- `provider` alanı (nullable): `google`, `github`, `slack`, vb.
- `status` enum: `active`, `inactive`, `error`, `pending`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 13. Translation
**Dosya:** `app/Models/Translation.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `key` alanı (string, 255) - Çeviri anahtarı (örn: `pages.home`, `menu.account`)
- `locale` alanı (string, 10) - Dil kodu (tr, en, de, fr, vb.)
- `value` alanı (text) - Çeviri metni
- `group` alanı (string, 50, nullable) - Grup (pages, menu, buttons, messages, vb.)
- `description` alanı (text, nullable) - Açıklama

**İlişkiler:**
- Yok

**Özel Metodlar:**
- `scopeActive()` - Aktif çeviriler
- `scopeLocale()` - Belirli bir dil için çeviriler
- `scopeGroup()` - Belirli bir grup için çeviriler
- `scopeKey()` - Belirli bir key için çeviriler

**Not:** Her key + locale kombinasyonu için tek bir çeviri kaydı olmalı (unique constraint)

---

### 14. Locale
**Dosya:** `app/Models/Locale.php`

**Özellikler:**
- `HasFactory` trait
- `code` alanı (string, 10, unique) - Dil kodu (tr, en, de, fr, vb.)
- `name` alanı (string) - Dil adı (İngilizce)
- `native_name` alanı (string) - Yerel dil adı
- `sort_order` alanı (integer, default: 0) - Sıralama
- `is_active` boolean alanı

**İlişkiler:**
- Yok

**Özel Metodlar:**
- `scopeActive()` - Aktif diller
- `getActiveCodes(): array` - Aktif dil kodlarını döndürür
- `getAllCodes(): array` - Tüm dil kodlarını döndürür

**Not:** Locale bilgileri veritabanında saklanır ve `LocaleHelper::getSupportedLocaleCodes()` ile dinamik olarak çekilir.

---

## 🎮 Controller'lar

Tüm controller'lar `app/Http/Controllers/Api/` klasöründe bulunur ve `HasPermissions` trait'ini kullanır.

### 1. AuthController
**Dosya:** `app/Http/Controllers/Api/AuthController.php`

**Metodlar:**
- `login(Request $request)` - Kullanıcı girişi
- `register(Request $request)` - Kullanıcı kaydı
  - İlk kullanıcı `mgmt.superadmin` rolü alır
  - Diğer kullanıcılar `mgmt.user` rolü alır
- `me(Request $request)` - Mevcut kullanıcı bilgileri
- `logout(Request $request)` - Oturum kapatma

**Permission:** Yok (public endpoint'ler)

---

### 2. UserController
**Dosya:** `app/Http/Controllers/Api/UserController.php`

**Metodlar:**
- `index(Request $request)` - Kullanıcı listesi
- `show(Request $request, User $user)` - Kullanıcı detayı
- `store(Request $request)` - Yeni kullanıcı oluşturma
- `update(Request $request, User $user)` - Kullanıcı güncelleme
- `destroy(Request $request, User $user)` - Kullanıcı silme
- `restore(Request $request, string $uuid)` - Kullanıcı geri yükleme

**Permission'lar:**
- `users.view` / `users.view.own`
- `users.create`
- `users.update` / `users.update.own`
- `users.delete` / `users.delete.own`
- `users.restore` / `users.restore.own`
- `users.export` / `users.export.own`
- `users.import`
- `users.manage.roles` / `users.manage.roles.own`
- `users.manage.status` / `users.manage.status.own`

---

### 3. UserDetailController
**Dosya:** `app/Http/Controllers/Api/UserDetailController.php`

**Metodlar:**
- `show(Request $request)` - Kullanıcı detayları görüntüleme
- `update(Request $request)` - Kullanıcı detayları güncelleme

**Permission'lar:**
- `user.details.view` / `user.details.view.own`
- `user.details.create` / `user.details.create.own`
- `user.details.update` / `user.details.update.own`
- `user.details.delete` / `user.details.delete.own`
- `user.details.restore` / `user.details.restore.own`

---

### 4. UserSettingController
**Dosya:** `app/Http/Controllers/Api/UserSettingController.php`

**Metodlar:**
- `show(Request $request)` - Kullanıcı ayarlarını görüntüleme
- `update(Request $request)` - Kullanıcı ayarlarını güncelleme
- `destroy(Request $request)` - Kullanıcı ayarlarını silme (soft delete)
- `restore(Request $request, string $uuid)` - Kullanıcı ayarlarını geri yükleme

**Permission'lar:**
- `user.settings.view` / `user.settings.view.own`
- `user.settings.update` / `user.settings.update.own`
- `user.settings.delete` / `user.settings.delete.own`
- `user.settings.restore` / `user.settings.restore.own`

---

### 5. PublicUserController
**Dosya:** `app/Http/Controllers/Api/PublicUserController.php`

**Metodlar:**
- `show(User $user)` - Public kullanıcı profili (UUID ile)

**Permission:** Yok (public endpoint)

---

### 6. RolePermissionController
**Dosya:** `app/Http/Controllers/Api/RolePermissionController.php`

**Metodlar:**
- `createRole(Request $request)` - Rol oluşturma
- `listRoles(Request $request)` - Rol listesi
- `updateRole(Request $request, int $id)` - Rol güncelleme
- `deleteRole(int $id)` - Rol silme
- `restoreRole(Request $request, int $id)` - Rol geri yükleme
- `createPermission(Request $request)` - Permission oluşturma
- `listPermissions(Request $request)` - Permission listesi
- `updatePermission(Request $request, int $id)` - Permission güncelleme
- `deletePermission(int $id)` - Permission silme
- `restorePermission(Request $request, int $id)` - Permission geri yükleme
- `assignPermissionToRole(Request $request)` - Rol'e permission atama
- `removePermissionFromRole(Request $request)` - Rol'den permission kaldırma
- `assignRoleToUser(Request $request)` - Kullanıcıya rol atama
- `removeRoleFromUser(Request $request)` - Kullanıcıdan rol kaldırma

**Permission'lar:**
- `roles.view`, `roles.create`, `roles.update`, `roles.delete`, `roles.restore`, `roles.assign`, `roles.revoke`
- `permissions.view`, `permissions.create`, `permissions.update`, `permissions.delete`, `permissions.restore`, `permissions.assign`

**Özel Kontroller:**
- Sistem rolleri (`system.toor`, `server.root`) için gizli anahtar zorunlu
- Rol hiyerarşisi kontrolü (kullanıcı sadece kendi seviyesinden düşük rolleri yönetebilir)

---

### 7. OrganizationController
**Dosya:** `app/Http/Controllers/Api/OrganizationController.php`

**Metodlar:**
- `index(Request $request)` - Organizasyon listesi
- `show(Request $request, Organization $organization)` - Organizasyon detayı
- `store(Request $request)` - Yeni organizasyon oluşturma
- `update(Request $request, Organization $organization)` - Organizasyon güncelleme
- `destroy(Request $request, Organization $organization)` - Organizasyon silme
- `restore(Request $request, string $uuid)` - Organizasyon geri yükleme

**Permission'lar:**
- `organizations.view` / `organizations.view.own`
- `organizations.create` / `organizations.create.own`
- `organizations.update` / `organizations.update.own`
- `organizations.delete` / `organizations.delete.own`
- `organizations.restore` / `organizations.restore.own`
- `organizations.export` / `organizations.export.own`
- `organizations.import`
- `organizations.manage.members` / `organizations.manage.members.own`

---

### 8. ContentController
**Dosya:** `app/Http/Controllers/Api/ContentController.php`

**Metodlar:**
- `index(Request $request)` - İçerik listesi
- `show(Request $request, Content $content)` - İçerik detayı
- `store(Request $request)` - Yeni içerik oluşturma
- `update(Request $request, Content $content)` - İçerik güncelleme
- `destroy(Request $request, Content $content)` - İçerik silme
- `restore(Request $request, string $uuid)` - İçerik geri yükleme
- `publish(Request $request, Content $content)` - İçerik yayınlama
- `unpublish(Request $request, Content $content)` - İçerik yayından kaldırma

**Permission'lar:**
- `content.view` / `content.view.own`
- `content.create` / `content.create.own`
- `content.update` / `content.update.own`
- `content.delete` / `content.delete.own`
- `content.restore` / `content.restore.own`
- `content.publish` / `content.publish.own`
- `content.unpublish` / `content.unpublish.own`
- `content.moderate`
- `content.edit` / `content.edit.own`
- `content.export` / `content.export.own`

---

### 9. ReportController
**Dosya:** `app/Http/Controllers/Api/ReportController.php`

**Metodlar:**
- `index(Request $request)` - Rapor listesi
- `show(Request $request, Report $report)` - Rapor detayı
- `store(Request $request)` - Yeni rapor oluşturma
- `update(Request $request, Report $report)` - Rapor güncelleme
- `destroy(Request $request, Report $report)` - Rapor silme

**Permission'lar:**
- `reports.view` / `reports.view.own`
- `reports.generate` / `reports.generate.own`
- `reports.export` / `reports.export.own`
- `reports.delete` / `reports.delete.own`

---

### 10. AuditController
**Dosya:** `app/Http/Controllers/Api/AuditController.php`

**Metodlar:**
- `index(Request $request)` - Denetim log listesi
- `show(Request $request, AuditLog $auditLog)` - Denetim log detayı
- `destroy(Request $request, AuditLog $auditLog)` - Denetim log silme

**Permission'lar:**
- `audit.view` / `audit.view.own`
- `audit.export` / `audit.export.own`
- `audit.delete` / `audit.delete.own`

---

### 11. NotificationController
**Dosya:** `app/Http/Controllers/Api/NotificationController.php`

**Metodlar:**
- `index(Request $request)` - Bildirim listesi
- `show(Request $request, Notification $notification)` - Bildirim detayı
- `store(Request $request)` - Yeni bildirim oluşturma
- `update(Request $request, Notification $notification)` - Bildirim güncelleme
- `destroy(Request $request, Notification $notification)` - Bildirim silme
- `markAsRead(Request $request, Notification $notification)` - Bildirim okundu işaretleme

**Permission'lar:**
- `notifications.send` / `notifications.send.own`
- `notifications.manage` / `notifications.manage.own`
- `notifications.view` / `notifications.view.own`
- `notifications.update` / `notifications.update.own`
- `notifications.delete` / `notifications.delete.own`
- `notifications.mark.read` / `notifications.mark.read.own`

---

### 12. ApiKeyController
**Dosya:** `app/Http/Controllers/Api/ApiKeyController.php`

**Metodlar:**
- `index(Request $request)` - API anahtarı listesi
- `show(Request $request, ApiKey $apiKey)` - API anahtarı detayı
- `store(Request $request)` - Yeni API anahtarı oluşturma
- `update(Request $request, ApiKey $apiKey)` - API anahtarı güncelleme
- `revoke(Request $request, ApiKey $apiKey)` - API anahtarı iptal etme

**Permission'lar:**
- `api.keys.manage` / `api.keys.manage.own`
- `api.keys.view` / `api.keys.view.own`
- `api.keys.create` / `api.keys.create.own`
- `api.keys.update` / `api.keys.update.own`
- `api.keys.revoke` / `api.keys.revoke.own`

---

### 13. IntegrationController
**Dosya:** `app/Http/Controllers/Api/IntegrationController.php`

**Metodlar:**
- `index(Request $request)` - Entegrasyon listesi
- `show(Request $request, Integration $integration)` - Entegrasyon detayı
- `store(Request $request)` - Yeni entegrasyon oluşturma
- `update(Request $request, Integration $integration)` - Entegrasyon güncelleme
- `destroy(Request $request, Integration $integration)` - Entegrasyon silme

**Permission'lar:**
- `integrations.manage` / `integrations.manage.own`
- `integrations.view` / `integrations.view.own`
- `integrations.create` / `integrations.create.own`

---

### 14. SystemController
**Dosya:** `app/Http/Controllers/Api/SystemController.php`

**Metodlar:**
- `getSettings(Request $request)` - Sistem ayarları
- `getLogs(Request $request)` - Sistem logları
- `getMetrics(Request $request)` - Sistem metrikleri

**Permission'lar:**
- `system.settings`
- `system.logs`
- `system.monitor`
- `system.backups` (RoleSeeder'da tanımlı)
- `system.maintenance` (RoleSeeder'da tanımlı)

---

### 15. SessionController
**Dosya:** `app/Http/Controllers/Api/SessionController.php`

**Metodlar:**
- `index(Request $request)` - Oturum listesi (Sanctum token'ları)
- `show(Request $request)` - Mevcut oturum
- `revoke(Request $request, $tokenId)` - Oturum iptal etme
- `revokeAll(Request $request)` - Tüm oturumları iptal etme (mevcut hariç)

**Permission'lar:**
- `sessions.view` / `sessions.view.own`
- `sessions.manage` / `sessions.manage.own`
- `sessions.create` / `sessions.create.own`
- `sessions.revoke` / `sessions.revoke.own`

---

### 16. TranslationController
**Dosya:** `app/Http/Controllers/Api/TranslationController.php`

**Metodlar:**
- `index(Request $request)` - Çeviri listesi (arama, filtreleme, pagination)
- `show(Request $request, Translation $translation)` - Çeviri detayı
- `get(Request $request, string $key)` - Çeviri anahtarına göre çeviri metnini döndürür (Public)
- `getGroup(Request $request, string $group)` - Bir grup için tüm çevirileri döndürür (Public)
- `getAll(Request $request)` - Tüm çevirileri döndürür (locale'e göre) (Public)
- `store(Request $request)` - Yeni çeviri oluşturma (values formatı desteklenir)
- `bulkStore(Request $request)` - Toplu çeviri oluşturma
- `update(Request $request, ?Translation $translation)` - Çeviri güncelleme (values formatı desteklenir)
- `bulkUpdate(Request $request)` - Toplu çeviri güncelleme (upsert)
- `destroy(Request $request, Translation $translation)` - Çeviri silme
- `bulkDelete(Request $request)` - Toplu çeviri silme
- `restore(Request $request, int $id)` - Çeviri geri yükleme

**Permission'lar:**
- `translations.view`
- `translations.create`
- `translations.update`
- `translations.delete`
- `translations.restore`
- `translations.manage`

**Not:** Locale validasyonu `LocaleHelper::getSupportedLocaleCodes()` ile veritabanından dinamik olarak yapılır.

---

### 17. LocaleController
**Dosya:** `app/Http/Controllers/Api/LocaleController.php`

**Metodlar:**
- `index(Request $request)` - Dil listesi (arama, filtreleme, pagination)
- `show(Locale $locale)` - Dil detayı (Public)
- `store(Request $request)` - Yeni dil oluşturma
- `update(Request $request, Locale $locale)` - Dil güncelleme
- `destroy(Request $request, Locale $locale)` - Dil silme (soft delete)
- `restore(Request $request, int $id)` - Dil geri yükleme

**Permission'lar:**
- `locales.view`
- `locales.create`
- `locales.update`
- `locales.delete`
- `locales.restore`
- `locales.manage`

---

## 🌐 API Endpoint'leri

Tüm endpoint'ler `/api/v1/` prefix'i altında çalışır.

### Authentication Endpoints
**Dosya:** `routes/api/auth.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| POST | `/login` | AuthController@login | Public | Kullanıcı girişi |
| POST | `/register` | AuthController@register | Public | Kullanıcı kaydı |
| GET | `/me` | AuthController@me | auth:sanctum | Mevcut kullanıcı bilgileri |
| POST | `/logout` | AuthController@logout | auth:sanctum | Oturum kapatma |

---

### User Endpoints
**Dosya:** `routes/api/user.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/user/details` | UserDetailController@show | `user.details.view` / `user.details.view.own` | Kullanıcı detayları |
| PUT | `/user/details` | UserDetailController@update | `user.details.update` / `user.details.update.own` | Kullanıcı detayları güncelleme |
| GET | `/user/settings` | UserSettingController@show | `user.settings.view` / `user.settings.view.own` | Kullanıcı ayarları |
| PUT | `/user/settings` | UserSettingController@update | `user.settings.update` / `user.settings.update.own` | Kullanıcı ayarları güncelleme |
| PATCH | `/user/settings` | UserSettingController@update | `user.settings.update` / `user.settings.update.own` | Kullanıcı ayarları güncelleme |
| DELETE | `/user/settings` | UserSettingController@destroy | `user.settings.delete` / `user.settings.delete.own` | Kullanıcı ayarları silme |
| POST | `/user/settings/{uuid}/restore` | UserSettingController@restore | `user.settings.restore` / `user.settings.restore.own` | Kullanıcı ayarları geri yükleme |
| GET | `/users` | UserController@index | `users.view` / `users.view.own` | Kullanıcı listesi |
| POST | `/users` | UserController@store | `users.create` | Yeni kullanıcı oluşturma |
| GET | `/users/{user}` | UserController@show | `users.view` / `users.view.own` | Kullanıcı detayı |
| PUT | `/users/{user}` | UserController@update | `users.update` / `users.update.own` | Kullanıcı güncelleme |
| PATCH | `/users/{user}` | UserController@update | `users.update` / `users.update.own` | Kullanıcı güncelleme |
| DELETE | `/users/{user}` | UserController@destroy | `users.delete` / `users.delete.own` | Kullanıcı silme |
| POST | `/users/{uuid}/restore` | UserController@restore | `users.restore` / `users.restore.own` | Kullanıcı geri yükleme |
| GET | `/users/{user}` | PublicUserController@show | Public | Public kullanıcı profili (UUID ile) |

---

### RBAC Endpoints
**Dosya:** `routes/api/rbac.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/roles` | RolePermissionController@listRoles | `admin.manage` | Rol listesi |
| POST | `/roles` | RolePermissionController@createRole | `admin.manage` | Rol oluşturma |
| PATCH | `/roles/{id}` | RolePermissionController@updateRole | `admin.manage` | Rol güncelleme |
| PUT | `/roles/{id}` | RolePermissionController@updateRole | `admin.manage` | Rol güncelleme |
| DELETE | `/roles/{id}` | RolePermissionController@deleteRole | `admin.manage` | Rol silme |
| POST | `/roles/{id}/restore` | RolePermissionController@restoreRole | `admin.manage` | Rol geri yükleme |
| GET | `/permissions` | RolePermissionController@listPermissions | `admin.manage` | Permission listesi |
| POST | `/permissions` | RolePermissionController@createPermission | `admin.manage` | Permission oluşturma |
| PATCH | `/permissions/{id}` | RolePermissionController@updatePermission | `admin.manage` | Permission güncelleme |
| PUT | `/permissions/{id}` | RolePermissionController@updatePermission | `admin.manage` | Permission güncelleme |
| DELETE | `/permissions/{id}` | RolePermissionController@deletePermission | `admin.manage` | Permission silme |
| POST | `/permissions/{id}/restore` | RolePermissionController@restorePermission | `admin.manage` | Permission geri yükleme |
| POST | `/roles/assign-permission` | RolePermissionController@assignPermissionToRole | `admin.manage` | Rol'e permission atama |
| POST | `/roles/remove-permission` | RolePermissionController@removePermissionFromRole | `admin.manage` | Rol'den permission kaldırma |
| POST | `/users/assign-role` | RolePermissionController@assignRoleToUser | `admin.manage` | Kullanıcıya rol atama |
| POST | `/users/remove-role` | RolePermissionController@removeRoleFromUser | `admin.manage` | Kullanıcıdan rol kaldırma |

**Özel Kontroller:**
- Sistem rolleri (`system.toor`, `server.root`) için gizli anahtar zorunlu
- Rol hiyerarşisi kontrolü

---

### Organization Endpoints
**Dosya:** `routes/api/organizations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/organizations/{organization}` | OrganizationController@show | Public | Public organizasyon detayı |
| GET | `/organizations` | OrganizationController@index | `organizations.view` / `organizations.view.own` | Organizasyon listesi |
| POST | `/organizations` | OrganizationController@store | `organizations.create` / `organizations.create.own` | Organizasyon oluşturma |
| PUT | `/organizations/{organization}` | OrganizationController@update | `organizations.update` / `organizations.update.own` | Organizasyon güncelleme |
| PATCH | `/organizations/{organization}` | OrganizationController@update | `organizations.update` / `organizations.update.own` | Organizasyon güncelleme |
| DELETE | `/organizations/{organization}` | OrganizationController@destroy | `organizations.delete` / `organizations.delete.own` | Organizasyon silme |
| POST | `/organizations/{uuid}/restore` | OrganizationController@restore | `organizations.restore` / `organizations.restore.own` | Organizasyon geri yükleme |

---

### Content Endpoints
**Dosya:** `routes/api/content.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/contents/{content}` | ContentController@show | Public | Public içerik görüntüleme |
| GET | `/contents` | ContentController@index | `content.view` / `content.view.own` | İçerik listesi |
| POST | `/contents` | ContentController@store | `content.create` / `content.create.own` | İçerik oluşturma |
| PUT | `/contents/{content}` | ContentController@update | `content.update` / `content.update.own` | İçerik güncelleme |
| PATCH | `/contents/{content}` | ContentController@update | `content.update` / `content.update.own` | İçerik güncelleme |
| DELETE | `/contents/{content}` | ContentController@destroy | `content.delete` / `content.delete.own` | İçerik silme |
| POST | `/contents/{uuid}/restore` | ContentController@restore | `content.restore` / `content.restore.own` | İçerik geri yükleme |
| POST | `/contents/{content}/publish` | ContentController@publish | `content.publish` / `content.publish.own` | İçerik yayınlama |
| POST | `/contents/{content}/unpublish` | ContentController@unpublish | `content.unpublish` / `content.unpublish.own` | İçerik yayından kaldırma |

---

### Report Endpoints
**Dosya:** `routes/api/reports.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/reports` | ReportController@index | `reports.view` / `reports.view.own` | Rapor listesi |
| GET | `/reports/{report}` | ReportController@show | `reports.view` / `reports.view.own` | Rapor detayı |
| POST | `/reports` | ReportController@store | `reports.generate` / `reports.generate.own` | Rapor oluşturma |
| PUT | `/reports/{report}` | ReportController@update | `reports.generate` / `reports.generate.own` | Rapor güncelleme |
| PATCH | `/reports/{report}` | ReportController@update | `reports.generate` / `reports.generate.own` | Rapor güncelleme |
| DELETE | `/reports/{report}` | ReportController@destroy | `reports.delete` / `reports.delete.own` | Rapor silme |

---

### Audit Endpoints
**Dosya:** `routes/api/audit.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/audit-logs` | AuditController@index | `audit.view` / `audit.view.own` | Denetim log listesi |
| GET | `/audit-logs/{auditLog}` | AuditController@show | `audit.view` / `audit.view.own` | Denetim log detayı |
| DELETE | `/audit-logs/{auditLog}` | AuditController@destroy | `audit.delete` / `audit.delete.own` | Denetim log silme |

---

### Notification Endpoints
**Dosya:** `routes/api/notifications.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/notifications` | NotificationController@index | `notifications.view` / `notifications.view.own` | Bildirim listesi |
| GET | `/notifications/{notification}` | NotificationController@show | `notifications.view` / `notifications.view.own` | Bildirim detayı |
| POST | `/notifications` | NotificationController@store | `notifications.send` / `notifications.send.own` | Bildirim oluşturma |
| PUT | `/notifications/{notification}` | NotificationController@update | `notifications.update` / `notifications.update.own` | Bildirim güncelleme |
| PATCH | `/notifications/{notification}` | NotificationController@update | `notifications.update` / `notifications.update.own` | Bildirim güncelleme |
| DELETE | `/notifications/{notification}` | NotificationController@destroy | `notifications.delete` / `notifications.delete.own` | Bildirim silme |
| POST | `/notifications/{notification}/mark-read` | NotificationController@markAsRead | `notifications.mark.read` / `notifications.mark.read.own` | Bildirim okundu işaretleme |

---

### API Key Endpoints
**Dosya:** `routes/api/api-keys.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/api-keys` | ApiKeyController@index | `api.keys.view` / `api.keys.view.own` | API anahtarı listesi |
| GET | `/api-keys/{apiKey}` | ApiKeyController@show | `api.keys.view` / `api.keys.view.own` | API anahtarı detayı |
| POST | `/api-keys` | ApiKeyController@store | `api.keys.create` / `api.keys.create.own` | API anahtarı oluşturma |
| PUT | `/api-keys/{apiKey}` | ApiKeyController@update | `api.keys.update` / `api.keys.update.own` | API anahtarı güncelleme |
| PATCH | `/api-keys/{apiKey}` | ApiKeyController@update | `api.keys.update` / `api.keys.update.own` | API anahtarı güncelleme |
| POST | `/api-keys/{apiKey}/revoke` | ApiKeyController@revoke | `api.keys.revoke` / `api.keys.revoke.own` | API anahtarı iptal etme |

---

### Integration Endpoints
**Dosya:** `routes/api/integrations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/integrations` | IntegrationController@index | `integrations.view` / `integrations.view.own` | Entegrasyon listesi |
| GET | `/integrations/{integration}` | IntegrationController@show | `integrations.view` / `integrations.view.own` | Entegrasyon detayı |
| POST | `/integrations` | IntegrationController@store | `integrations.create` / `integrations.create.own` | Entegrasyon oluşturma |
| PUT | `/integrations/{integration}` | IntegrationController@update | `integrations.manage` / `integrations.manage.own` | Entegrasyon güncelleme |
| PATCH | `/integrations/{integration}` | IntegrationController@update | `integrations.manage` / `integrations.manage.own` | Entegrasyon güncelleme |
| DELETE | `/integrations/{integration}` | IntegrationController@destroy | `integrations.manage` / `integrations.manage.own` | Entegrasyon silme |

---

### System Endpoints
**Dosya:** `routes/api/system.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/system/settings` | SystemController@getSettings | `system.settings` | Sistem ayarları |
| GET | `/system/logs` | SystemController@getLogs | `system.logs` | Sistem logları |
| GET | `/system/metrics` | SystemController@getMetrics | `system.monitor` | Sistem metrikleri |

---

### Session Endpoints
**Dosya:** `routes/api/sessions.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/sessions` | SessionController@index | `sessions.view` / `sessions.view.own` | Oturum listesi |
| GET | `/sessions/current` | SessionController@show | `sessions.view` / `sessions.view.own` | Mevcut oturum |
| POST | `/sessions/{tokenId}/revoke` | SessionController@revoke | `sessions.revoke` / `sessions.revoke.own` | Oturum iptal etme |
| POST | `/sessions/revoke-all` | SessionController@revokeAll | `sessions.revoke` / `sessions.revoke.own` | Tüm oturumları iptal etme |

---

### Translation Endpoints
**Dosya:** `routes/api/translations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/translations` | TranslationController@getAll | Public | Tüm çevirileri alma (locale'e göre) |
| GET | `/translations/{key}` | TranslationController@get | Public | Çeviri metnini alma (key ile) |
| GET | `/translations/group/{group}` | TranslationController@getGroup | Public | Grup çevirilerini alma |
| GET | `/admin/translations` | TranslationController@index | `translations.view` | Çeviri listesi (arama, filtreleme) |
| GET | `/admin/translations/{translation}` | TranslationController@show | `translations.view` | Çeviri detayı |
| POST | `/admin/translations` | TranslationController@store | `translations.create` | Çeviri oluşturma (tek veya toplu) |
| POST | `/admin/translations/bulk` | TranslationController@bulkStore | `translations.create` | Toplu çeviri oluşturma |
| PUT | `/admin/translations/bulk` | TranslationController@bulkUpdate | `translations.update` | Toplu çeviri güncelleme (upsert) |
| PATCH | `/admin/translations/bulk` | TranslationController@bulkUpdate | `translations.update` | Toplu çeviri güncelleme (upsert) |
| DELETE | `/admin/translations/bulk` | TranslationController@bulkDelete | `translations.delete` | Toplu çeviri silme |
| PUT | `/admin/translations/{translation}` | TranslationController@update | `translations.update` | Çeviri güncelleme |
| PATCH | `/admin/translations/{translation}` | TranslationController@update | `translations.update` | Çeviri güncelleme |
| DELETE | `/admin/translations/{translation}` | TranslationController@destroy | `translations.delete` | Çeviri silme |
| POST | `/admin/translations/{id}/restore` | TranslationController@restore | `translations.restore` | Çeviri geri yükleme |

---

### Locale Endpoints
**Dosya:** `routes/api/locales.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/locales/{locale}` | LocaleController@show | Public | Dil detayı |
| GET | `/admin/locales` | LocaleController@index | `locales.view` | Dil listesi (arama, filtreleme) |
| POST | `/admin/locales` | LocaleController@store | `locales.create` | Yeni dil oluşturma |
| PUT | `/admin/locales/{locale}` | LocaleController@update | `locales.update` | Dil güncelleme |
| PATCH | `/admin/locales/{locale}` | LocaleController@update | `locales.update` | Dil güncelleme |
| DELETE | `/admin/locales/{locale}` | LocaleController@destroy | `locales.delete` | Dil silme |
| POST | `/admin/locales/{id}/restore` | LocaleController@restore | `locales.restore` | Dil geri yükleme |

---

### Misc Endpoints
**Dosya:** `routes/api/misc.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/ping` | Closure | Public | Sağlık kontrolü |

---

## 🔐 Permission Sistemi

### Permission Tanımlama
Permission'lar controller'larda `getPermissions()` metodu ile tanımlanır:

```php
public static function getPermissions(): array
{
    return [
        ['slug' => 'module.view', 'name' => 'Modül Görüntüleme'],
        ['slug' => 'module.view.own', 'name' => 'Kendi Modülünü Görüntüleme'],
        // ...
    ];
}
```

### Permission Kontrolü
Controller metodları içinde permission kontrolü yapılır:

```php
// Genel permission veya .own permission kontrolü
if (! $user->hasPermissionOrOwn('module.view')) {
    return response()->json(['message' => 'Forbidden'], 403);
}

// Kaynak sahibi kontrolü ile permission kontrolü
if (! $user->canManageResource('module.update', $resource->user)) {
    return response()->json(['message' => 'Forbidden'], 403);
}
```

### Permission Kategorileri

#### 1. Sistem Yönetimi
- `admin.manage` - Admin Yönetimi - Tam Yetki
- `system.settings` - Sistem Ayarları Yönetimi
- `system.logs` - Sistem Loglarını Görüntüleme
- `system.backups` - Yedekleme Yönetimi
- `system.monitor` - Sistem İzleme ve Metrikler
- `system.maintenance` - Bakım Modu Yönetimi

#### 2. Kullanıcı Yönetimi
- `users.manage` - Kullanıcı Yönetimi - Tam Yetki
- `users.view` / `users.view.own`
- `users.create`
- `users.update` / `users.update.own`
- `users.delete` / `users.delete.own`
- `users.restore` / `users.restore.own`
- `users.export` / `users.export.own`
- `users.import`
- `users.manage.roles` / `users.manage.roles.own`
- `users.manage.status` / `users.manage.status.own`

#### 3. Kullanıcı Detayları
- `user.details.view` / `user.details.view.own`
- `user.details.create` / `user.details.create.own`
- `user.details.update` / `user.details.update.own`
- `user.details.delete` / `user.details.delete.own`
- `user.details.restore` / `user.details.restore.own`

#### 4. Rol Yönetimi
- `roles.manage` - Rol Yönetimi - Tam Yetki
- `roles.view`
- `roles.create`
- `roles.update`
- `roles.delete`
- `roles.restore`
- `roles.assign`
- `roles.revoke`

#### 5. İzin Yönetimi
- `permissions.manage` - İzin Yönetimi - Tam Yetki
- `permissions.view`
- `permissions.create`
- `permissions.update`
- `permissions.delete`
- `permissions.restore`
- `permissions.assign`

#### 6. Organizasyon Yönetimi
- `organizations.manage` - Organizasyon Yönetimi - Tam Yetki
- `organizations.view` / `organizations.view.own`
- `organizations.create` / `organizations.create.own`
- `organizations.update` / `organizations.update.own`
- `organizations.delete` / `organizations.delete.own`
- `organizations.restore` / `organizations.restore.own`
- `organizations.export` / `organizations.export.own`
- `organizations.import`
- `organizations.manage.members` / `organizations.manage.members.own`

#### 7. İçerik Yönetimi
- `content.manage` - İçerik Yönetimi - Tam Yetki
- `content.view` / `content.view.own`
- `content.create` / `content.create.own`
- `content.update` / `content.update.own`
- `content.delete` / `content.delete.own`
- `content.restore` / `content.restore.own`
- `content.publish` / `content.publish.own`
- `content.unpublish` / `content.unpublish.own`
- `content.moderate`
- `content.edit` / `content.edit.own`
- `content.export` / `content.export.own`

#### 8. Raporlar
- `reports.view` / `reports.view.own`
- `reports.generate` / `reports.generate.own`
- `reports.export` / `reports.export.own`
- `reports.delete` / `reports.delete.own`

#### 9. Denetim ve Loglar
- `audit.view` / `audit.view.own`
- `audit.export` / `audit.export.own`
- `audit.delete` / `audit.delete.own`

#### 10. Oturum Yönetimi
- `sessions.view` / `sessions.view.own`
- `sessions.manage` / `sessions.manage.own`
- `sessions.create` / `sessions.create.own`
- `sessions.revoke` / `sessions.revoke.own`

#### 11. Bildirimler
- `notifications.send` / `notifications.send.own`
- `notifications.manage` / `notifications.manage.own`
- `notifications.view` / `notifications.view.own`
- `notifications.update` / `notifications.update.own`
- `notifications.delete` / `notifications.delete.own`
- `notifications.mark.read` / `notifications.mark.read.own`

#### 12. API ve Entegrasyonlar
- `api.keys.manage` / `api.keys.manage.own`
- `api.keys.view` / `api.keys.view.own`
- `api.keys.create` / `api.keys.create.own`
- `api.keys.update` / `api.keys.update.own`
- `api.keys.revoke` / `api.keys.revoke.own`
- `integrations.manage` / `integrations.manage.own`
- `integrations.view` / `integrations.view.own`
- `integrations.create` / `integrations.create.own`

#### 13. Çeviriler
- `translations.view`
- `translations.create`
- `translations.update`
- `translations.delete`
- `translations.restore`
- `translations.manage`

#### 14. Diller
- `locales.manage` - Dil Yönetimi - Tam Yetki
- `locales.view`
- `locales.create`
- `locales.update`
- `locales.delete`
- `locales.restore`

---

## 👥 Rol Sistemi

### Rol Hiyerarşisi

Rol hiyerarşisi `priority` alanı ile belirlenir. Düşük sayı = Yüksek yetki.

| Rol | Slug | Priority | Açıklama |
|-----|------|----------|----------|
| Toor | `system.toor` | 1 | Sistem Seviyesinde erişim (gizli anahtar ile) |
| Root | `server.root` | 2 | Sunucu Seviyesinde Erişim (gizli anahtar ile) |
| Super Admin | `mgmt.superadmin` | 10 | Tam Yetkili Erişim |
| Admin | `mgmt.admin` | 20 | Yönetici erişimi |
| Moderator | `mgmt.moderator` | 30 | Moderasyon |
| Editor | `mgmt.editor` | 40 | Editör |
| User | `mgmt.user` | 50 | Kullanıcı |
| Anonymous | `mgmt.anonymous` | 60 | Anonim |

### Sistem Rolleri

`system.toor` ve `server.root` rolleri:
- Tüm permission kontrollerini otomatik bypass eder
- Sadece gizli anahtar (`ROLE_SYSTEM_SECRET`) ile atanabilir
- Tüm permission'lara otomatik sahip olur
- Rol hiyerarşisi kontrolünden muaf

### Rol-Permission Atamaları

Rol-permission atamaları controller'larda `getDefaultRolePermissions()` metodu ile tanımlanır:

```php
public static function getDefaultRolePermissions(): array
{
    return [
        'mgmt.superadmin' => [
            'module.view', 'module.create', 'module.update', 'module.delete',
        ],
        'mgmt.user' => [
            'module.view.own', 'module.create.own', 'module.update.own',
        ],
    ];
}
```

### RoleSeeder

`RoleSeeder` otomatik olarak:
1. Controller'lardan permission'ları toplar (`collectAllPermissions()`)
2. Controller'lardan rol-permission atamalarını toplar (`collectAllRolePermissions()`)
3. Sistem rolleri için tüm permission'ları atar
4. Veritabanına kaydeder

---

## 👁️ Observer'lar

Observer'lar `app/Observers/` klasöründe bulunur ve `AppServiceProvider` içinde kaydedilir.

### Mevcut Observer'lar:
1. **UserObserver** - UUID oluşturma, UserDetail ve UserSetting otomatik oluşturma
2. **OrganizationObserver** - UUID oluşturma
3. **ContentObserver** - UUID oluşturma, slug otomatik oluşturma
4. **ReportObserver** - UUID oluşturma
5. **NotificationObserver** - UUID oluşturma
6. **ApiKeyObserver** - UUID oluşturma, API anahtarı hash'leme
7. **IntegrationObserver** - UUID oluşturma
8. **UserSettingObserver** - Ayarlar için event handler'ları
9. **LocaleObserver** - Locale oluşturma için event handler'ları

---

## 🗄️ Migration'lar

### Mevcut Tablolar:
1. **users** - Kullanıcılar (uuid, email, password, is_active, deleted_at)
2. **users_details** - Kullanıcı detayları (user_id, is_active, deleted_at)
3. **user_settings** - Kullanıcı ayarları (user_id, locale, timezone, date_format, time_format, currency, country_code, notifications, theme, items_per_page, custom_settings, is_active, deleted_at)
4. **user_role** - Roller (slug, name, priority, is_active, deleted_at)
5. **permissions** - İzinler (slug, name, is_active, deleted_at)
6. **user_roles** - Kullanıcı-Rol pivot tablosu
7. **role_permissions** - Rol-Permission pivot tablosu
8. **organizations** - Organizasyonlar (uuid, organization_code, organization_name, vb.)
9. **contents** - İçerikler (uuid, user_id, title, slug, content, type, status, vb.)
10. **reports** - Raporlar (uuid, user_id, title, report_type, status, vb.)
11. **audit_logs** - Denetim logları (user_id, action, model_type, model_id, vb.)
12. **notifications** - Bildirimler (uuid, user_id, sender_id, type, title, message, vb.)
13. **api_keys** - API anahtarları (uuid, user_id, name, key, key_prefix, vb.)
14. **integrations** - Entegrasyonlar (uuid, user_id, name, type, provider, vb.)
15. **translations** - Çeviriler (key, locale, value, group, description, is_active, deleted_at)
16. **locales** - Diller (code, name, native_name, sort_order, is_active, timestamps)

**Ortak Özellikler:**
- Tüm tablolarda `id` (primary key)
- Public erişim gereken tablolarda `uuid` (string, 36, unique)
- Tüm tablolarda `is_active` (boolean, default: true)
- Tüm tablolarda `deleted_at` (soft delete)
- Tüm tablolarda `created_at` ve `updated_at` (timestamps)

---

## ⚡ Önemli Özellikler

### 1. Sistem Rolleri Bypass
- `system.toor` ve `server.root` rolleri tüm permission kontrollerini otomatik bypass eder
- `User::hasPermission()` metodunda sistem rolleri kontrolü yapılır
- `PermissionMiddleware` içinde sistem rolleri bypass edilir

### 2. Permission Sistemi
- Permission'lar controller'larda tanımlanır
- Rol-permission atamaları controller'larda tanımlanır
- `RoleSeeder` otomatik olarak toplar ve veritabanına ekler
- `.own` permission'ları ile kullanıcı sadece kendi kaynaklarını yönetebilir

### 3. UUID Kullanımı
- Public identifier olarak UUID kullanılır (veritabanı ID'si gizlenir)
- UUID string (VARCHAR 36) olarak saklanır (veritabanı uyumluluğu için)
- Observer'lar ile otomatik oluşturulur

### 4. Soft Delete ve Soft Enabled
- Tüm veriler hardware düzeyinde silinmez (soft delete)
- `is_active` flag'i ile kayıtlar aktif/pasif yapılabilir
- Silinmiş kayıtlar `with_trashed` parametresi ile görüntülenebilir

### 5. Rol Hiyerarşisi
- `priority` alanı ile rol hiyerarşisi belirlenir
- Kullanıcı sadece kendi seviyesinden düşük rolleri yönetebilir
- Sistem rolleri hiyerarşi kontrolünden muaf

### 6. HasPermissions Trait
- `getPermissions()` - Permission tanımları
- `getDefaultRolePermissions()` - Rol-permission atamaları
- `collectAllPermissions()` - Tüm controller'lardan permission'ları toplar
- `collectAllRolePermissions()` - Tüm controller'lardan rol-permission atamalarını toplar

### 7. Dil ve Kullanıcı Ayarları Sistemi
- **UserSetting Model**: Her kullanıcı için otomatik olarak oluşturulan ayar kaydı
- **LocaleHelper**: Dil, tarih/saat formatı ve saat dilimi yönetimi için helper sınıfı
- **SetUserLocale Middleware**: API isteklerinde kullanıcının dil ve saat dilimi tercihlerini otomatik uygular
- **Desteklenen Diller**: tr, en, de, fr, es, it, ru, ar, zh, ja
- **Tarih Formatları**: d/m/Y, Y-m-d, m/d/Y, d.m.Y, Y.m.d
- **Saat Formatları**: H:i, H:i:s, h:i A, h:i:s A
- **Kullanım**: `LocaleHelper::formatDate()`, `LocaleHelper::formatTime()`, `LocaleHelper::formatDateTime()`

### 8. Çeviri Sistemi (Translation System)
- **Translation Model**: Veritabanı tabanlı çeviri sistemi
- **TranslationHelper**: Çeviri yönetimi için helper sınıfı
- **TranslationController**: Çeviri CRUD işlemleri
- **Cache Sistemi**: Çeviriler 24 saat cache'lenir (performans için)
- **Kullanım**: `TranslationHelper::trans()`, `TranslationHelper::getGroup()`, `TranslationHelper::getAll()`
- **Özellikler**: Parametreli çeviriler, grup bazlı organizasyon, otomatik cache temizleme
- **Values Formatı**: Tek bir key için birden fazla dilde çeviri ekleme/güncelleme (`{"key": "...", "values": {"tr": "...", "en": "..."}}`)
- **Bulk Operations**: Toplu ekleme, güncelleme ve silme işlemleri desteklenir

### 9. Dil Yönetimi Sistemi (Locale System)
- **Locale Model**: Veritabanı tabanlı dil yönetimi sistemi
- **LocaleController**: Dil CRUD işlemleri
- **LocaleHelper**: Dinamik dil kodları (`getSupportedLocaleCodes()`, `getSupportedLocales()`)
- **LocaleObserver**: Dil oluşturma için event handler'ları
- **LocaleSeeder**: Başlangıç dil verilerini ekler (tr, en, de, fr, es, it, ru, ar, zh, ja)
- **Özellikler**: 
  - Diller veritabanında saklanır ve dinamik olarak yönetilir
  - `sort_order` ile sıralama yapılabilir
  - `is_active` ile aktif/pasif kontrolü yapılabilir
  - TranslationController'da locale validasyonu dinamik olarak yapılır

---

## 🔧 Yapılandırma

### Ortam Değişkenleri (.env)

```env
# Sistem rol gizli anahtarı
ROLE_SYSTEM_SECRET=your-super-secret-key-here
```

### Config Dosyası

`config/app.php` içinde:
```php
'role_system_secret' => env('ROLE_SYSTEM_SECRET', null),
```

---

## 📝 Notlar

1. **İlk Kullanıcı:** İlk kayıt olan kullanıcı otomatik olarak `mgmt.superadmin` rolü alır
2. **Sonraki Kullanıcılar:** Sonraki tüm kullanıcılar `mgmt.user` rolü ile kayıt olur
3. **UserDetail:** Her kullanıcı oluşturulduğunda otomatik olarak boş bir UserDetail kaydı oluşturulur
4. **Permission Güncelleme:** Permission'ları güncellemek için controller'lardaki `getPermissions()` metodunu güncelleyin ve `RoleSeeder` çalıştırın
5. **Rol-Permission Güncelleme:** Rol-permission atamalarını güncellemek için controller'lardaki `getDefaultRolePermissions()` metodunu güncelleyin ve `RoleSeeder` çalıştırın

---

## 🚀 Hızlı Başlangıç

### 1. Migration'ları Çalıştır
```bash
php artisan migrate
```

### 2. RoleSeeder'ı Çalıştır
```bash
php artisan db:seed --class=RoleSeeder
```

### 3. TranslationSeeder'ı Çalıştır (Örnek çeviriler için)
```bash
php artisan db:seed --class=TranslationSeeder
```

### 4. İlk Kullanıcı Oluştur
```bash
POST /api/v1/register
{
  "email": "admin@example.com",
  "password": "password123"
}
```

İlk kullanıcı otomatik olarak `mgmt.superadmin` rolü alır.

---

**Son Güncelleme:** 2025-11-04 22:22 (Proje Yapısı bölümü tamamen güncellendi, tüm dosyalar listelendi)
**Versiyon:** 1.2.1

# Sistem Genel Bakış Dokümantasyonu

Bu dokümantasyon, mevcut sistemin tüm bileşenlerini, modellerini, controller'larını, API endpoint'lerini ve permission sistemini içerir.

**Son Güncelleme:** 2025-11-04 22:22

---

## 📋 İçindekiler

1. [Proje Yapısı](#proje-yapısı)
2. [Modeller](#modeller)
3. [Controller'lar](#controllerlar)
4. [API Endpoint'leri](#api-endpointleri)
5. [Permission Sistemi](#permission-sistemi)
6. [Rol Sistemi](#rol-sistemi)
7. [Observer'lar](#observerlar)
8. [Migration'lar](#migrationlar)
9. [Önemli Özellikler](#önemli-özellikler)

---

## 📁 Proje Yapısı

```
laravel/example-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Api/          # API Controller'ları (17 controller)
│   │   │       ├── ApiKeyController.php
│   │   │       ├── AuditController.php
│   │   │       ├── AuthController.php
│   │   │       ├── ContentController.php
│   │   │       ├── IntegrationController.php
│   │   │       ├── LocaleController.php
│   │   │       ├── NotificationController.php
│   │   │       ├── OrganizationController.php
│   │   │       ├── PublicUserController.php
│   │   │       ├── ReportController.php
│   │   │       ├── RolePermissionController.php
│   │   │       ├── SessionController.php
│   │   │       ├── SystemController.php
│   │   │       ├── TranslationController.php
│   │   │       ├── UserController.php
│   │   │       ├── UserDetailController.php
│   │   │       └── UserSettingController.php
│   │   └── Middleware/
│   │       ├── PermissionMiddleware.php
│   │       └── SetUserLocale.php
│   ├── Models/                # Eloquent Modeller (14 model)
│   │   ├── ApiKey.php
│   │   ├── AuditLog.php
│   │   ├── Content.php
│   │   ├── Integration.php
│   │   ├── Locale.php
│   │   ├── Notification.php
│   │   ├── Organization.php
│   │   ├── Permission.php
│   │   ├── Report.php
│   │   ├── Role.php
│   │   ├── Translation.php
│   │   ├── User.php
│   │   ├── UserDetail.php
│   │   └── UserSetting.php
│   ├── Observers/             # Model Observer'ları (9 observer)
│   │   ├── ApiKeyObserver.php
│   │   ├── ContentObserver.php
│   │   ├── IntegrationObserver.php
│   │   ├── LocaleObserver.php
│   │   ├── NotificationObserver.php
│   │   ├── OrganizationObserver.php
│   │   ├── ReportObserver.php
│   │   ├── UserObserver.php
│   │   └── UserSettingObserver.php
│   ├── Helpers/
│   │   ├── LocaleHelper.php   # Dil ve yerelleştirme helper'ı
│   │   └── TranslationHelper.php  # Çeviri yönetimi helper'ı
│   └── Traits/
│       └── HasPermissions.php  # Permission yönetimi trait'i
├── database/
│   ├── migrations/            # Veritabanı migration'ları
│   └── seeders/
│       ├── DatabaseSeeder.php # Ana seeder (tüm seeder'ları çağırır)
│       ├── LocaleSeeder.php   # Dil seeder'ı
│       ├── RoleSeeder.php     # Rol ve Permission seeder'ı
│       └── TranslationSeeder.php # Çeviri seeder'ı
├── routes/
│   ├── api.php                # Ana API route dosyası (tüm modül route'larını yükler)
│   └── api/                   # Modüler API route dosyaları (alfabetik sırada)
│       ├── api-keys.php
│       ├── audit.php
│       ├── auth.php
│       ├── content.php
│       ├── integrations.php
│       ├── locales.php
│       ├── misc.php
│       ├── notifications.php
│       ├── organizations.php
│       ├── rbac.php
│       ├── reports.php
│       ├── sessions.php
│       ├── system.php
│       ├── translations.php
│       ├── user.php
│       └── user-settings.php
└── docs/
    ├── ApiResources.md
    ├── DevelopmentGuide.md
    ├── SystemOverview.md
    └── SystemOverview-*.md    # Versiyonlanmış SystemOverview dosyaları
```

---

## 🗄️ Modeller

### 1. User
**Dosya:** `app/Models/User.php`

**Özellikler:**
- `SoftDeletes` trait
- `HasApiTokens` trait (Sanctum)
- `uuid` kolonu (string, 36) - Public identifier
- `is_active` boolean alanı
- `email`, `password` alanları
- `name` alanı kaldırıldı (UserDetail'e taşındı)

**İlişkiler:**
- `detail()` - HasOne (UserDetail)
- `setting()` - HasOne (UserSetting)
- `roles()` - BelongsToMany (Role)

**Özel Metodlar:**
- `hasPermission(string $permissionSlug): bool` - Permission kontrolü
- `getHighestPriorityRole(): ?Role` - En yüksek öncelikli rol
- `getHighestPriorityLevel(): int` - En yüksek öncelik seviyesi
- `canManageRole(Role $targetRole): bool` - Rol yönetebilme kontrolü
- `canAssignRoleToUser(User $targetUser, Role $targetRole): bool` - Rol atayabilme kontrolü
- `hasPermissionOrOwn(string $permission, ?User $resourceOwner = null): bool` - Genel veya `.own` permission kontrolü
- `canManageOwn(string $basePermission, ?User $resourceOwner = null): bool` - Kendi kaynağını yönetebilme
- `canManageResource(string $basePermission, ?User $resourceOwner = null): bool` - Kaynak yönetebilme kontrolü

**Sistem Rolleri:**
- `system.toor` ve `server.root` rolleri tüm permission kontrollerini otomatik bypass eder

---

### 2. UserDetail
**Dosya:** `app/Models/UserDetail.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key

**İlişkiler:**
- `user()` - BelongsTo (User)

**Not:** User oluşturulduğunda otomatik olarak boş bir UserDetail kaydı oluşturulur (Observer ile)

---

### 3. UserSetting
**Dosya:** `app/Models/UserSetting.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key (unique)
- `locale` alanı (string, 10) - Dil kodu (tr, en, de, fr, es, it, ru, ar, zh, ja)
- `timezone` alanı (string, 50) - Saat dilimi (Europe/Istanbul, vb.)
- `date_format` alanı (string, 20) - Tarih formatı (d/m/Y, Y-m-d, vb.)
- `time_format` alanı (string, 20) - Saat formatı (H:i, h:i A, vb.)
- `currency` alanı (string, 3, nullable) - Para birimi (TRY, USD, EUR, vb.)
- `country_code` alanı (string, 2, nullable) - Ülke kodu (TR, US, DE, vb.)
- `notifications_email`, `notifications_push`, `notifications_sms` boolean alanları
- `theme` alanı (string, 20) - Tema (light, dark, auto)
- `items_per_page` alanı (integer) - Sayfa başına öğe sayısı
- `custom_settings` alanı (JSON, nullable) - Özel ayarlar

**İlişkiler:**
- `user()` - BelongsTo (User)

**Özel Metodlar:**
- `getLocale(): string` - Kullanıcının dil tercihini döndürür
- `getTimezone(): string` - Kullanıcının saat dilimini döndürür
- `getDateFormat(): string` - Kullanıcının tarih formatını döndürür
- `getTimeFormat(): string` - Kullanıcının saat formatını döndürür
- `getCurrency(): ?string` - Kullanıcının para birimini döndürür
- `getCountryCode(): ?string` - Kullanıcının ülke kodunu döndürür

**Not:** User oluşturulduğunda otomatik olarak varsayılan değerlerle bir UserSetting kaydı oluşturulur (Observer ile)

---

### 4. Role
**Dosya:** `app/Models/Role.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `slug` alanı (namespace'li: `system.toor`, `mgmt.superadmin`, vb.)
- `priority` alanı (integer) - Düşük sayı = Yüksek yetki

**İlişkiler:**
- `permissions()` - BelongsToMany (Permission)
- `users()` - BelongsToMany (User)

**Özel Metodlar:**
- `isSystemRole(): bool` - Sistem rolü kontrolü (`system.toor`, `server.root`)
- `hasHigherPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması
- `hasLowerPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması
- `hasHigherOrEqualPriorityThan(Role $otherRole): bool` - Öncelik karşılaştırması

**Rol Slug'ları:**
- `system.toor` (priority: 1) - Sistem Seviyesinde erişim
- `server.root` (priority: 2) - Sunucu Seviyesinde Erişim
- `mgmt.superadmin` (priority: 10) - Tam Yetkili Erişim
- `mgmt.admin` (priority: 20) - Yönetici erişimi
- `mgmt.moderator` (priority: 30) - Moderasyon
- `mgmt.editor` (priority: 40) - Editör
- `mgmt.user` (priority: 50) - Kullanıcı
- `mgmt.anonymous` (priority: 60) - Anonim

---

### 5. Permission
**Dosya:** `app/Models/Permission.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `slug` alanı (örn: `users.view`, `content.create.own`)
- `name` alanı (Türkçe açıklama)

**İlişkiler:**
- `roles()` - BelongsToMany (Role)

**Not:** Permission'lar controller'larda tanımlanır ve `RoleSeeder` tarafından otomatik toplanır

---

### 6. Organization
**Dosya:** `app/Models/Organization.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `is_active` boolean alanı
- Kapsamlı alanlar: `organization_code`, `organization_name`, `legal_name`, `tax_number`, vb.

**İlişkiler:**
- Yok (şimdilik)

---

### 7. Content
**Dosya:** `app/Models/Content.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `slug` alanı (otomatik oluşturulur)
- `type` enum: `post`, `page`, `article`, `news`, `document`
- `status` enum: `draft`, `published`, `archived`, `pending`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 8. Report
**Dosya:** `app/Models/Report.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `report_type` alanı
- `status` enum: `pending`, `generating`, `completed`, `failed`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 9. AuditLog
**Dosya:** `app/Models/AuditLog.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `user_id` foreign key (nullable)
- `action` alanı
- `model_type` ve `model_id` alanları (polymorphic)
- `ip_address`, `user_agent` alanları
- `old_values`, `new_values` JSON alanları

**İlişkiler:**
- `user()` - BelongsTo (User)
- `model()` - MorphTo (polymorphic)

---

### 10. Notification
**Dosya:** `app/Models/Notification.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `sender_id` foreign key (nullable)
- `is_active` boolean alanı
- `is_read` boolean alanı
- `type` alanı: `info`, `success`, `warning`, `error`, `system`

**İlişkiler:**
- `user()` - BelongsTo (User)
- `sender()` - BelongsTo (User)

---

### 11. ApiKey
**Dosya:** `app/Models/ApiKey.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `key` alanı (hash'lenmiş, 64 karakter)
- `key_prefix` alanı (ilk 8 karakter)
- `expires_at` timestamp (nullable)

**İlişkiler:**
- `user()` - BelongsTo (User)

**Not:** API anahtarı oluşturulduğunda otomatik olarak hash'lenir ve key_prefix kaydedilir

---

### 12. Integration
**Dosya:** `app/Models/Integration.php`

**Özellikler:**
- `SoftDeletes` trait
- `uuid` kolonu (string, 36)
- `user_id` foreign key
- `is_active` boolean alanı
- `type` alanı: `webhook`, `oauth`, `api`, `custom`
- `provider` alanı (nullable): `google`, `github`, `slack`, vb.
- `status` enum: `active`, `inactive`, `error`, `pending`

**İlişkiler:**
- `user()` - BelongsTo (User)

---

### 13. Translation
**Dosya:** `app/Models/Translation.php`

**Özellikler:**
- `SoftDeletes` trait
- `is_active` boolean alanı
- `key` alanı (string, 255) - Çeviri anahtarı (örn: `pages.home`, `menu.account`)
- `locale` alanı (string, 10) - Dil kodu (tr, en, de, fr, vb.)
- `value` alanı (text) - Çeviri metni
- `group` alanı (string, 50, nullable) - Grup (pages, menu, buttons, messages, vb.)
- `description` alanı (text, nullable) - Açıklama

**İlişkiler:**
- Yok

**Özel Metodlar:**
- `scopeActive()` - Aktif çeviriler
- `scopeLocale()` - Belirli bir dil için çeviriler
- `scopeGroup()` - Belirli bir grup için çeviriler
- `scopeKey()` - Belirli bir key için çeviriler

**Not:** Her key + locale kombinasyonu için tek bir çeviri kaydı olmalı (unique constraint)

---

### 14. Locale
**Dosya:** `app/Models/Locale.php`

**Özellikler:**
- `HasFactory` trait
- `code` alanı (string, 10, unique) - Dil kodu (tr, en, de, fr, vb.)
- `name` alanı (string) - Dil adı (İngilizce)
- `native_name` alanı (string) - Yerel dil adı
- `sort_order` alanı (integer, default: 0) - Sıralama
- `is_active` boolean alanı

**İlişkiler:**
- Yok

**Özel Metodlar:**
- `scopeActive()` - Aktif diller
- `getActiveCodes(): array` - Aktif dil kodlarını döndürür
- `getAllCodes(): array` - Tüm dil kodlarını döndürür

**Not:** Locale bilgileri veritabanında saklanır ve `LocaleHelper::getSupportedLocaleCodes()` ile dinamik olarak çekilir.

---

## 🎮 Controller'lar

Tüm controller'lar `app/Http/Controllers/Api/` klasöründe bulunur ve `HasPermissions` trait'ini kullanır.

### 1. AuthController
**Dosya:** `app/Http/Controllers/Api/AuthController.php`

**Metodlar:**
- `login(Request $request)` - Kullanıcı girişi
- `register(Request $request)` - Kullanıcı kaydı
  - İlk kullanıcı `mgmt.superadmin` rolü alır
  - Diğer kullanıcılar `mgmt.user` rolü alır
- `me(Request $request)` - Mevcut kullanıcı bilgileri
- `logout(Request $request)` - Oturum kapatma

**Permission:** Yok (public endpoint'ler)

---

### 2. UserController
**Dosya:** `app/Http/Controllers/Api/UserController.php`

**Metodlar:**
- `index(Request $request)` - Kullanıcı listesi
- `show(Request $request, User $user)` - Kullanıcı detayı
- `store(Request $request)` - Yeni kullanıcı oluşturma
- `update(Request $request, User $user)` - Kullanıcı güncelleme
- `destroy(Request $request, User $user)` - Kullanıcı silme
- `restore(Request $request, string $uuid)` - Kullanıcı geri yükleme

**Permission'lar:**
- `users.view` / `users.view.own`
- `users.create`
- `users.update` / `users.update.own`
- `users.delete` / `users.delete.own`
- `users.restore` / `users.restore.own`
- `users.export` / `users.export.own`
- `users.import`
- `users.manage.roles` / `users.manage.roles.own`
- `users.manage.status` / `users.manage.status.own`

---

### 3. UserDetailController
**Dosya:** `app/Http/Controllers/Api/UserDetailController.php`

**Metodlar:**
- `show(Request $request)` - Kullanıcı detayları görüntüleme
- `update(Request $request)` - Kullanıcı detayları güncelleme

**Permission'lar:**
- `user.details.view` / `user.details.view.own`
- `user.details.create` / `user.details.create.own`
- `user.details.update` / `user.details.update.own`
- `user.details.delete` / `user.details.delete.own`
- `user.details.restore` / `user.details.restore.own`

---

### 4. UserSettingController
**Dosya:** `app/Http/Controllers/Api/UserSettingController.php`

**Metodlar:**
- `show(Request $request)` - Kullanıcı ayarlarını görüntüleme
- `update(Request $request)` - Kullanıcı ayarlarını güncelleme
- `destroy(Request $request)` - Kullanıcı ayarlarını silme (soft delete)
- `restore(Request $request, string $uuid)` - Kullanıcı ayarlarını geri yükleme

**Permission'lar:**
- `user.settings.view` / `user.settings.view.own`
- `user.settings.update` / `user.settings.update.own`
- `user.settings.delete` / `user.settings.delete.own`
- `user.settings.restore` / `user.settings.restore.own`

---

### 5. PublicUserController
**Dosya:** `app/Http/Controllers/Api/PublicUserController.php`

**Metodlar:**
- `show(User $user)` - Public kullanıcı profili (UUID ile)

**Permission:** Yok (public endpoint)

---

### 6. RolePermissionController
**Dosya:** `app/Http/Controllers/Api/RolePermissionController.php`

**Metodlar:**
- `createRole(Request $request)` - Rol oluşturma
- `listRoles(Request $request)` - Rol listesi
- `updateRole(Request $request, int $id)` - Rol güncelleme
- `deleteRole(int $id)` - Rol silme
- `restoreRole(Request $request, int $id)` - Rol geri yükleme
- `createPermission(Request $request)` - Permission oluşturma
- `listPermissions(Request $request)` - Permission listesi
- `updatePermission(Request $request, int $id)` - Permission güncelleme
- `deletePermission(int $id)` - Permission silme
- `restorePermission(Request $request, int $id)` - Permission geri yükleme
- `assignPermissionToRole(Request $request)` - Rol'e permission atama
- `removePermissionFromRole(Request $request)` - Rol'den permission kaldırma
- `assignRoleToUser(Request $request)` - Kullanıcıya rol atama
- `removeRoleFromUser(Request $request)` - Kullanıcıdan rol kaldırma

**Permission'lar:**
- `roles.view`, `roles.create`, `roles.update`, `roles.delete`, `roles.restore`, `roles.assign`, `roles.revoke`
- `permissions.view`, `permissions.create`, `permissions.update`, `permissions.delete`, `permissions.restore`, `permissions.assign`

**Özel Kontroller:**
- Sistem rolleri (`system.toor`, `server.root`) için gizli anahtar zorunlu
- Rol hiyerarşisi kontrolü (kullanıcı sadece kendi seviyesinden düşük rolleri yönetebilir)

---

### 7. OrganizationController
**Dosya:** `app/Http/Controllers/Api/OrganizationController.php`

**Metodlar:**
- `index(Request $request)` - Organizasyon listesi
- `show(Request $request, Organization $organization)` - Organizasyon detayı
- `store(Request $request)` - Yeni organizasyon oluşturma
- `update(Request $request, Organization $organization)` - Organizasyon güncelleme
- `destroy(Request $request, Organization $organization)` - Organizasyon silme
- `restore(Request $request, string $uuid)` - Organizasyon geri yükleme

**Permission'lar:**
- `organizations.view` / `organizations.view.own`
- `organizations.create` / `organizations.create.own`
- `organizations.update` / `organizations.update.own`
- `organizations.delete` / `organizations.delete.own`
- `organizations.restore` / `organizations.restore.own`
- `organizations.export` / `organizations.export.own`
- `organizations.import`
- `organizations.manage.members` / `organizations.manage.members.own`

---

### 8. ContentController
**Dosya:** `app/Http/Controllers/Api/ContentController.php`

**Metodlar:**
- `index(Request $request)` - İçerik listesi
- `show(Request $request, Content $content)` - İçerik detayı
- `store(Request $request)` - Yeni içerik oluşturma
- `update(Request $request, Content $content)` - İçerik güncelleme
- `destroy(Request $request, Content $content)` - İçerik silme
- `restore(Request $request, string $uuid)` - İçerik geri yükleme
- `publish(Request $request, Content $content)` - İçerik yayınlama
- `unpublish(Request $request, Content $content)` - İçerik yayından kaldırma

**Permission'lar:**
- `content.view` / `content.view.own`
- `content.create` / `content.create.own`
- `content.update` / `content.update.own`
- `content.delete` / `content.delete.own`
- `content.restore` / `content.restore.own`
- `content.publish` / `content.publish.own`
- `content.unpublish` / `content.unpublish.own`
- `content.moderate`
- `content.edit` / `content.edit.own`
- `content.export` / `content.export.own`

---

### 9. ReportController
**Dosya:** `app/Http/Controllers/Api/ReportController.php`

**Metodlar:**
- `index(Request $request)` - Rapor listesi
- `show(Request $request, Report $report)` - Rapor detayı
- `store(Request $request)` - Yeni rapor oluşturma
- `update(Request $request, Report $report)` - Rapor güncelleme
- `destroy(Request $request, Report $report)` - Rapor silme

**Permission'lar:**
- `reports.view` / `reports.view.own`
- `reports.generate` / `reports.generate.own`
- `reports.export` / `reports.export.own`
- `reports.delete` / `reports.delete.own`

---

### 10. AuditController
**Dosya:** `app/Http/Controllers/Api/AuditController.php`

**Metodlar:**
- `index(Request $request)` - Denetim log listesi
- `show(Request $request, AuditLog $auditLog)` - Denetim log detayı
- `destroy(Request $request, AuditLog $auditLog)` - Denetim log silme

**Permission'lar:**
- `audit.view` / `audit.view.own`
- `audit.export` / `audit.export.own`
- `audit.delete` / `audit.delete.own`

---

### 11. NotificationController
**Dosya:** `app/Http/Controllers/Api/NotificationController.php`

**Metodlar:**
- `index(Request $request)` - Bildirim listesi
- `show(Request $request, Notification $notification)` - Bildirim detayı
- `store(Request $request)` - Yeni bildirim oluşturma
- `update(Request $request, Notification $notification)` - Bildirim güncelleme
- `destroy(Request $request, Notification $notification)` - Bildirim silme
- `markAsRead(Request $request, Notification $notification)` - Bildirim okundu işaretleme

**Permission'lar:**
- `notifications.send` / `notifications.send.own`
- `notifications.manage` / `notifications.manage.own`
- `notifications.view` / `notifications.view.own`
- `notifications.update` / `notifications.update.own`
- `notifications.delete` / `notifications.delete.own`
- `notifications.mark.read` / `notifications.mark.read.own`

---

### 12. ApiKeyController
**Dosya:** `app/Http/Controllers/Api/ApiKeyController.php`

**Metodlar:**
- `index(Request $request)` - API anahtarı listesi
- `show(Request $request, ApiKey $apiKey)` - API anahtarı detayı
- `store(Request $request)` - Yeni API anahtarı oluşturma
- `update(Request $request, ApiKey $apiKey)` - API anahtarı güncelleme
- `revoke(Request $request, ApiKey $apiKey)` - API anahtarı iptal etme

**Permission'lar:**
- `api.keys.manage` / `api.keys.manage.own`
- `api.keys.view` / `api.keys.view.own`
- `api.keys.create` / `api.keys.create.own`
- `api.keys.update` / `api.keys.update.own`
- `api.keys.revoke` / `api.keys.revoke.own`

---

### 13. IntegrationController
**Dosya:** `app/Http/Controllers/Api/IntegrationController.php`

**Metodlar:**
- `index(Request $request)` - Entegrasyon listesi
- `show(Request $request, Integration $integration)` - Entegrasyon detayı
- `store(Request $request)` - Yeni entegrasyon oluşturma
- `update(Request $request, Integration $integration)` - Entegrasyon güncelleme
- `destroy(Request $request, Integration $integration)` - Entegrasyon silme

**Permission'lar:**
- `integrations.manage` / `integrations.manage.own`
- `integrations.view` / `integrations.view.own`
- `integrations.create` / `integrations.create.own`

---

### 14. SystemController
**Dosya:** `app/Http/Controllers/Api/SystemController.php`

**Metodlar:**
- `getSettings(Request $request)` - Sistem ayarları
- `getLogs(Request $request)` - Sistem logları
- `getMetrics(Request $request)` - Sistem metrikleri

**Permission'lar:**
- `system.settings`
- `system.logs`
- `system.monitor`
- `system.backups` (RoleSeeder'da tanımlı)
- `system.maintenance` (RoleSeeder'da tanımlı)

---

### 15. SessionController
**Dosya:** `app/Http/Controllers/Api/SessionController.php`

**Metodlar:**
- `index(Request $request)` - Oturum listesi (Sanctum token'ları)
- `show(Request $request)` - Mevcut oturum
- `revoke(Request $request, $tokenId)` - Oturum iptal etme
- `revokeAll(Request $request)` - Tüm oturumları iptal etme (mevcut hariç)

**Permission'lar:**
- `sessions.view` / `sessions.view.own`
- `sessions.manage` / `sessions.manage.own`
- `sessions.create` / `sessions.create.own`
- `sessions.revoke` / `sessions.revoke.own`

---

### 16. TranslationController
**Dosya:** `app/Http/Controllers/Api/TranslationController.php`

**Metodlar:**
- `index(Request $request)` - Çeviri listesi (arama, filtreleme, pagination)
- `show(Request $request, Translation $translation)` - Çeviri detayı
- `get(Request $request, string $key)` - Çeviri anahtarına göre çeviri metnini döndürür (Public)
- `getGroup(Request $request, string $group)` - Bir grup için tüm çevirileri döndürür (Public)
- `getAll(Request $request)` - Tüm çevirileri döndürür (locale'e göre) (Public)
- `store(Request $request)` - Yeni çeviri oluşturma (values formatı desteklenir)
- `bulkStore(Request $request)` - Toplu çeviri oluşturma
- `update(Request $request, ?Translation $translation)` - Çeviri güncelleme (values formatı desteklenir)
- `bulkUpdate(Request $request)` - Toplu çeviri güncelleme (upsert)
- `destroy(Request $request, Translation $translation)` - Çeviri silme
- `bulkDelete(Request $request)` - Toplu çeviri silme
- `restore(Request $request, int $id)` - Çeviri geri yükleme

**Permission'lar:**
- `translations.view`
- `translations.create`
- `translations.update`
- `translations.delete`
- `translations.restore`
- `translations.manage`

**Not:** Locale validasyonu `LocaleHelper::getSupportedLocaleCodes()` ile veritabanından dinamik olarak yapılır.

---

### 17. LocaleController
**Dosya:** `app/Http/Controllers/Api/LocaleController.php`

**Metodlar:**
- `index(Request $request)` - Dil listesi (arama, filtreleme, pagination)
- `show(Locale $locale)` - Dil detayı (Public)
- `store(Request $request)` - Yeni dil oluşturma
- `update(Request $request, Locale $locale)` - Dil güncelleme
- `destroy(Request $request, Locale $locale)` - Dil silme (soft delete)
- `restore(Request $request, int $id)` - Dil geri yükleme

**Permission'lar:**
- `locales.view`
- `locales.create`
- `locales.update`
- `locales.delete`
- `locales.restore`
- `locales.manage`

---

## 🌐 API Endpoint'leri

Tüm endpoint'ler `/api/v1/` prefix'i altında çalışır.

### Authentication Endpoints
**Dosya:** `routes/api/auth.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| POST | `/login` | AuthController@login | Public | Kullanıcı girişi |
| POST | `/register` | AuthController@register | Public | Kullanıcı kaydı |
| GET | `/me` | AuthController@me | auth:sanctum | Mevcut kullanıcı bilgileri |
| POST | `/logout` | AuthController@logout | auth:sanctum | Oturum kapatma |

---

### User Endpoints
**Dosya:** `routes/api/user.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/user/details` | UserDetailController@show | `user.details.view` / `user.details.view.own` | Kullanıcı detayları |
| PUT | `/user/details` | UserDetailController@update | `user.details.update` / `user.details.update.own` | Kullanıcı detayları güncelleme |
| GET | `/user/settings` | UserSettingController@show | `user.settings.view` / `user.settings.view.own` | Kullanıcı ayarları |
| PUT | `/user/settings` | UserSettingController@update | `user.settings.update` / `user.settings.update.own` | Kullanıcı ayarları güncelleme |
| PATCH | `/user/settings` | UserSettingController@update | `user.settings.update` / `user.settings.update.own` | Kullanıcı ayarları güncelleme |
| DELETE | `/user/settings` | UserSettingController@destroy | `user.settings.delete` / `user.settings.delete.own` | Kullanıcı ayarları silme |
| POST | `/user/settings/{uuid}/restore` | UserSettingController@restore | `user.settings.restore` / `user.settings.restore.own` | Kullanıcı ayarları geri yükleme |
| GET | `/users` | UserController@index | `users.view` / `users.view.own` | Kullanıcı listesi |
| POST | `/users` | UserController@store | `users.create` | Yeni kullanıcı oluşturma |
| GET | `/users/{user}` | UserController@show | `users.view` / `users.view.own` | Kullanıcı detayı |
| PUT | `/users/{user}` | UserController@update | `users.update` / `users.update.own` | Kullanıcı güncelleme |
| PATCH | `/users/{user}` | UserController@update | `users.update` / `users.update.own` | Kullanıcı güncelleme |
| DELETE | `/users/{user}` | UserController@destroy | `users.delete` / `users.delete.own` | Kullanıcı silme |
| POST | `/users/{uuid}/restore` | UserController@restore | `users.restore` / `users.restore.own` | Kullanıcı geri yükleme |
| GET | `/users/{user}` | PublicUserController@show | Public | Public kullanıcı profili (UUID ile) |

---

### RBAC Endpoints
**Dosya:** `routes/api/rbac.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/roles` | RolePermissionController@listRoles | `admin.manage` | Rol listesi |
| POST | `/roles` | RolePermissionController@createRole | `admin.manage` | Rol oluşturma |
| PATCH | `/roles/{id}` | RolePermissionController@updateRole | `admin.manage` | Rol güncelleme |
| PUT | `/roles/{id}` | RolePermissionController@updateRole | `admin.manage` | Rol güncelleme |
| DELETE | `/roles/{id}` | RolePermissionController@deleteRole | `admin.manage` | Rol silme |
| POST | `/roles/{id}/restore` | RolePermissionController@restoreRole | `admin.manage` | Rol geri yükleme |
| GET | `/permissions` | RolePermissionController@listPermissions | `admin.manage` | Permission listesi |
| POST | `/permissions` | RolePermissionController@createPermission | `admin.manage` | Permission oluşturma |
| PATCH | `/permissions/{id}` | RolePermissionController@updatePermission | `admin.manage` | Permission güncelleme |
| PUT | `/permissions/{id}` | RolePermissionController@updatePermission | `admin.manage` | Permission güncelleme |
| DELETE | `/permissions/{id}` | RolePermissionController@deletePermission | `admin.manage` | Permission silme |
| POST | `/permissions/{id}/restore` | RolePermissionController@restorePermission | `admin.manage` | Permission geri yükleme |
| POST | `/roles/assign-permission` | RolePermissionController@assignPermissionToRole | `admin.manage` | Rol'e permission atama |
| POST | `/roles/remove-permission` | RolePermissionController@removePermissionFromRole | `admin.manage` | Rol'den permission kaldırma |
| POST | `/users/assign-role` | RolePermissionController@assignRoleToUser | `admin.manage` | Kullanıcıya rol atama |
| POST | `/users/remove-role` | RolePermissionController@removeRoleFromUser | `admin.manage` | Kullanıcıdan rol kaldırma |

**Özel Kontroller:**
- Sistem rolleri (`system.toor`, `server.root`) için gizli anahtar zorunlu
- Rol hiyerarşisi kontrolü

---

### Organization Endpoints
**Dosya:** `routes/api/organizations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/organizations/{organization}` | OrganizationController@show | Public | Public organizasyon detayı |
| GET | `/organizations` | OrganizationController@index | `organizations.view` / `organizations.view.own` | Organizasyon listesi |
| POST | `/organizations` | OrganizationController@store | `organizations.create` / `organizations.create.own` | Organizasyon oluşturma |
| PUT | `/organizations/{organization}` | OrganizationController@update | `organizations.update` / `organizations.update.own` | Organizasyon güncelleme |
| PATCH | `/organizations/{organization}` | OrganizationController@update | `organizations.update` / `organizations.update.own` | Organizasyon güncelleme |
| DELETE | `/organizations/{organization}` | OrganizationController@destroy | `organizations.delete` / `organizations.delete.own` | Organizasyon silme |
| POST | `/organizations/{uuid}/restore` | OrganizationController@restore | `organizations.restore` / `organizations.restore.own` | Organizasyon geri yükleme |

---

### Content Endpoints
**Dosya:** `routes/api/content.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/contents/{content}` | ContentController@show | Public | Public içerik görüntüleme |
| GET | `/contents` | ContentController@index | `content.view` / `content.view.own` | İçerik listesi |
| POST | `/contents` | ContentController@store | `content.create` / `content.create.own` | İçerik oluşturma |
| PUT | `/contents/{content}` | ContentController@update | `content.update` / `content.update.own` | İçerik güncelleme |
| PATCH | `/contents/{content}` | ContentController@update | `content.update` / `content.update.own` | İçerik güncelleme |
| DELETE | `/contents/{content}` | ContentController@destroy | `content.delete` / `content.delete.own` | İçerik silme |
| POST | `/contents/{uuid}/restore` | ContentController@restore | `content.restore` / `content.restore.own` | İçerik geri yükleme |
| POST | `/contents/{content}/publish` | ContentController@publish | `content.publish` / `content.publish.own` | İçerik yayınlama |
| POST | `/contents/{content}/unpublish` | ContentController@unpublish | `content.unpublish` / `content.unpublish.own` | İçerik yayından kaldırma |

---

### Report Endpoints
**Dosya:** `routes/api/reports.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/reports` | ReportController@index | `reports.view` / `reports.view.own` | Rapor listesi |
| GET | `/reports/{report}` | ReportController@show | `reports.view` / `reports.view.own` | Rapor detayı |
| POST | `/reports` | ReportController@store | `reports.generate` / `reports.generate.own` | Rapor oluşturma |
| PUT | `/reports/{report}` | ReportController@update | `reports.generate` / `reports.generate.own` | Rapor güncelleme |
| PATCH | `/reports/{report}` | ReportController@update | `reports.generate` / `reports.generate.own` | Rapor güncelleme |
| DELETE | `/reports/{report}` | ReportController@destroy | `reports.delete` / `reports.delete.own` | Rapor silme |

---

### Audit Endpoints
**Dosya:** `routes/api/audit.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/audit-logs` | AuditController@index | `audit.view` / `audit.view.own` | Denetim log listesi |
| GET | `/audit-logs/{auditLog}` | AuditController@show | `audit.view` / `audit.view.own` | Denetim log detayı |
| DELETE | `/audit-logs/{auditLog}` | AuditController@destroy | `audit.delete` / `audit.delete.own` | Denetim log silme |

---

### Notification Endpoints
**Dosya:** `routes/api/notifications.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/notifications` | NotificationController@index | `notifications.view` / `notifications.view.own` | Bildirim listesi |
| GET | `/notifications/{notification}` | NotificationController@show | `notifications.view` / `notifications.view.own` | Bildirim detayı |
| POST | `/notifications` | NotificationController@store | `notifications.send` / `notifications.send.own` | Bildirim oluşturma |
| PUT | `/notifications/{notification}` | NotificationController@update | `notifications.update` / `notifications.update.own` | Bildirim güncelleme |
| PATCH | `/notifications/{notification}` | NotificationController@update | `notifications.update` / `notifications.update.own` | Bildirim güncelleme |
| DELETE | `/notifications/{notification}` | NotificationController@destroy | `notifications.delete` / `notifications.delete.own` | Bildirim silme |
| POST | `/notifications/{notification}/mark-read` | NotificationController@markAsRead | `notifications.mark.read` / `notifications.mark.read.own` | Bildirim okundu işaretleme |

---

### API Key Endpoints
**Dosya:** `routes/api/api-keys.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/api-keys` | ApiKeyController@index | `api.keys.view` / `api.keys.view.own` | API anahtarı listesi |
| GET | `/api-keys/{apiKey}` | ApiKeyController@show | `api.keys.view` / `api.keys.view.own` | API anahtarı detayı |
| POST | `/api-keys` | ApiKeyController@store | `api.keys.create` / `api.keys.create.own` | API anahtarı oluşturma |
| PUT | `/api-keys/{apiKey}` | ApiKeyController@update | `api.keys.update` / `api.keys.update.own` | API anahtarı güncelleme |
| PATCH | `/api-keys/{apiKey}` | ApiKeyController@update | `api.keys.update` / `api.keys.update.own` | API anahtarı güncelleme |
| POST | `/api-keys/{apiKey}/revoke` | ApiKeyController@revoke | `api.keys.revoke` / `api.keys.revoke.own` | API anahtarı iptal etme |

---

### Integration Endpoints
**Dosya:** `routes/api/integrations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/integrations` | IntegrationController@index | `integrations.view` / `integrations.view.own` | Entegrasyon listesi |
| GET | `/integrations/{integration}` | IntegrationController@show | `integrations.view` / `integrations.view.own` | Entegrasyon detayı |
| POST | `/integrations` | IntegrationController@store | `integrations.create` / `integrations.create.own` | Entegrasyon oluşturma |
| PUT | `/integrations/{integration}` | IntegrationController@update | `integrations.manage` / `integrations.manage.own` | Entegrasyon güncelleme |
| PATCH | `/integrations/{integration}` | IntegrationController@update | `integrations.manage` / `integrations.manage.own` | Entegrasyon güncelleme |
| DELETE | `/integrations/{integration}` | IntegrationController@destroy | `integrations.manage` / `integrations.manage.own` | Entegrasyon silme |

---

### System Endpoints
**Dosya:** `routes/api/system.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/system/settings` | SystemController@getSettings | `system.settings` | Sistem ayarları |
| GET | `/system/logs` | SystemController@getLogs | `system.logs` | Sistem logları |
| GET | `/system/metrics` | SystemController@getMetrics | `system.monitor` | Sistem metrikleri |

---

### Session Endpoints
**Dosya:** `routes/api/sessions.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/sessions` | SessionController@index | `sessions.view` / `sessions.view.own` | Oturum listesi |
| GET | `/sessions/current` | SessionController@show | `sessions.view` / `sessions.view.own` | Mevcut oturum |
| POST | `/sessions/{tokenId}/revoke` | SessionController@revoke | `sessions.revoke` / `sessions.revoke.own` | Oturum iptal etme |
| POST | `/sessions/revoke-all` | SessionController@revokeAll | `sessions.revoke` / `sessions.revoke.own` | Tüm oturumları iptal etme |

---

### Translation Endpoints
**Dosya:** `routes/api/translations.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/translations` | TranslationController@getAll | Public | Tüm çevirileri alma (locale'e göre) |
| GET | `/translations/{key}` | TranslationController@get | Public | Çeviri metnini alma (key ile) |
| GET | `/translations/group/{group}` | TranslationController@getGroup | Public | Grup çevirilerini alma |
| GET | `/admin/translations` | TranslationController@index | `translations.view` | Çeviri listesi (arama, filtreleme) |
| GET | `/admin/translations/{translation}` | TranslationController@show | `translations.view` | Çeviri detayı |
| POST | `/admin/translations` | TranslationController@store | `translations.create` | Çeviri oluşturma (tek veya toplu) |
| POST | `/admin/translations/bulk` | TranslationController@bulkStore | `translations.create` | Toplu çeviri oluşturma |
| PUT | `/admin/translations/bulk` | TranslationController@bulkUpdate | `translations.update` | Toplu çeviri güncelleme (upsert) |
| PATCH | `/admin/translations/bulk` | TranslationController@bulkUpdate | `translations.update` | Toplu çeviri güncelleme (upsert) |
| DELETE | `/admin/translations/bulk` | TranslationController@bulkDelete | `translations.delete` | Toplu çeviri silme |
| PUT | `/admin/translations/{translation}` | TranslationController@update | `translations.update` | Çeviri güncelleme |
| PATCH | `/admin/translations/{translation}` | TranslationController@update | `translations.update` | Çeviri güncelleme |
| DELETE | `/admin/translations/{translation}` | TranslationController@destroy | `translations.delete` | Çeviri silme |
| POST | `/admin/translations/{id}/restore` | TranslationController@restore | `translations.restore` | Çeviri geri yükleme |

---

### Locale Endpoints
**Dosya:** `routes/api/locales.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/locales/{locale}` | LocaleController@show | Public | Dil detayı |
| GET | `/admin/locales` | LocaleController@index | `locales.view` | Dil listesi (arama, filtreleme) |
| POST | `/admin/locales` | LocaleController@store | `locales.create` | Yeni dil oluşturma |
| PUT | `/admin/locales/{locale}` | LocaleController@update | `locales.update` | Dil güncelleme |
| PATCH | `/admin/locales/{locale}` | LocaleController@update | `locales.update` | Dil güncelleme |
| DELETE | `/admin/locales/{locale}` | LocaleController@destroy | `locales.delete` | Dil silme |
| POST | `/admin/locales/{id}/restore` | LocaleController@restore | `locales.restore` | Dil geri yükleme |

---

### Misc Endpoints
**Dosya:** `routes/api/misc.php`

| Method | Endpoint | Controller | Permission | Açıklama |
|--------|----------|------------|------------|----------|
| GET | `/ping` | Closure | Public | Sağlık kontrolü |

---

## 🔐 Permission Sistemi

### Permission Tanımlama
Permission'lar controller'larda `getPermissions()` metodu ile tanımlanır:

```php
public static function getPermissions(): array
{
    return [
        ['slug' => 'module.view', 'name' => 'Modül Görüntüleme'],
        ['slug' => 'module.view.own', 'name' => 'Kendi Modülünü Görüntüleme'],
        // ...
    ];
}
```

### Permission Kontrolü
Controller metodları içinde permission kontrolü yapılır:

```php
// Genel permission veya .own permission kontrolü
if (! $user->hasPermissionOrOwn('module.view')) {
    return response()->json(['message' => 'Forbidden'], 403);
}

// Kaynak sahibi kontrolü ile permission kontrolü
if (! $user->canManageResource('module.update', $resource->user)) {
    return response()->json(['message' => 'Forbidden'], 403);
}
```

### Permission Kategorileri

#### 1. Sistem Yönetimi
- `admin.manage` - Admin Yönetimi - Tam Yetki
- `system.settings` - Sistem Ayarları Yönetimi
- `system.logs` - Sistem Loglarını Görüntüleme
- `system.backups` - Yedekleme Yönetimi
- `system.monitor` - Sistem İzleme ve Metrikler
- `system.maintenance` - Bakım Modu Yönetimi

#### 2. Kullanıcı Yönetimi
- `users.manage` - Kullanıcı Yönetimi - Tam Yetki
- `users.view` / `users.view.own`
- `users.create`
- `users.update` / `users.update.own`
- `users.delete` / `users.delete.own`
- `users.restore` / `users.restore.own`
- `users.export` / `users.export.own`
- `users.import`
- `users.manage.roles` / `users.manage.roles.own`
- `users.manage.status` / `users.manage.status.own`

#### 3. Kullanıcı Detayları
- `user.details.view` / `user.details.view.own`
- `user.details.create` / `user.details.create.own`
- `user.details.update` / `user.details.update.own`
- `user.details.delete` / `user.details.delete.own`
- `user.details.restore` / `user.details.restore.own`

#### 4. Rol Yönetimi
- `roles.manage` - Rol Yönetimi - Tam Yetki
- `roles.view`
- `roles.create`
- `roles.update`
- `roles.delete`
- `roles.restore`
- `roles.assign`
- `roles.revoke`

#### 5. İzin Yönetimi
- `permissions.manage` - İzin Yönetimi - Tam Yetki
- `permissions.view`
- `permissions.create`
- `permissions.update`
- `permissions.delete`
- `permissions.restore`
- `permissions.assign`

#### 6. Organizasyon Yönetimi
- `organizations.manage` - Organizasyon Yönetimi - Tam Yetki
- `organizations.view` / `organizations.view.own`
- `organizations.create` / `organizations.create.own`
- `organizations.update` / `organizations.update.own`
- `organizations.delete` / `organizations.delete.own`
- `organizations.restore` / `organizations.restore.own`
- `organizations.export` / `organizations.export.own`
- `organizations.import`
- `organizations.manage.members` / `organizations.manage.members.own`

#### 7. İçerik Yönetimi
- `content.manage` - İçerik Yönetimi - Tam Yetki
- `content.view` / `content.view.own`
- `content.create` / `content.create.own`
- `content.update` / `content.update.own`
- `content.delete` / `content.delete.own`
- `content.restore` / `content.restore.own`
- `content.publish` / `content.publish.own`
- `content.unpublish` / `content.unpublish.own`
- `content.moderate`
- `content.edit` / `content.edit.own`
- `content.export` / `content.export.own`

#### 8. Raporlar
- `reports.view` / `reports.view.own`
- `reports.generate` / `reports.generate.own`
- `reports.export` / `reports.export.own`
- `reports.delete` / `reports.delete.own`

#### 9. Denetim ve Loglar
- `audit.view` / `audit.view.own`
- `audit.export` / `audit.export.own`
- `audit.delete` / `audit.delete.own`

#### 10. Oturum Yönetimi
- `sessions.view` / `sessions.view.own`
- `sessions.manage` / `sessions.manage.own`
- `sessions.create` / `sessions.create.own`
- `sessions.revoke` / `sessions.revoke.own`

#### 11. Bildirimler
- `notifications.send` / `notifications.send.own`
- `notifications.manage` / `notifications.manage.own`
- `notifications.view` / `notifications.view.own`
- `notifications.update` / `notifications.update.own`
- `notifications.delete` / `notifications.delete.own`
- `notifications.mark.read` / `notifications.mark.read.own`

#### 12. API ve Entegrasyonlar
- `api.keys.manage` / `api.keys.manage.own`
- `api.keys.view` / `api.keys.view.own`
- `api.keys.create` / `api.keys.create.own`
- `api.keys.update` / `api.keys.update.own`
- `api.keys.revoke` / `api.keys.revoke.own`
- `integrations.manage` / `integrations.manage.own`
- `integrations.view` / `integrations.view.own`
- `integrations.create` / `integrations.create.own`

#### 13. Çeviriler
- `translations.view`
- `translations.create`
- `translations.update`
- `translations.delete`
- `translations.restore`
- `translations.manage`

#### 14. Diller
- `locales.manage` - Dil Yönetimi - Tam Yetki
- `locales.view`
- `locales.create`
- `locales.update`
- `locales.delete`
- `locales.restore`

---

## 👥 Rol Sistemi

### Rol Hiyerarşisi

Rol hiyerarşisi `priority` alanı ile belirlenir. Düşük sayı = Yüksek yetki.

| Rol | Slug | Priority | Açıklama |
|-----|------|----------|----------|
| Toor | `system.toor` | 1 | Sistem Seviyesinde erişim (gizli anahtar ile) |
| Root | `server.root` | 2 | Sunucu Seviyesinde Erişim (gizli anahtar ile) |
| Super Admin | `mgmt.superadmin` | 10 | Tam Yetkili Erişim |
| Admin | `mgmt.admin` | 20 | Yönetici erişimi |
| Moderator | `mgmt.moderator` | 30 | Moderasyon |
| Editor | `mgmt.editor` | 40 | Editör |
| User | `mgmt.user` | 50 | Kullanıcı |
| Anonymous | `mgmt.anonymous` | 60 | Anonim |

### Sistem Rolleri

`system.toor` ve `server.root` rolleri:
- Tüm permission kontrollerini otomatik bypass eder
- Sadece gizli anahtar (`ROLE_SYSTEM_SECRET`) ile atanabilir
- Tüm permission'lara otomatik sahip olur
- Rol hiyerarşisi kontrolünden muaf

### Rol-Permission Atamaları

Rol-permission atamaları controller'larda `getDefaultRolePermissions()` metodu ile tanımlanır:

```php
public static function getDefaultRolePermissions(): array
{
    return [
        'mgmt.superadmin' => [
            'module.view', 'module.create', 'module.update', 'module.delete',
        ],
        'mgmt.user' => [
            'module.view.own', 'module.create.own', 'module.update.own',
        ],
    ];
}
```

### RoleSeeder

`RoleSeeder` otomatik olarak:
1. Controller'lardan permission'ları toplar (`collectAllPermissions()`)
2. Controller'lardan rol-permission atamalarını toplar (`collectAllRolePermissions()`)
3. Sistem rolleri için tüm permission'ları atar
4. Veritabanına kaydeder

---

## 👁️ Observer'lar

Observer'lar `app/Observers/` klasöründe bulunur ve `AppServiceProvider` içinde kaydedilir.

### Mevcut Observer'lar:
1. **UserObserver** - UUID oluşturma, UserDetail ve UserSetting otomatik oluşturma
2. **OrganizationObserver** - UUID oluşturma
3. **ContentObserver** - UUID oluşturma, slug otomatik oluşturma
4. **ReportObserver** - UUID oluşturma
5. **NotificationObserver** - UUID oluşturma
6. **ApiKeyObserver** - UUID oluşturma, API anahtarı hash'leme
7. **IntegrationObserver** - UUID oluşturma
8. **UserSettingObserver** - Ayarlar için event handler'ları
9. **LocaleObserver** - Locale oluşturma için event handler'ları

---

## 🗄️ Migration'lar

### Mevcut Tablolar:
1. **users** - Kullanıcılar (uuid, email, password, is_active, deleted_at)
2. **users_details** - Kullanıcı detayları (user_id, is_active, deleted_at)
3. **user_settings** - Kullanıcı ayarları (user_id, locale, timezone, date_format, time_format, currency, country_code, notifications, theme, items_per_page, custom_settings, is_active, deleted_at)
4. **user_role** - Roller (slug, name, priority, is_active, deleted_at)
5. **permissions** - İzinler (slug, name, is_active, deleted_at)
6. **user_roles** - Kullanıcı-Rol pivot tablosu
7. **role_permissions** - Rol-Permission pivot tablosu
8. **organizations** - Organizasyonlar (uuid, organization_code, organization_name, vb.)
9. **contents** - İçerikler (uuid, user_id, title, slug, content, type, status, vb.)
10. **reports** - Raporlar (uuid, user_id, title, report_type, status, vb.)
11. **audit_logs** - Denetim logları (user_id, action, model_type, model_id, vb.)
12. **notifications** - Bildirimler (uuid, user_id, sender_id, type, title, message, vb.)
13. **api_keys** - API anahtarları (uuid, user_id, name, key, key_prefix, vb.)
14. **integrations** - Entegrasyonlar (uuid, user_id, name, type, provider, vb.)
15. **translations** - Çeviriler (key, locale, value, group, description, is_active, deleted_at)
16. **locales** - Diller (code, name, native_name, sort_order, is_active, timestamps)

**Ortak Özellikler:**
- Tüm tablolarda `id` (primary key)
- Public erişim gereken tablolarda `uuid` (string, 36, unique)
- Tüm tablolarda `is_active` (boolean, default: true)
- Tüm tablolarda `deleted_at` (soft delete)
- Tüm tablolarda `created_at` ve `updated_at` (timestamps)

---

## ⚡ Önemli Özellikler

### 1. Sistem Rolleri Bypass
- `system.toor` ve `server.root` rolleri tüm permission kontrollerini otomatik bypass eder
- `User::hasPermission()` metodunda sistem rolleri kontrolü yapılır
- `PermissionMiddleware` içinde sistem rolleri bypass edilir

### 2. Permission Sistemi
- Permission'lar controller'larda tanımlanır
- Rol-permission atamaları controller'larda tanımlanır
- `RoleSeeder` otomatik olarak toplar ve veritabanına ekler
- `.own` permission'ları ile kullanıcı sadece kendi kaynaklarını yönetebilir

### 3. UUID Kullanımı
- Public identifier olarak UUID kullanılır (veritabanı ID'si gizlenir)
- UUID string (VARCHAR 36) olarak saklanır (veritabanı uyumluluğu için)
- Observer'lar ile otomatik oluşturulur

### 4. Soft Delete ve Soft Enabled
- Tüm veriler hardware düzeyinde silinmez (soft delete)
- `is_active` flag'i ile kayıtlar aktif/pasif yapılabilir
- Silinmiş kayıtlar `with_trashed` parametresi ile görüntülenebilir

### 5. Rol Hiyerarşisi
- `priority` alanı ile rol hiyerarşisi belirlenir
- Kullanıcı sadece kendi seviyesinden düşük rolleri yönetebilir
- Sistem rolleri hiyerarşi kontrolünden muaf

### 6. HasPermissions Trait
- `getPermissions()` - Permission tanımları
- `getDefaultRolePermissions()` - Rol-permission atamaları
- `collectAllPermissions()` - Tüm controller'lardan permission'ları toplar
- `collectAllRolePermissions()` - Tüm controller'lardan rol-permission atamalarını toplar

### 7. Dil ve Kullanıcı Ayarları Sistemi
- **UserSetting Model**: Her kullanıcı için otomatik olarak oluşturulan ayar kaydı
- **LocaleHelper**: Dil, tarih/saat formatı ve saat dilimi yönetimi için helper sınıfı
- **SetUserLocale Middleware**: API isteklerinde kullanıcının dil ve saat dilimi tercihlerini otomatik uygular
- **Desteklenen Diller**: tr, en, de, fr, es, it, ru, ar, zh, ja
- **Tarih Formatları**: d/m/Y, Y-m-d, m/d/Y, d.m.Y, Y.m.d
- **Saat Formatları**: H:i, H:i:s, h:i A, h:i:s A
- **Kullanım**: `LocaleHelper::formatDate()`, `LocaleHelper::formatTime()`, `LocaleHelper::formatDateTime()`

### 8. Çeviri Sistemi (Translation System)
- **Translation Model**: Veritabanı tabanlı çeviri sistemi
- **TranslationHelper**: Çeviri yönetimi için helper sınıfı
- **TranslationController**: Çeviri CRUD işlemleri
- **Cache Sistemi**: Çeviriler 24 saat cache'lenir (performans için)
- **Kullanım**: `TranslationHelper::trans()`, `TranslationHelper::getGroup()`, `TranslationHelper::getAll()`
- **Özellikler**: Parametreli çeviriler, grup bazlı organizasyon, otomatik cache temizleme
- **Values Formatı**: Tek bir key için birden fazla dilde çeviri ekleme/güncelleme (`{"key": "...", "values": {"tr": "...", "en": "..."}}`)
- **Bulk Operations**: Toplu ekleme, güncelleme ve silme işlemleri desteklenir

### 9. Dil Yönetimi Sistemi (Locale System)
- **Locale Model**: Veritabanı tabanlı dil yönetimi sistemi
- **LocaleController**: Dil CRUD işlemleri
- **LocaleHelper**: Dinamik dil kodları (`getSupportedLocaleCodes()`, `getSupportedLocales()`)
- **LocaleObserver**: Dil oluşturma için event handler'ları
- **LocaleSeeder**: Başlangıç dil verilerini ekler (tr, en, de, fr, es, it, ru, ar, zh, ja)
- **Özellikler**: 
  - Diller veritabanında saklanır ve dinamik olarak yönetilir
  - `sort_order` ile sıralama yapılabilir
  - `is_active` ile aktif/pasif kontrolü yapılabilir
  - TranslationController'da locale validasyonu dinamik olarak yapılır

---

## 🔧 Yapılandırma

### Ortam Değişkenleri (.env)

```env
# Sistem rol gizli anahtarı
ROLE_SYSTEM_SECRET=your-super-secret-key-here
```

### Config Dosyası

`config/app.php` içinde:
```php
'role_system_secret' => env('ROLE_SYSTEM_SECRET', null),
```

---

## 📝 Notlar

1. **İlk Kullanıcı:** İlk kayıt olan kullanıcı otomatik olarak `mgmt.superadmin` rolü alır
2. **Sonraki Kullanıcılar:** Sonraki tüm kullanıcılar `mgmt.user` rolü ile kayıt olur
3. **UserDetail:** Her kullanıcı oluşturulduğunda otomatik olarak boş bir UserDetail kaydı oluşturulur
4. **Permission Güncelleme:** Permission'ları güncellemek için controller'lardaki `getPermissions()` metodunu güncelleyin ve `RoleSeeder` çalıştırın
5. **Rol-Permission Güncelleme:** Rol-permission atamalarını güncellemek için controller'lardaki `getDefaultRolePermissions()` metodunu güncelleyin ve `RoleSeeder` çalıştırın

---

## 🚀 Hızlı Başlangıç

### 1. Migration'ları Çalıştır
```bash
php artisan migrate
```

### 2. RoleSeeder'ı Çalıştır
```bash
php artisan db:seed --class=RoleSeeder
```

### 3. TranslationSeeder'ı Çalıştır (Örnek çeviriler için)
```bash
php artisan db:seed --class=TranslationSeeder
```

### 4. İlk Kullanıcı Oluştur
```bash
POST /api/v1/register
{
  "email": "admin@example.com",
  "password": "password123"
}
```

İlk kullanıcı otomatik olarak `mgmt.superadmin` rolü alır.

---

**Son Güncelleme:** 2025-11-04 22:22 (Proje Yapısı bölümü tamamen güncellendi, tüm dosyalar listelendi)
**Versiyon:** 1.2.1

