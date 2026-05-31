# Booking Page — Admin Guide & Strategy
## RW Beauty Salon & Spa by Isma

---

## 1. STRATEGY SUMMARY

### Masalah yang Diselesaikan
| Sebelum (Google Form) | Sesudah (Booking Page) |
|---|---|
| Semua treatment tampil sekaligus (overwhelming) | Difilter berdasarkan kebutuhan pelanggan |
| Tidak ada panduan memilih treatment | Ada "Need Selector" — pelanggan pilih kebutuhan dulu |
| Terasa seperti formulir administrasi | Terasa seperti guided booking assistant |
| Tidak ada estimasi harga | Ada estimasi total sebelum submit |
| Tidak ada ringkasan sebelum kirim | Ada summary step untuk review ulang |
| Pesan WA harus diketik manual oleh admin | Pesan WA di-generate otomatis, siap kirim |

### Perjalanan Pelanggan (6 Langkah)
```
Step 1: Welcome → Mulai Reservasi
Step 2: Pilih kebutuhan (12 opsi)
Step 3: Pilih treatment (difilter + searchable + cart)
Step 4: Isi jadwal & data diri
Step 5: Review ringkasan
Step 6: Kirim ke WA admin
```

### Prinsip UX Utama
1. **Jangan tampilkan semua treatment sekaligus** — filter berdasarkan need
2. **Satu CTA dominan** — semua berujung ke WhatsApp
3. **Pelanggan bisa pilih banyak treatment** sekaligus, dengan estimasi total
4. **Salon vs Home Service** — branching yang menentukan field alamat
5. **Mobile-first** — touchable cards, font besar, scroll pendek

---

## 2. CARA AKTIFKAN GOOGLE SHEETS INTEGRATION

### Langkah 1 — Buat Google Sheet
Buat Google Sheet baru dengan kolom berikut (baris 1 sebagai header):

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Timestamp | Nama | WhatsApp | Gender | Kategori Kebutuhan | Treatment Dipilih | Estimasi Total | Tanggal | Jam | Jumlah Orang | Tipe Kunjungan | Kecamatan | Kota | Kelurahan | Alamat | Catatan | Lead Source | Status | Admin Notes |

### Langkah 2 — Buat Google Apps Script
1. Di Google Sheet, buka **Extensions → Apps Script**
2. Hapus kode yang ada, paste kode berikut:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = JSON.parse(e.postData.contents);
  
  sheet.appendRow([
    data.timestamp,
    data.nama,
    data.whatsapp,
    data.gender,
    data.needCategory,
    data.treatments,
    data.estimatedTotal,
    data.tanggal,
    data.jam,
    data.jumlah,
    data.visitType,
    data.kecamatan,
    data.kota,
    data.kelurahan || '',
    data.alamat || '',
    data.catatan || '',
    data.leadSource,
    data.status || 'New Lead',
    '' // Admin Notes (diisi manual)
  ]);
  
  return ContentService
    .createTextOutput(JSON.stringify({status:'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet(e) {
  return ContentService.createTextOutput('RW Beauty Booking API - OK');
}
```

3. Klik **Deploy → New Deployment**
4. Pilih Type: **Web App**
5. Execute as: **Me**
6. Who has access: **Anyone**
7. Klik **Deploy** → **Authorize**
8. Copy **Web App URL** yang muncul

### Langkah 3 — Pasang URL di booking.html
Buka `booking.html`, cari baris:
```javascript
const CONFIG = {
  waNumber: '628xxxxxxxxxx',
  googleScriptURL: '', /* Paste Google Apps Script URL here */
```
Ganti string kosong dengan URL yang dicopy:
```javascript
  googleScriptURL: 'https://script.google.com/macros/s/ABC.../exec',
```

### Langkah 4 — Ganti nomor WhatsApp
Di baris yang sama, ganti `628xxxxxxxxxx` dengan nomor WA aktif admin:
```javascript
  waNumber: '6281234567890',
```

---

## 3. ADMIN WORKFLOW

### Status Lead
Setiap lead baru masuk dengan status **"New Lead"**. Admin update status secara manual di Google Sheet:

| Status | Keterangan |
|---|---|
| **New Lead** | Form baru masuk, belum dihubungi |
| **Contacted** | Admin sudah kirim WA ke pelanggan |
| **Waiting Customer Reply** | Menunggu balasan pelanggan |
| **Booked** | Jadwal sudah dikonfirmasi kedua pihak |
| **Completed** | Treatment selesai dilakukan |
| **Cancelled** | Dibatalkan (catat alasan di Admin Notes) |
| **No Show** | Pelanggan tidak datang tanpa kabar |

### Alur Kerja Admin Harian
```
Pagi: Buka Google Sheet → filter status "New Lead"
  ↓
Hubungi via WA dalam 1-2 jam (gunakan template di bawah)
  ↓
Update status ke "Contacted"
  ↓
Setelah jadwal dikonfirmasi → update ke "Booked"
  ↓
1 hari sebelum → kirim reminder (Template 5)
  ↓
2 jam sebelum → kirim reminder (Template 6)
  ↓
Setelah selesai → update "Completed", kirim terima kasih (Template 7)
  ↓
2–3 hari setelah → kirim permintaan review (Template 8)
```

---

## 4. WHATSAPP MESSAGE TEMPLATES

### Template 1 — Respons Lead Baru
```
Halo [NAMA] 🌸

Terima kasih sudah mengisi formulir reservasi Rumah Wangi!

Kami sudah menerima permintaan kamu untuk:
✨ [TREATMENT DIPILIH]
📅 [TANGGAL] pukul [JAM]
👥 [JUMLAH ORANG]

Kami sedang cek ketersediaan slot dan akan konfirmasi dalam waktu dekat.

Ada yang bisa kami bantu atau ada pertanyaan sebelum kami konfirmasi?

Salam hangat,
Admin Rumah Wangi 🌸
```

---

### Template 2 — Konfirmasi Jadwal
```
Halo [NAMA] 🌸

Kabar baik! Jadwal kamu sudah kami konfirmasi:

✅ Treatment: [TREATMENT]
📅 Tanggal: [TANGGAL]
🕐 Jam: [JAM] WIB
👥 Jumlah: [JUMLAH ORANG]
📍 Lokasi: [SALON / HOME SERVICE]

Terapis kami akan siap menyambut kamu.

Mohon hadir 5–10 menit sebelum waktu yang dijadwalkan ya 😊

Sampai jumpa,
Admin Rumah Wangi 🌸
```

---

### Template 3 — Konfirmasi Harga
```
Halo [NAMA] 🌸

Berikut detail harga untuk treatment yang kamu pilih:

[TREATMENT 1] → Rp [HARGA]
[TREATMENT 2] → Rp [HARGA]

Total: Rp [TOTAL]

*Harga belum termasuk biaya tambahan (jika ada)

Apakah kamu setuju dengan harga di atas? Kami bisa lanjutkan konfirmasi jadwalnya 😊

Salam,
Admin Rumah Wangi 🌸
```

---

### Template 4 — Rekomendasi Treatment
```
Halo [NAMA] 🌸

Terima kasih sudah menghubungi Rumah Wangi!

Berdasarkan kebutuhan kamu ([KEBUTUHAN]), kami merekomendasikan:

💆 [TREATMENT 1] – Rp [HARGA]
   [Deskripsi singkat manfaat]

✨ [TREATMENT 2] – Rp [HARGA]
   [Deskripsi singkat manfaat]

Kamu bisa pilih salah satu atau keduanya sekaligus 😊

Mau kami bantu booking jadwalnya?

Salam,
Admin Rumah Wangi 🌸
```

---

### Template 5 — Reminder H-1
```
Halo [NAMA] 🌸

Reminder jadwal treatment kamu besok:

✨ [TREATMENT]
📅 [TANGGAL], pukul [JAM] WIB
📍 [LOKASI]

Tips sebelum treatment:
• Hadir 5–10 menit lebih awal
• Kenakan pakaian yang nyaman
• Informasikan jika ada kondisi kesehatan khusus

Sampai jumpa besok! 🌸

Admin Rumah Wangi
```

---

### Template 6 — Reminder 2 Jam Sebelum
```
Halo [NAMA] 🌸

Kamu ada jadwal treatment dalam 2 jam lagi:

🕐 [JAM] WIB — [TREATMENT]

Kami sudah siapkan segalanya untukmu. Tinggal berangkat! 😊

Jika ada perubahan, hubungi kami segera ya.

Sampai jumpa,
Admin Rumah Wangi 🌸
```

---

### Template 7 — Terima Kasih Setelah Treatment
```
Halo [NAMA] 🌸

Terima kasih sudah mempercayakan perawatanmu kepada Rumah Wangi hari ini!

Semoga treatment [TREATMENT] yang tadi terasa bermanfaat dan membuat kamu lebih segar dan nyaman 💆

Jika ada pertanyaan atau masukan, jangan ragu hubungi kami ya.

Nantikan promo dan treatment terbaru kami di:
📸 Instagram: @rumahwangi_cm

Sampai jumpa di kunjungan berikutnya! 🌸

Admin Rumah Wangi
```

---

### Template 8 — Permintaan Review (H+2 atau H+3)
```
Halo [NAMA] 🌸

Sudah 2–3 hari sejak treatment [TREATMENT] di Rumah Wangi.

Bagaimana hasilnya? Semoga kamu sudah merasakan manfaatnya ya 😊

Kami sangat berterima kasih jika kamu bersedia memberikan ulasan di Google Review kami:
👉 [LINK GOOGLE REVIEW]

Review kamu sangat membantu kami untuk terus berkembang dan melayani lebih baik 🙏

Sebagai tanda terima kasih, kamu bisa menyebut nama kamu saat booking berikutnya untuk mendapatkan *surprise* dari kami 🎁

Terima kasih banyak!
Admin Rumah Wangi 🌸
```

---

## 5. PLACEHOLDER CHECKLIST

Sebelum booking page diluncurkan, pastikan semua item ini sudah diisi:

### booking.html
- [ ] `628xxxxxxxxxx` → Ganti dengan nomor WA admin aktif (muncul di ~8 tempat)
- [ ] `CONFIG.googleScriptURL` → Paste URL Google Apps Script
- [ ] Footer: alamat lengkap salon
- [ ] Footer: jam operasional
- [ ] Google Maps URL di footer

### booking-admin.md (dokumen ini)
- [ ] Link Google Review → Ganti `[LINK GOOGLE REVIEW]` di Template 8
- [ ] Detail alamat salon untuk Template 2

### Sebelum Go-Live
- [ ] Test submit form → cek data masuk ke Google Sheet
- [ ] Test klik "Kirim ke WhatsApp Admin" → cek pesan pre-filled benar
- [ ] Test di iPhone (Safari) dan Android (Chrome)
- [ ] Test semua FAQ accordion berfungsi
- [ ] Test Treatment Finder — semua 12 kategori kebutuhan muncul
- [ ] Test search treatment
- [ ] Test cart — tambah/hapus item
- [ ] Test branching Home Service → field alamat muncul
- [ ] Pastikan progress bar tampil dengan benar di semua step
- [ ] Floating WA button berfungsi di semua halaman

---

## 6. TIPS OPTIMASI KONVERSI

1. **Pasang Google Analytics** — tambahkan GA4 tracking di `<head>` untuk pantau drop-off per step
2. **UTM parameter** — gunakan link dengan UTM saat share ke IG Story, WA Broadcast, dsb:
   ```
   booking.html?utm_source=instagram&utm_medium=story&utm_campaign=juni2025
   ```
3. **Pin di WA Business** — jadikan link booking sebagai pesan yang di-pin di profil WA Business
4. **Tambah di bio Instagram** — link booking di bio @rumahwangi_cm
5. **WA Auto-Reply** — setel auto-reply WA Business dengan link booking untuk pelanggan yang WA duluan
6. **Google Business** — tambahkan tombol "Book Online" di Google My Business yang mengarah ke booking page

---

*Dokumen ini dibuat sebagai panduan teknis dan operasional untuk tim Rumah Wangi Beauty Salon & Spa.*
*Update terakhir: 2025*
