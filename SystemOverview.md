# Laravel 12 API Projesi - Detaylı Analiz Dökümanı

**Versiyon:** 1.0.0  
**Son Güncelleme:** 11.11.2025 - 15:13  
**Laravel Versiyonu:** 12.0  
**PHP Versiyonu:** 8.2+

---

## İçindekiler

1. [Proje Genel Bakış](#proje-genel-bakış)
2. [Teknoloji Stack](#teknoloji-stack)
3. [Proje Yapısı](#proje-yapısı)
4. [Veritabanı Yapısı](#veritabanı-yapısı)
5. [Modeller (Models)](#modeller-models)
6. [Controller'lar](#controllerlar)
7. [API Endpoint'leri](#api-endpointleri)
8. [RBAC (Role-Based Access Control) Sistemi](#rbac-role-based-access-control-sistemi)
9. [Permission Sistemi](#permission-sistemi)
10. [Observer Pattern](#observer-pattern)
11. [Middleware'ler](#middlewareler)
12. [Helper Sınıfları](#helper-sınıfları)
13. [Service Sınıfları](#service-sınıfları)
14. [Çeviri Sistemi](#çeviri-sistemi)
15. [Yerelleştirme (Localization)](#yerelleştirme-localization)
16. [Para Birimi (Currency) Sistemi](#para-birimi-currency-sistemi)
17. [Audit Log Sistemi](#audit-log-sistemi)
18. [API Güvenliği](#api-güvenliği)
19. [Kurulum ve Yapılandırma](#kurulum-ve-yapılandırma)
20. [Geliştirme Rehberi](#geliştirme-rehberi)

---

## Proje Genel Bakış

Bu proje, Laravel 12 framework'ü kullanılarak geliştirilmiş kapsamlı bir RESTful API projesidir. Proje, çoklu kullanıcı yönetimi, rol bazlı erişim kontrolü (RBAC), organizasyon yönetimi, müşteri yönetimi, başvuru yönetimi, içerik yönetimi ve daha birçok özellik içermektedir.

### Temel Özellikler

- ✅ **RESTful API** - Tüm endpoint'ler REST standartlarına uygun
- ✅ **RBAC Sistemi** - Rol ve permission bazlı erişim kontrolü
- ✅ **Soft Delete** - Tüm kayıtlar soft delete ile silinir
- ✅ **UUID Kullanımı** - Public erişim gereken kayıtlar UUID ile erişilir
- ✅ **Çoklu Dil Desteği** - Çeviri sistemi ile çoklu dil desteği
- ✅ **Yerelleştirme** - Kullanıcı bazlı dil, saat dilimi ve tarih formatı
- ✅ **Para Birimi Yönetimi** - Para birimi dönüştürme ve yönetimi
- ✅ **Audit Log** - Tüm işlemler audit log ile kaydedilir
- ✅ **API Key Yönetimi** - API anahtarları ile erişim kontrolü
- ✅ **Entegrasyon Yönetimi** - Harici sistem entegrasyonları
- ✅ **Bildirim Sistemi** - Kullanıcı bildirimleri
- ✅ **Rapor Sistemi** - Rapor oluşturma ve yönetimi

---

## Teknoloji Stack

### Backend
- **Framework:** Laravel 12.0
- **PHP:** 8.2+
- **Veritabanı:** MySQL/MariaDB (InnoDB engine)
- **Authentication:** Laravel Sanctum 4.2
- **ORM:** Eloquent

### Development Tools
- **Code Formatter:** Laravel Pint 1.24
- **Testing:** Pest 3.8
- **Package Manager:** Composer

### Frontend Tools
- **Build Tool:** Vite 7.0.7
- **CSS Framework:** Tailwind CSS 4.0.0
- **Package Manager:** npm

---

## Proje Yapısı

```
example-app/
├── app/
│   ├── Helpers/              # Helper sınıfları
│   │   ├── LocaleHelper.php
│   │   └── TranslationHelper.php
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Api/          # API Controller'ları
│   │   ├── Middleware/       # Middleware'ler
│   │   └── Resources/        # API Resources
│   ├── Models/               # Eloquent Modelleri
│   ├── Observers/            # Model Observer'ları
│   ├── Providers/            # Service Provider'lar
│   ├── Services/             # Service sınıfları
│   └── Traits/               # Trait'ler
├── bootstrap/
│   └── app.php               # Laravel 12 bootstrap dosyası
├── config/                    # Yapılandırma dosyaları
├── database/
│   ├── migrations/           # Veritabanı migration'ları
│   └── seeders/              # Veritabanı seeder'ları
├── routes/
│   └── api/                  # Modüler API route dosyaları
├── storage/                   # Dosya depolama
├── tests/                     # Test dosyaları
└── public/                    # Public dosyalar
```

---

## Veritabanı Yapısı

### Temel Tablolar

#### Users (Kullanıcılar)
- `id` - Primary key
- `uuid` - Unique identifier (string, 36)
- `email` - E-posta adresi (unique)
- `password` - Şifre (hashed)
- `is_active` - Aktiflik durumu (boolean, default: true)
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Roles (Roller)
- `id` - Primary key
- `name` - Rol adı
- `slug` - Rol slug'ı (unique)
- `priority` - Öncelik seviyesi (düşük = yüksek yetki)
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Permissions (İzinler)
- `id` - Primary key
- `name` - İzin adı
- `slug` - İzin slug'ı (unique)
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Organizations (Organizasyonlar)
- `id` - Primary key
- `uuid` - Unique identifier (string, 36)
- `organization_code` - Organizasyon kodu
- `organization_name` - Organizasyon adı
- `legal_name` - Yasal ad
- `tax_number` - Vergi numarası
- `registration_number` - Kayıt numarası
- `sector` - Sektör
- `founded_date` - Kuruluş tarihi
- `status` - Durum
- `email`, `phone`, `website` - İletişim bilgileri
- `address`, `country`, `city` - Adres bilgileri
- `description` - Açıklama
- `logo_path` - Logo dosya yolu
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Customers (Müşteriler)
- `id` - Primary key
- `uuid` - Unique identifier (string, 36)
- `user_id` - Kullanıcı ID (foreign key)
- `company_name` - Şirket adı
- `legal_name` - Yasal ad
- `tax_number` - Vergi numarası
- `tax_office` - Vergi dairesi
- `registration_number` - Kayıt numarası
- `email`, `phone`, `website` - İletişim bilgileri
- `address`, `country`, `city`, `district`, `postal_code` - Adres bilgileri
- `general_manager` - Genel müdür
- `gm_phone`, `gm_email` - Genel müdür iletişim bilgileri
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Applications (Başvurular)
- `id` - Primary key
- `uuid` - Unique identifier (string, 36)
- `customer_id` - Müşteri ID (foreign key)
- `title` - Başlık
- `description` - Açıklama
- `type` - Tip
- `status` - Durum
- `application_date` - Başvuru tarihi
- `expiry_date` - Son geçerlilik tarihi
- `metadata` - JSON metadata
- `notes` - Notlar
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Application Subjects (Başvuru Konuları)
- `id` - Primary key
- `uuid` - Unique identifier (string, 36)
- `name` - Konu adı
- `description` - Açıklama
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Translations (Çeviriler)
- `id` - Primary key
- `key` - Çeviri anahtarı
- `locale` - Dil kodu
- `value` - Çeviri metni
- `group` - Grup adı
- `description` - Açıklama
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### Currencies (Para Birimleri)
- `id` - Primary key
- `code` - Para birimi kodu (unique)
- `name` - Para birimi adı
- `symbol` - Para birimi sembolü
- `name_en` - İngilizce ad
- `decimal_places` - Ondalık basamak sayısı
- `sort_order` - Sıralama
- `is_active` - Aktiflik durumu
- `description` - Açıklama
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

#### User Settings (Kullanıcı Ayarları)
- `id` - Primary key
- `user_id` - Kullanıcı ID (foreign key, unique)
- `locale` - Dil kodu
- `timezone` - Saat dilimi
- `date_format` - Tarih formatı
- `time_format` - Saat formatı
- `currency_id` - Para birimi ID (foreign key)
- `country_code` - Ülke kodu
- `notifications_email` - E-posta bildirimleri (boolean)
- `notifications_push` - Push bildirimleri (boolean)
- `notifications_sms` - SMS bildirimleri (boolean)
- `theme` - Tema
- `items_per_page` - Sayfa başına öğe sayısı
- `custom_settings` - JSON özel ayarlar
- `is_active` - Aktiflik durumu
- `deleted_at` - Soft delete timestamp
- `created_at`, `updated_at` - Timestamps

### İlişki Tabloları

#### user_roles
- `user_id` - Kullanıcı ID (foreign key)
- `role_id` - Rol ID (foreign key)

#### role_permissions
- `role_id` - Rol ID (foreign key)
- `permission_id` - İzin ID (foreign key)

#### organization_user
- `organization_id` - Organizasyon ID (foreign key)
- `user_id` - Kullanıcı ID (foreign key)
- `role` - Organizasyon içindeki rol
- `is_active` - Aktiflik durumu
- `joined_at` - Katılım tarihi
- `created_at`, `updated_at` - Timestamps

#### application_application_subject
- `application_id` - Başvuru ID (foreign key)
- `application_subject_id` - Başvuru konusu ID (foreign key)

---

## Modeller (Models)

### User Model
**Dosya:** `app/Models/User.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- `uuid` ile route key name
- `HasApiTokens` trait (Sanctum)
- `Notifiable` trait

**İlişkiler:**
- `detail()` - UserDetail (HasOne)
- `setting()` - UserSetting (HasOne)
- `roles()` - Role (BelongsToMany)
- `organizations()` - Organization (BelongsToMany)
- `customers()` - Customer (HasMany)
- `auditLogs()` - AuditLog (MorphMany)

**Metodlar:**
- `hasPermission(string $permissionSlug): bool` - İzin kontrolü
- `getHighestPriorityRole(): ?Role` - En yüksek öncelikli rol
- `getHighestPriorityLevel(): int` - En yüksek öncelik seviyesi
- `canManageRole(Role $targetRole): bool` - Rol yönetim kontrolü
- `canAssignRoleToUser(User $targetUser, Role $targetRole): bool` - Rol atama kontrolü
- `hasPermissionOrOwn(string $permission, ?User $resourceOwner = null): bool` - İzin veya sahiplik kontrolü
- `canManageOwn(string $basePermission, ?User $resourceOwner = null): bool` - Kendi kaynağını yönetme kontrolü
- `canManageResource(string $basePermission, ?User $resourceOwner = null): bool` - Kaynak yönetim kontrolü

### Role Model
**Dosya:** `app/Models/Role.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- Tablo adı: `user_role`

**İlişkiler:**
- `permissions()` - Permission (BelongsToMany)
- `users()` - User (BelongsToMany)
- `auditLogs()` - AuditLog (MorphMany)

**Metodlar:**
- `hasHigherPriorityThan(Role $otherRole): bool` - Daha yüksek öncelik kontrolü
- `hasLowerPriorityThan(Role $otherRole): bool` - Daha düşük öncelik kontrolü
- `hasHigherOrEqualPriorityThan(Role $otherRole): bool` - Daha yüksek veya eşit öncelik kontrolü
- `isSystemRole(): bool` - Sistem rolü kontrolü (system.toor, server.root)

### Permission Model
**Dosya:** `app/Models/Permission.php`

**Özellikler:**
- `SoftDeletes` trait kullanır

**İlişkiler:**
- `roles()` - Role (BelongsToMany)
- `auditLogs()` - AuditLog (MorphMany)

### Organization Model
**Dosya:** `app/Models/Organization.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- `uuid` ile route key name

**İlişkiler:**
- `users()` - User (BelongsToMany)

### Customer Model
**Dosya:** `app/Models/Customer.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- `uuid` ile route key name

**İlişkiler:**
- `user()` - User (BelongsTo)
- `detail()` - CustomerDetail (HasOne)
- `applications()` - Application (HasMany)
- `auditLogs()` - AuditLog (MorphMany)

### Application Model
**Dosya:** `app/Models/Application.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- `uuid` ile route key name
- `metadata` JSON cast

**İlişkiler:**
- `customer()` - Customer (BelongsTo)
- `subjects()` - ApplicationSubject (BelongsToMany)
- `auditLogs()` - AuditLog (MorphMany)

**Accessor:**
- `getUserAttribute(): ?User` - Müşteri üzerinden kullanıcıya erişim

### Translation Model
**Dosya:** `app/Models/Translation.php`

**Özellikler:**
- `SoftDeletes` trait kullanır

**Scopes:**
- `scopeActive($query)` - Aktif çeviriler
- `scopeLocale($query, string $locale)` - Belirli dil için çeviriler
- `scopeGroup($query, string $group)` - Belirli grup için çeviriler
- `scopeKey($query, string $key)` - Belirli key için çeviriler

**İlişkiler:**
- `auditLogs()` - AuditLog (MorphMany)

### Currency Model
**Dosya:** `app/Models/Currency.php`

**Özellikler:**
- `SoftDeletes` trait kullanır

**Scopes:**
- `scopeActive($query)` - Aktif para birimleri

**İlişkiler:**
- `customerDetails()` - CustomerDetail (HasMany)
- `userSettings()` - UserSetting (HasMany)
- `auditLogs()` - AuditLog (MorphMany)

**Static Metodlar:**
- `getActiveCodes(): array` - Aktif para birimi kodları
- `getAllCodes(): array` - Tüm para birimi kodları
- `getDefaultCurrency(): ?self` - Varsayılan para birimi (TRY)
- `getDefaultCurrencyId(): ?int` - Varsayılan para birimi ID'si

### UserSetting Model
**Dosya:** `app/Models/UserSetting.php`

**Özellikler:**
- `SoftDeletes` trait kullanır
- Tablo adı: `user_settings`
- `custom_settings` JSON cast

**İlişkiler:**
- `user()` - User (BelongsTo)
- `currencyModel()` - Currency (BelongsTo)

**Metodlar:**
- `getLocale(): string` - Dil tercihi
- `getTimezone(): string` - Saat dilimi
- `getUserDateFormat(): string` - Tarih formatı
- `getTimeFormat(): string` - Saat formatı
- `getCurrency(): string` - Para birimi kodu
- `getCountryCode(): ?string` - Ülke kodu

---

## Controller'lar

### API Controller'ları

Tüm API controller'ları `app/Http/Controllers/Api/` klasöründe bulunur ve `HasPermissions` trait'ini kullanır.

#### UserController
**Dosya:** `app/Http/Controllers/Api/UserController.php`

**Endpoint'ler:**
- `GET /api/v1/users` - Kullanıcı listesi
- `GET /api/v1/users/{uuid}` - Kullanıcı detayı
- `POST /api/v1/users` - Yeni kullanıcı oluştur
- `PUT/PATCH /api/v1/users/{uuid}` - Kullanıcı güncelle
- `DELETE /api/v1/users/{uuid}` - Kullanıcı sil (soft delete)
- `POST /api/v1/users/{uuid}/restore` - Kullanıcı geri yükle

**Permission'lar:**
- `users.manage` - Tam yetki
- `users.view` / `users.view.own` - Görüntüleme
- `users.create` / `users.create.own` - Oluşturma
- `users.update` / `users.update.own` - Güncelleme
- `users.delete` / `users.delete.own` - Silme
- `users.restore` / `users.restore.own` - Geri yükleme

#### OrganizationController
**Dosya:** `app/Http/Controllers/Api/OrganizationController.php`

**Endpoint'ler:**
- `GET /api/v1/organizations` - Organizasyon listesi
- `GET /api/v1/organizations/{uuid}` - Organizasyon detayı
- `POST /api/v1/organizations` - Yeni organizasyon oluştur
- `PUT/PATCH /api/v1/organizations/{uuid}` - Organizasyon güncelle
- `DELETE /api/v1/organizations/{uuid}` - Organizasyon sil (soft delete)
- `POST /api/v1/organizations/{uuid}/restore` - Organizasyon geri yükle

#### CustomerController
**Dosya:** `app/Http/Controllers/Api/CustomerController.php`

**Endpoint'ler:**
- `GET /api/v1/customers` - Müşteri listesi
- `GET /api/v1/customers/{uuid}` - Müşteri detayı
- `POST /api/v1/customers` - Yeni müşteri oluştur
- `PUT/PATCH /api/v1/customers/{uuid}` - Müşteri güncelle
- `DELETE /api/v1/customers/{uuid}` - Müşteri sil (soft delete)
- `POST /api/v1/customers/{uuid}/restore` - Müşteri geri yükle

#### ApplicationController
**Dosya:** `app/Http/Controllers/Api/ApplicationController.php`

**Endpoint'ler:**
- `GET /api/v1/applications` - Başvuru listesi
- `GET /api/v1/applications/{uuid}` - Başvuru detayı
- `POST /api/v1/applications` - Yeni başvuru oluştur
- `PUT/PATCH /api/v1/applications/{uuid}` - Başvuru güncelle
- `DELETE /api/v1/applications/{uuid}` - Başvuru sil (soft delete)
- `POST /api/v1/applications/{uuid}/restore` - Başvuru geri yükle

#### TranslationController
**Dosya:** `app/Http/Controllers/Api/TranslationController.php`

**Endpoint'ler:**
- `GET /api/v1/translations/{key}` - Çeviri metni (Public)
- `GET /api/v1/translations/group/{group}` - Grup çevirileri (Public)
- `GET /api/v1/translations` - Tüm çeviriler (Public)
- `GET /api/v1/admin/translations` - Çeviri listesi (Protected)
- `POST /api/v1/admin/translations` - Çeviri oluştur (Protected)
- `PUT /api/v1/admin/translations/{translation}` - Çeviri güncelle (Protected)
- `DELETE /api/v1/admin/translations/{translation}` - Çeviri sil (Protected)
- `POST /api/v1/admin/translations/{id}/restore` - Çeviri geri yükle (Protected)

#### CurrencyController
**Dosya:** `app/Http/Controllers/Api/CurrencyController.php`

**Endpoint'ler:**
- `GET /api/v1/currencies` - Para birimi listesi
- `GET /api/v1/currencies/{id}` - Para birimi detayı
- `POST /api/v1/currencies` - Yeni para birimi oluştur
- `PUT/PATCH /api/v1/currencies/{id}` - Para birimi güncelle
- `DELETE /api/v1/currencies/{id}` - Para birimi sil (soft delete)
- `POST /api/v1/currencies/{id}/restore` - Para birimi geri yükle

#### Diğer Controller'lar
- **AuthController** - Kimlik doğrulama (login, register, logout, me)
- **UserDetailController** - Kullanıcı detayları yönetimi
- **UserSettingController** - Kullanıcı ayarları yönetimi
- **RolePermissionController** - Rol ve izin yönetimi
- **ContentController** - İçerik yönetimi
- **ReportController** - Rapor yönetimi
- **AuditController** - Denetim log yönetimi
- **SessionController** - Oturum yönetimi
- **NotificationController** - Bildirim yönetimi
- **ApiKeyController** - API anahtarı yönetimi
- **IntegrationController** - Entegrasyon yönetimi
- **SystemController** - Sistem yönetimi
- **LocaleController** - Dil yönetimi
- **ApplicationSubjectController** - Başvuru konusu yönetimi
- **CustomerDetailController** - Müşteri detayları yönetimi
- **PublicUserController** - Public kullanıcı bilgileri

---

## API Endpoint'leri

### Authentication Endpoints
**Base URL:** `/api/v1/auth`

- `POST /api/v1/auth/login` - Giriş yap
- `POST /api/v1/auth/register` - Kayıt ol
- `POST /api/v1/auth/logout` - Çıkış yap
- `GET /api/v1/auth/me` - Mevcut kullanıcı bilgileri

### User Endpoints
**Base URL:** `/api/v1/users`

- `GET /api/v1/users` - Kullanıcı listesi (pagination, search, filter)
- `GET /api/v1/users/{uuid}` - Kullanıcı detayı
- `POST /api/v1/users` - Yeni kullanıcı oluştur
- `PUT /api/v1/users/{uuid}` - Kullanıcı güncelle
- `PATCH /api/v1/users/{uuid}` - Kullanıcı güncelle (kısmi)
- `DELETE /api/v1/users/{uuid}` - Kullanıcı sil (soft delete)
- `POST /api/v1/users/{uuid}/restore` - Kullanıcı geri yükle

### Organization Endpoints
**Base URL:** `/api/v1/organizations`

- `GET /api/v1/organizations` - Organizasyon listesi
- `GET /api/v1/organizations/{uuid}` - Organizasyon detayı
- `POST /api/v1/organizations` - Yeni organizasyon oluştur
- `PUT /api/v1/organizations/{uuid}` - Organizasyon güncelle
- `PATCH /api/v1/organizations/{uuid}` - Organizasyon güncelle (kısmi)
- `DELETE /api/v1/organizations/{uuid}` - Organizasyon sil (soft delete)
- `POST /api/v1/organizations/{uuid}/restore` - Organizasyon geri yükle

### Customer Endpoints
**Base URL:** `/api/v1/customers`

- `GET /api/v1/customers` - Müşteri listesi
- `GET /api/v1/customers/{uuid}` - Müşteri detayı
- `POST /api/v1/customers` - Yeni müşteri oluştur
- `PUT /api/v1/customers/{uuid}` - Müşteri güncelle
- `PATCH /api/v1/customers/{uuid}` - Müşteri güncelle (kısmi)
- `DELETE /api/v1/customers/{uuid}` - Müşteri sil (soft delete)
- `POST /api/v1/customers/{uuid}/restore` - Müşteri geri yükle

### Application Endpoints
**Base URL:** `/api/v1/applications`

- `GET /api/v1/applications` - Başvuru listesi
- `GET /api/v1/applications/{uuid}` - Başvuru detayı
- `POST /api/v1/applications` - Yeni başvuru oluştur
- `PUT /api/v1/applications/{uuid}` - Başvuru güncelle
- `PATCH /api/v1/applications/{uuid}` - Başvuru güncelle (kısmi)
- `DELETE /api/v1/applications/{uuid}` - Başvuru sil (soft delete)
- `POST /api/v1/applications/{uuid}/restore` - Başvuru geri yükle

### Translation Endpoints
**Base URL:** `/api/v1/translations`

**Public Endpoints:**
- `GET /api/v1/translations/{key}` - Çeviri metni
- `GET /api/v1/translations/group/{group}` - Grup çevirileri
- `GET /api/v1/translations` - Tüm çeviriler

**Protected Endpoints:**
- `GET /api/v1/admin/translations` - Çeviri listesi
- `POST /api/v1/admin/translations` - Çeviri oluştur
- `PUT /api/v1/admin/translations/{translation}` - Çeviri güncelle
- `DELETE /api/v1/admin/translations/{translation}` - Çeviri sil
- `POST /api/v1/admin/translations/{id}/restore` - Çeviri geri yükle

### Currency Endpoints
**Base URL:** `/api/v1/currencies`

- `GET /api/v1/currencies` - Para birimi listesi
- `GET /api/v1/currencies/{id}` - Para birimi detayı
- `POST /api/v1/currencies` - Yeni para birimi oluştur
- `PUT /api/v1/currencies/{id}` - Para birimi güncelle
- `PATCH /api/v1/currencies/{id}` - Para birimi güncelle (kısmi)
- `DELETE /api/v1/currencies/{id}` - Para birimi sil (soft delete)
- `POST /api/v1/currencies/{id}/restore` - Para birimi geri yükle

---

## RBAC (Role-Based Access Control) Sistemi

### Rol Hiyerarşisi

Roller öncelik (priority) seviyesine göre sıralanır. Düşük priority değeri = yüksek yetki.

1. **system.toor** (priority: 1) - Sistem Seviyesinde erişim
   - Gizli anahtar ile erişim (`ROLE_SYSTEM_SECRET`)
   - Tüm permission'lara otomatik erişim
   - Tüm permission kontrollerini bypass eder

2. **server.root** (priority: 2) - Sunucu Seviyesinde Erişim
   - Gizli anahtar ile erişim (`ROLE_SYSTEM_SECRET`)
   - Tüm permission'lara otomatik erişim
   - Tüm permission kontrollerini bypass eder

3. **mgmt.superadmin** (priority: 10) - Tam Yetkili Erişim
   - Tüm yönetim işlemleri
   - Sistem yönetimi hariç bazı kritik işlemler

4. **mgmt.admin** (priority: 20) - Yönetici seviyesinde erişim
   - Yönetim yetkileri
   - Bazı kritik sistem işlemleri hariç

5. **mgmt.moderator** (priority: 30) - Moderasyon seviyesinde erişim
   - Moderasyon ve içerik yönetimi odaklı

6. **mgmt.editor** (priority: 40) - Editör seviyesinde erişim
   - İçerik oluşturma ve düzenleme odaklı

7. **mgmt.user** (priority: 50) - Kullanıcı seviyesinde erişim
   - Standart kullanıcı yetkileri
   - Sadece kendi kaynaklarını yönetebilir

8. **mgmt.anonymous** (priority: 60) - Anonim seviyede erişim
   - Sadece genel görüntüleme

### Sistem Rolleri

Sistem rolleri (`system.toor`, `server.root`) özel güvenlik önlemleri ile korunur:

- Gizli anahtar zorunlu (`ROLE_SYSTEM_SECRET` environment variable)
- Header: `X-Role-Secret` veya body: `secret` parametresi
- Tüm permission kontrollerini otomatik bypass eder
- RoleSeeder'da otomatik olarak tüm permission'lar atanır

---

## Permission Sistemi

### Permission Tanımlama

Permission'lar controller'larda `getPermissions()` static metodu ile tanımlanır:

```php
public static function getPermissions(): array
{
    return [
        ['slug' => 'module.manage', 'name' => 'Modül Yönetimi - Tam Yetki'],
        ['slug' => 'module.view', 'name' => 'Modül Görüntüleme'],
        ['slug' => 'module.view.own', 'name' => 'Kendi Modül Görüntüleme'],
        // ...
    ];
}
```

### Rol-Permission Atamaları

Rol-permission atamaları controller'larda `getDefaultRolePermissions()` static metodu ile tanımlanır:

```php
public static function getDefaultRolePermissions(): array
{
    return [
        'mgmt.superadmin' => [
            'module.manage', 'module.view', 'module.view.own',
            // ...
        ],
        'mgmt.user' => [
            'module.view.own', 'module.create.own',
            // ...
        ],
    ];
}
```

### Permission Kontrolü

#### Controller İçinde Permission Kontrolü

```php
// Genel permission kontrolü
if (! $user->hasPermission('module.view')) {
    return response()->json(['message' => 'Forbidden'], 403);
}

// Genel veya .own permission kontrolü
if (! $user->hasPermissionOrOwn('module.view')) {
    return response()->json(['message' => 'Forbidden'], 403);
}

// Kaynak sahibi kontrolü ile permission kontrolü
if (! $user->canManageResource('module.view', $resource->user)) {
    return response()->json(['message' => 'Forbidden'], 403);
}
```

#### Middleware ile Permission Kontrolü

```php
Route::middleware(['auth:sanctum', 'permission:module.view'])->group(function () {
    // ...
});
```

### .own Permission'ları

`.own` permission'ları, kullanıcının sadece kendi kaynaklarını yönetebilmesini sağlar:

- `module.view.own` - Sadece kendi kaynaklarını görüntüleme
- `module.create.own` - Sadece kendi kaynaklarını oluşturma
- `module.update.own` - Sadece kendi kaynaklarını güncelleme
- `module.delete.own` - Sadece kendi kaynaklarını silme
- `module.restore.own` - Sadece kendi kaynaklarını geri yükleme

### Permission Toplama

Permission'lar `HasPermissions` trait'inin `collectAllPermissions()` metodu ile otomatik olarak toplanır:

```php
$permissions = \App\Traits\HasPermissions::collectAllPermissions();
```

Rol-permission atamaları `collectAllRolePermissions()` metodu ile toplanır:

```php
$rolePermissions = \App\Traits\HasPermissions::collectAllRolePermissions();
```

RoleSeeder bu metodları kullanarak otomatik olarak permission'ları ve atamaları veritabanına kaydeder.

---

## Observer Pattern

Tüm modeller için Observer'lar `app/Observers/` klasöründe bulunur ve `AppServiceProvider` içinde kayıtlıdır.

### Observer Görevleri

1. **UUID Oluşturma** - Model oluşturulurken otomatik UUID oluşturma
2. **İlişkili Kayıt Oluşturma** - User oluşturulurken UserDetail ve UserSetting oluşturma
3. **Varsayılan Değer Atama** - Varsayılan değerlerin atanması

### Örnek Observer

```php
class UserObserver
{
    public function creating(User $user): void
    {
        if (empty($user->uuid)) {
            $user->uuid = (string) Str::uuid();
        }
    }

    public function created(User $user): void
    {
        // UserDetail oluştur
        $user->detail()->create([]);
        
        // UserSetting oluştur
        $user->setting()->create([
            'locale' => 'tr',
            'timezone' => 'Europe/Istanbul',
            'currency_id' => Currency::getDefaultCurrencyId(),
        ]);
    }
}
```

---

## Middleware'ler

### SetUserLocale Middleware
**Dosya:** `app/Http/Middleware/SetUserLocale.php`

**Görev:**
- Kullanıcının dil tercihini uygular
- Kullanıcının saat dilimini uygular
- Laravel'in locale sistemini günceller

**Kullanım:**
- API route'larında otomatik olarak çalışır (`bootstrap/app.php` içinde kayıtlı)

### PermissionMiddleware
**Dosya:** `app/Http/Middleware/PermissionMiddleware.php`

**Görev:**
- Permission kontrolü yapar
- Sistem rolleri için bypass yapar
- `.own` permission kontrolü yapar

**Kullanım:**
```php
Route::middleware(['auth:sanctum', 'permission:module.view'])->group(function () {
    // ...
});
```

---

## Helper Sınıfları

### LocaleHelper
**Dosya:** `app/Helpers/LocaleHelper.php`

**Metodlar:**
- `setUserLocale(?User $user): void` - Kullanıcının dil tercihini uygular
- `setUserTimezone(?User $user): void` - Kullanıcının saat dilimini uygular
- `formatDate($date, ?User $user, ?string $format): string` - Tarih formatlama
- `formatTime($time, ?User $user, ?string $format): string` - Saat formatlama
- `formatDateTime($datetime, ?User $user): string` - Tarih ve saat formatlama
- `getSupportedLocales(): array` - Desteklenen dilleri döndürür
- `getSupportedDateFormats(): array` - Desteklenen tarih formatlarını döndürür
- `getSupportedTimeFormats(): array` - Desteklenen saat formatlarını döndürür

### TranslationHelper
**Dosya:** `app/Helpers/TranslationHelper.php`

**Metodlar:**
- `trans(string $key, array $replace = [], ?string $locale = null, ?string $default = null): string` - Çeviri metni
- `getGroup(string $group, ?string $locale = null): array` - Grup çevirileri
- `getAll(?string $locale = null): array` - Tüm çeviriler
- `getCurrentLocale(): string` - Mevcut dil tercihi
- `clearCache(?string $locale = null, ?string $key = null): void` - Cache temizleme
- `has(string $key, ?string $locale = null): bool` - Çeviri kontrolü

**Cache:**
- Çeviriler 24 saat cache'lenir (performans için)
- Cache key formatı: `translation:{locale}:{key}`

---

## Service Sınıfları

### CurrencyService
**Dosya:** `app/Services/CurrencyService.php`

**Metodlar:**
- `getExchangeRate(string $fromCurrency, string $toCurrency, ?bool $isTCMB = null): ?float` - Döviz kuru
- `convert(float $amount, string $fromCurrency, string $toCurrency, ?bool $isTCMB = null): ?float` - Para birimi dönüştürme
- `isSupported(string $currency): bool` - Para birimi desteği kontrolü
- `getSupportedCurrencies(): array` - Desteklenen para birimleri
- `getSymbol(string $currency): string` - Para birimi sembolü
- `format(float $amount, string $currency, int $decimals = 2): string` - Para birimi formatlama

**Özellikler:**
- TCMB (Türkiye Cumhuriyet Merkez Bankası) API desteği
- ExchangeRate-API desteği
- Fallback oranları
- Cache desteği (1 saat)

---

## Çeviri Sistemi

### Translation Model

Çeviriler `translations` tablosunda saklanır:
- `key` - Çeviri anahtarı (örn: 'pages.home')
- `locale` - Dil kodu (örn: 'tr', 'en')
- `value` - Çeviri metni
- `group` - Grup adı (örn: 'pages', 'menu')
- `description` - Açıklama
- `is_active` - Aktiflik durumu

### TranslationHelper Kullanımı

```php
use App\Helpers\TranslationHelper;

// Tek bir çeviri
$title = TranslationHelper::trans('pages.home');

// Parametreli çeviri
$message = TranslationHelper::trans('messages.welcome', ['name' => $userName]);

// Belirli bir dil için çeviri
$title = TranslationHelper::trans('pages.home', [], 'en');

// Grup çevirileri
$menuItems = TranslationHelper::getGroup('menu');

// Tüm çeviriler
$allTranslations = TranslationHelper::getAll();
```

### Çeviri Cache

Çeviriler 24 saat cache'lenir. Cache temizlemek için:

```php
TranslationHelper::clearCache(); // Tüm cache
TranslationHelper::clearCache('tr'); // Belirli dil için
TranslationHelper::clearCache('tr', 'pages.home'); // Belirli çeviri için
```

---

## Yerelleştirme (Localization)

### UserSetting Model

Kullanıcı ayarları `user_settings` tablosunda saklanır:
- `locale` - Dil kodu (varsayılan: 'tr')
- `timezone` - Saat dilimi (varsayılan: 'Europe/Istanbul')
- `date_format` - Tarih formatı (varsayılan: 'd/m/Y')
- `time_format` - Saat formatı (varsayılan: 'H:i')
- `currency_id` - Para birimi ID'si

### LocaleHelper Kullanımı

```php
use App\Helpers\LocaleHelper;

// Tarih formatlama
$formattedDate = LocaleHelper::formatDate($date, $user);

// Saat formatlama
$formattedTime = LocaleHelper::formatTime($time, $user);

// Tarih ve saat formatlama
$formattedDateTime = LocaleHelper::formatDateTime($datetime, $user);

// Desteklenen diller
$locales = LocaleHelper::getSupportedLocales();
```

### SetUserLocale Middleware

Her API isteğinde otomatik olarak:
1. Kullanıcının dil tercihi uygulanır
2. Kullanıcının saat dilimi uygulanır
3. Laravel'in locale sistemi güncellenir

---

## Para Birimi (Currency) Sistemi

### Currency Model

Para birimleri `currencies` tablosunda saklanır:
- `code` - Para birimi kodu (unique)
- `name` - Para birimi adı
- `symbol` - Para birimi sembolü
- `name_en` - İngilizce ad
- `decimal_places` - Ondalık basamak sayısı
- `sort_order` - Sıralama
- `is_active` - Aktiflik durumu

### CurrencyService Kullanımı

```php
use App\Services\CurrencyService;

$currencyService = new CurrencyService();

// Döviz kuru
$rate = $currencyService->getExchangeRate('USD', 'TRY');

// Para birimi dönüştürme
$converted = $currencyService->convert(100, 'USD', 'TRY');

// Para birimi formatlama
$formatted = $currencyService->format(1000.50, 'TRY'); // "1.000,50 ₺"

// Para birimi sembolü
$symbol = $currencyService->getSymbol('USD'); // "$"
```

### TCMB Desteği

TRY ile ilgili dönüştürmeler için otomatik olarak TCMB API'si kullanılır:
- TRY → Diğer para birimleri
- Diğer para birimleri → TRY
- TRY olmayan para birimleri arası dönüştürme (TRY üzerinden)

---

## Audit Log Sistemi

### AuditLog Model

Denetim logları `audit_logs` tablosunda saklanır:
- `model_type` - Model sınıfı (polymorphic)
- `model_id` - Model ID'si
- `user_id` - Kullanıcı ID'si
- `action` - İşlem tipi (create, update, delete, restore)
- `old_values` - Eski değerler (JSON)
- `new_values` - Yeni değerler (JSON)
- `ip_address` - IP adresi
- `user_agent` - User agent
- `created_at` - Oluşturulma tarihi

### Polymorphic İlişki

Tüm modeller `auditLogs()` ilişkisi ile audit log'lara erişebilir:

```php
$user->auditLogs; // Kullanıcının tüm audit log'ları
$organization->auditLogs; // Organizasyonun tüm audit log'ları
```

---

## API Güvenliği

### Authentication

- **Laravel Sanctum** - Token-based authentication
- Tüm protected endpoint'ler `auth:sanctum` middleware'i ile korunur

### Authorization

- **RBAC Sistemi** - Rol ve permission bazlı erişim kontrolü
- **Permission Kontrolü** - Controller metodları içinde veya middleware ile
- **Sistem Rolleri** - Gizli anahtar ile korunur

### Sistem Rolleri Güvenliği

Sistem rolleri (`system.toor`, `server.root`) için:
- Gizli anahtar zorunlu (`ROLE_SYSTEM_SECRET`)
- Header: `X-Role-Secret` veya body: `secret` parametresi
- Tüm permission kontrollerini otomatik bypass eder

### Soft Delete

- Tüm kayıtlar soft delete ile silinir
- Silinen kayıtlar `deleted_at` kolonu ile işaretlenir
- `with_trashed` parametresi ile silinen kayıtlar görüntülenebilir
- `restore` endpoint'i ile kayıtlar geri yüklenebilir

### UUID Kullanımı

Public erişim gereken kayıtlar UUID ile erişilir:
- `uuid` kolonu (string, 36, unique)
- `getRouteKeyName()` metodu ile route key name olarak kullanılır
- Observer ile otomatik oluşturulur

---

## Kurulum ve Yapılandırma

### Gereksinimler

- PHP 8.2+
- Composer
- MySQL/MariaDB
- Node.js ve npm

### Kurulum Adımları

1. **Projeyi klonlayın:**
```bash
git clone <repository-url>
cd example-app
```

2. **Bağımlılıkları yükleyin:**
```bash
composer install
npm install
```

3. **Environment dosyasını oluşturun:**
```bash
cp .env.example .env
php artisan key:generate
```

4. **Veritabanı yapılandırması:**
`.env` dosyasında veritabanı bilgilerini güncelleyin:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=example_app
DB_USERNAME=root
DB_PASSWORD=
```

5. **Migration'ları çalıştırın:**
```bash
php artisan migrate
```

6. **Seeder'ları çalıştırın:**
```bash
php artisan db:seed --class=RoleSeeder
php artisan db:seed --class=LocaleSeeder
php artisan db:seed --class=CurrencySeeder
php artisan db:seed --class=TranslationSeeder
```

7. **Frontend build:**
```bash
npm run build
# veya development için
npm run dev
```

### Environment Variables

Önemli environment değişkenleri:

```env
# Uygulama
APP_NAME="Example App"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

# Veritabanı
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=example_app
DB_USERNAME=root
DB_PASSWORD=

# Sistem Rolleri (Gizli Anahtar)
ROLE_SYSTEM_SECRET=your-secret-key-here

# Cache
CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_CONNECTION=sync
```

---

## Geliştirme Rehberi

### Yeni Modül Ekleme

1. **Migration Oluştur:**
```bash
php artisan make:migration create_module_table --no-interaction
```

2. **Model Oluştur:**
```bash
php artisan make:model Module --no-interaction
```

3. **Observer Oluştur:**
```bash
php artisan make:observer ModuleObserver --model=Module --no-interaction
```

4. **Controller Oluştur:**
```bash
php artisan make:controller Api/ModuleController --no-interaction
```

5. **Route Dosyası Oluştur:**
`routes/api/module.php` dosyasını oluşturun ve `routes/api.php` içinde require edin.

6. **Permission Tanımları:**
Controller içinde `getPermissions()` ve `getDefaultRolePermissions()` metodlarını ekleyin.

7. **HasPermissions Trait'ine Ekle:**
`app/Traits/HasPermissions.php` içinde controller'ı ekleyin.

8. **Observer'ı Kaydet:**
`app/Providers/AppServiceProvider.php` içinde observer'ı kaydedin.

9. **Migration ve Seeder Çalıştır:**
```bash
php artisan migrate
php artisan db:seed --class=RoleSeeder
```

### Kod Formatlama

Laravel Pint kullanarak kod formatlama:

```bash
vendor/bin/pint --dirty
```

### Test Yazma

Pest kullanarak test yazma:

```bash
php artisan make:test --pest ModuleTest
```

Test çalıştırma:

```bash
php artisan test
php artisan test --filter=ModuleTest
```

### API Dokümantasyonu

API endpoint'leri için detaylı dokümantasyon:
- `docs/ApiResources.md` - API Resource'ları
- `docs/DevelopmentGuide.md` - Geliştirme rehberi

---

## Sonuç

Bu proje, Laravel 12 framework'ü kullanılarak geliştirilmiş kapsamlı bir RESTful API projesidir. Proje, modern yazılım geliştirme prensipleri, güvenlik standartları ve best practice'ler ile geliştirilmiştir.

### Öne Çıkan Özellikler

- ✅ Modüler yapı
- ✅ RBAC sistemi
- ✅ Permission yönetimi
- ✅ Çoklu dil desteği
- ✅ Yerelleştirme
- ✅ Para birimi yönetimi
- ✅ Audit log sistemi
- ✅ Soft delete
- ✅ UUID kullanımı
- ✅ Observer pattern
- ✅ Service layer
- ✅ Helper sınıfları

### Geliştirme Notları

- Tüm kodlar Laravel 12 standartlarına uygundur
- PSR-12 kod formatı kullanılır
- Türkçe yorumlar kullanılır
- Modüler route yapısı kullanılır
- Controller bazlı permission tanımları kullanılır
- Observer pattern ile otomatik işlemler yapılır

---

**Döküman Versiyonu:** 1.0.0  
**Son Güncelleme:** 11.11.2025 - 15:13  
**Hazırlayan:** AI Assistant

