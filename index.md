# Gizlilik Siyasəti

Bu sənəd «Loan Docs» Chrome genişlənməsinin hansı məlumatı topladığını,
harada saxladığını və kimə ötürdüyünü izah edir.

> ⚠️ **Ən qısa cavab:** kredit müştərilərinizin şəxsi məlumatı **kompüterinizi
> tərk etmir**. Buluda yalnız lisenziyanın işləməsi üçün lazım olan minimum
> gedir: işçinin Google e-poçtu və cihazın adı.

### Sənəddəki iki cür ifadə

Bölmələr iki qrupa ayrılır və başlıqda göstərilir:

- **Ölçülmüş fakt** — proqramın necə işlədiyini təsvir edir. Kodun özündən
  yoxlanılıb; proqram dəyişməsə, dəyişmir. Bölmə **1–7**.
- **Təchizatçının siyasəti** — proqramın necə işlədiyindən deyil, təchizatçının
  öhdəliyindən asılıdır. Kodda yoxlanıla bilməz. Bölmə **8–9**.

Ayrım qəsdəndir: birincisi yoxlanıla bilən, ikincisi verilən sözdür.

---

## 1. Toplanan məlumat — ölçülmüş fakt

### 1.1 Buluda göndərilən — cəmi iki şəxsi məlumat

| Məlumat | Nə üçün | Harada saxlanılır |
|---|---|---|
| İşçinin **Google e-poçtu** | hesabı lisenziyaya bağlamaq | Firestore, `users/{uid}` |
| İşçinin yazdığı **cihaz adı** | «hansı kompüterdir» sualına cavab, cihaz limiti | Firestore, `users/{uid}.devices` |

Bunlardan başqa buluda **şəxsi olmayan** texniki qeydlər gedir:

- şirkət identifikatoru (`companyId`), hesabın aktivliyi, icazə verilən cihaz
  sayı, istifadəçinin rolu (`admin` və ya `staff`);
- cihaz üçün: təsadüfi identifikator, ilk və sonuncu görünmə vaxtı;
- aktivləşdirmə kodunun istifadə vəziyyəti: kodun işlədildiyi, kim tərəfindən
  və nə vaxt;
- **yalnız admin rolundakı istifadəçi üçün:** şirkətin mərkəzləşdirilmiş
  ayarları — rekvizitlər, faiz dərəcələri, cərimə şərtləri. Bunlar
  konfiqurasiyadır, müştəri məlumatı deyil.

### 1.2 Buluda GETMƏYƏN — kredit müştərilərinin bütün məlumatı

Genişlənmə sənəd hazırlamaq üçün müştəri haqqında geniş məlumat saxlayır:
ad, doğum tarixi və yeri, şəxsiyyət vəsiqəsinin məlumatları, FİN, qeydiyyat
və faktiki ünvan, ailə vəziyyəti, iş yeri, gəlir, telefon nömrələri, kredit
şərtləri, girov və lizinq obyektinin təsviri.

**Bunların heç biri şəbəkəyə göndərilmir.** Hamısı brauzerin daxili bazasında
(IndexedDB) — yəni sizin kompüterinizdə qalır.

---

## 2. Kompüterdə nə saxlanılır — ölçülmüş fakt

| Yer | Nə saxlanılır |
|---|---|
| **IndexedDB** — `loans` | kredit/lizinq qeydləri, sənədə düşən dondurulmuş nüsxə |
| **IndexedDB** — `customers` | müştəri kartları |
| **IndexedDB** — `settings` | şirkət rekvizitləri, tariflər, məntəqələr, işçi siyahıları |
| **`chrome.storage.local`** | cihaz identifikatoru və adı, lisenziya güzgüsü, aktivləşdirmə vəziyyəti |
| **`chrome.storage.session`** | açıq tabın nömrəsi; brauzer bağlananda itir |

⚠️ **`chrome.storage.sync` istifadə edilmir.** Əks halda məlumat Google
hesabınız vasitəsilə cihazlar arasında Google serverlərinə köçürülərdi.

---

## 3. Google ilə giriş — ölçülmüş fakt

Giriş yalnız Google hesabı ilədir. **Şifrə istənilmir və saxlanılmır** —
genişlənmədə şifrə sahəsi ümumiyyətlə yoxdur.

Google razılıq ekranında `openid email profile` icazələri istənilir. Bu, Google-un
standart dəstidir və ekranda «adınızı və profil şəklinizi görmək» kimi görünə
bilər. ⚠️ **Adınız və profil şəkliniz saxlanılmır** — genişlənmə yalnız
**e-poçt ünvanını** oxuyur və saxlayır.

---

## 4. Şəbəkə əlaqələri — ölçülmüş fakt

Genişlənmə yalnız üç Google ünvanı ilə əlaqə qurur:

```
identitytoolkit.googleapis.com     — giriş
securetoken.googleapis.com         — sessiyanın yenilənməsi
firestore.googleapis.com           — lisenziya və mərkəzi ayarlar
```

Başqa heç bir ünvana sorğu getmir; brauzer bunu texniki olaraq bloklayır.

- **Analitika yoxdur.** İstifadə statistikası, davranış izlənməsi, hansı
  düymələrə basdığınız — heç biri toplanmır.
- **Telemetriya yoxdur.** Xəta hesabatları avtomatik göndərilmir.
- **Üçüncü tərəf skripti yoxdur.** Reklam şəbəkəsi, izləyici piksel və xarici
  kitabxana yüklənmir; təhlükəsizlik siyasəti bunu qadağan edir.
- **Şriftlər daxilidir** — işləmə zamanı internetdən yüklənmir.

Cihaz identifikatoru təsadüfi qurulur və **izləmə üçün deyil**: yeganə məqsədi
lisenziyanın icazə verdiyi cihaz sayını saymaqdır. Aparatınızın «barmaq izi»
(ekran ölçüsü, brauzer versiyası və s.) toplanmır.

---

## 5. Sənədlərin çapı və ixracı — ölçülmüş fakt

Hazırlanan PDF, DOCX və XLSX faylları, həmçinin ehtiyat nüsxə faylı **birbaşa
sizin diskinizə** yazılır. Onlar heç bir serverə göndərilmir.

⚠️ **Ehtiyat nüsxə şifrələnmir.** O, bütün müştəri məlumatını açıq mətnlə
daşıyır — faylın saxlanması və ötürülməsi sizin məsuliyyətinizdədir.

---

## 6. Məlumatın silinməsi — ölçülmüş fakt

**Ayarlar → Məlumat → «Hər şeyi sil»** düyməsi bu kompüterdəki bütün kreditləri,
müştəriləri və ayarları silir.

> ⚠️ **Ayrıca bir müştərini silmək mümkün deyil.** Genişlənmədə belə funksiya
> yoxdur — silmə yalnız hamısı birlikdə işləyir. Bunu bilərək planlaşdırın.

Buluddakı qeydin silinməsi 8-ci bölmədədir.

---

## 7. Məlumatı kim saxlayır — ölçülmüş fakt

- **Sizin kompüteriniz** — müştəri məlumatının hamısı.
- **Google Firebase (Firestore)** — 1.1-də sadalanan lisenziya qeydləri.
  Google-un öz şərtləri tətbiq olunur.
- **Genişlənmənin təchizatçısı** — lisenziya bazasına inzibati çıxışı var, yəni
  e-poçtları və cihaz siyahılarını görə bilir. Müştəri məlumatına çıxışı
  **yoxdur**, çünki o məlumat buluda ümumiyyətlə göndərilmir.

---

## 8. Saxlama müddəti — təchizatçının siyasəti

> ⚠️ Bu bölmə proqramın necə işlədiyini deyil, təchizatçının öhdəliyini təsvir
> edir. Kodda avtomatik silinmə mexanizmi yoxdur — silinmə əl ilə aparılır.

- **Buluddakı qeydlər** — e-poçt, cihaz siyahısı, rol və aktivləşdirmə
  vəziyyəti — lisenziya **aktiv olduğu müddətdə** saxlanılır.
- Lisenziya **ləğv olunanda və ya müddəti bitəndə** həmin qeydlər silinir.
- ⚠️ **Cihazınızdakı məlumat buna daxil DEYİL.** Kreditlər, müştəri kartları və
  ayarlar sizin kompüterinizdədir; təchizatçı onlara toxuna bilmir və silə
  bilmir. Onları yalnız siz silirsiniz — 6-cı bölmədəki qaydada.

Buluddakı qeydinizin vaxtından əvvəl silinməsini istəyirsinizsə, 10-cu
bölmədəki ünvana yazın.

---

## 9. Yaş həddi — təchizatçının siyasəti

Genişlənmə peşəkar iş alətidir və lombard və lizinq şirkətlərinin işçiləri
üçün nəzərdə tutulub. 18 yaşdan kiçiklərə **yönəlmir** və onlardan qəsdən
məlumat toplanmır.

---

## 10. Əlaqə

Suallar və məlumatın silinməsi tələbi üçün:

**niyazi@qasimsoy.info**

---

## Sənəd haqqında

| | |
|---|---|
| **Sonuncu yenilənmə** | 11 avqust 2026 |
| **Sənəd versiyası** | 1.0 |

Sənəd dəyişdirilərsə, hər iki sətir yenilənir.

---
---

# Privacy Policy

This document explains what data the "Loan Docs" Chrome extension collects,
where it is stored, and who it is shared with.

> ⚠️ **Short answer:** your loan customers' personal data **never leaves your
> computer**. Only the minimum required for licensing is sent to the cloud: the
> employee's Google email address and a device name.

### Two kinds of statement in this document

Sections fall into two groups, marked in their headings:

- **Measured fact** — describes how the software actually works. Verified
  against the source code; it does not change unless the software changes.
  Sections **1–7**.
- **Supplier policy** — depends on the supplier's commitment rather than on how
  the software works. It cannot be verified from the code. Sections **8–9**.

The distinction is deliberate: the first is verifiable, the second is a promise.

---

## 1. Data collected — measured fact

### 1.1 Sent to the cloud — only two pieces of personal data

| Data | Purpose | Stored in |
|---|---|---|
| Employee's **Google email** | linking the account to a licence | Firestore, `users/{uid}` |
| **Device name** entered by the employee | identifying the machine, enforcing the device limit | Firestore, `users/{uid}.devices` |

Alongside these, **non-personal** technical records are sent:

- company identifier, account status, permitted device count, user role
  (`admin` or `staff`);
- per device: a random identifier, first-seen and last-seen timestamps;
- activation-code status: whether a code was used, by whom, and when;
- **for admin users only:** the company's centralised settings — corporate
  details, interest rates, penalty terms. These are configuration, not customer
  data.

### 1.2 NOT sent to the cloud — all loan customer data

To produce documents, the extension stores extensive customer information:
name, date and place of birth, identity document details, personal
identification number, registered and actual address, marital status, employer,
income, phone numbers, loan terms, and descriptions of collateral or leased
assets.

**None of this is transmitted over the network.** It all stays in the browser's
local database (IndexedDB) on your computer.

---

## 2. What is stored on your computer — measured fact

| Location | Contents |
|---|---|
| **IndexedDB** — `loans` | loan and lease records, plus the frozen copy printed on documents |
| **IndexedDB** — `customers` | customer records |
| **IndexedDB** — `settings` | corporate details, tariffs, service points, staff lists |
| **`chrome.storage.local`** | device identifier and name, licence mirror, activation state |
| **`chrome.storage.session`** | the open tab's id; discarded when the browser closes |

⚠️ **`chrome.storage.sync` is not used.** It would replicate data to Google's
servers across your devices.

---

## 3. Signing in with Google — measured fact

Sign-in is via Google only. **No password is requested or stored** — the
extension has no password field at all.

The Google consent screen requests `openid email profile`. This is Google's
standard set and may be shown as "see your name and profile picture".
⚠️ **Your name and profile picture are not stored** — the extension reads and
keeps only the **email address**.

---

## 4. Network connections — measured fact

The extension contacts only three Google endpoints:

```
identitytoolkit.googleapis.com     — sign-in
securetoken.googleapis.com         — session refresh
firestore.googleapis.com           — licence and centralised settings
```

No other destination is contacted; the browser blocks this technically.

- **No analytics.** No usage statistics, behavioural tracking, or click data.
- **No telemetry.** Crash reports are not sent automatically.
- **No third-party scripts.** No ad networks, tracking pixels, or remote
  libraries; the content security policy forbids them.
- **Fonts are bundled** and never fetched at runtime.

The device identifier is randomly generated and is **not used for tracking**:
its only purpose is counting devices against the licence limit. No hardware
fingerprinting (screen size, browser version, and so on) is performed.

---

## 5. Printing and exporting — measured fact

Generated PDF, DOCX and XLSX files, and the backup file, are written **directly
to your disk**. They are not uploaded anywhere.

⚠️ **The backup file is not encrypted.** It contains all customer data in plain
text; storing and transferring it safely is your responsibility.

---

## 6. Deleting data — measured fact

**Settings → Data → "Delete everything"** removes all loans, customers, and
settings from this computer.

> ⚠️ **Deleting an individual customer is not possible.** The extension has no
> such function — deletion works only for everything at once. Please plan
> accordingly.

Deletion of your cloud record is covered in section 8.

---

## 7. Who stores the data — measured fact

- **Your computer** — all customer data.
- **Google Firebase (Firestore)** — the licensing records listed in 1.1.
  Google's own terms apply.
- **The extension's supplier** — has administrative access to the licence
  database and can therefore see email addresses and device lists. The supplier
  has **no access** to customer data, because that data is never sent to the
  cloud.

---

## 8. Retention period — supplier policy

> ⚠️ This section describes the supplier's commitment, not how the software
> works. There is no automatic deletion mechanism in the code; deletion is
> performed manually.

- **Cloud records** — email address, device list, role, and activation status —
  are kept **for as long as the licence is active**.
- When the licence is **revoked or expires**, those records are deleted.
- ⚠️ **Data on your device is NOT included.** Loans, customer records, and
  settings live on your computer; the supplier cannot reach them and cannot
  delete them. Only you can, using the method in section 6.

To have your cloud record deleted sooner, write to the address in section 10.

---

## 9. Age limit — supplier policy

The extension is a professional tool intended for employees of pawnshops and
leasing companies. It is **not directed at** anyone under 18, and no data is
knowingly collected from them.

---

## 10. Contact

For questions or data deletion requests:

**niyazi@qasimsoy.info**

---

## About this document

| | |
|---|---|
| **Last updated** | 11 August 2026 |
| **Document version** | 1.0 |

If the document changes, both rows are updated.
