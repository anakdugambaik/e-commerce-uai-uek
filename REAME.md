# Tech Stack
Project ini menggunakan Alpinejs, Tailwind, php, mysql, datatables (dan jquery yang dibutuhkannya), sweetalert2, chartjs.
Grid atau tabel interaktif (editable / inline input) harus dibuat manual menggunakan HTML + Alpine.js.
Datatables hanya untuk tampilan read-only, list besar, dan pagination otomatis.

# Struktur Folder
index.php -> semua halaman diakses dari ini dengan router. misal, index.php?r=master_data -> mengarah ke /pages/master_data.php
/pages -> halaman-halaman php terletak di folder ini 
/api -> file php untuk api masing-masing halaman.

### Catatan :
Karena index.php sudah memuat tag head, body, dan link stylesheet dan cdn, 
ketika membuat halaman tidak perlu link ulang selain apa yang dibutuhkan halaman yang sedang dibuat.
Php import/include UNTUK halaman harus dengan asumsi filenya terletak di root public, karena diakses via index.php
Php import/include UNTUK file api harus dengan asumsi filenya terletak di api/

# Struktur "1 halaman"
1 halaman berdiri sendiri, terdiri dari dua file :
1. nama_halaman.php → Berisi main page layout dengan tailwind, data bindings, dan script tag untuk alpinejs di bawa. Minimalkan css inline kecuali dibutuhkan, gunakan tailwind. Jangan ubah tipe font.
2. nama_halaman_api.php → backend logic, untuk provide atau store data (return JSON). Baik untuk get data untuk select list, api endpoint untuk handle data input dan request lainnya.

## Perilaku Frontend (Alpine.js)
Gunakan Alpine.store() untuk data reaktif global atau bersama (serupa dengan layanan Angular).
1. User membuka halaman.php
2. onInit() memanggil fetch('halaman_api.php?action=get_data')
3. Data JSON diisi ke variabel reaktif (Alpine)
4. UI otomatis update
5. onSubmit() dipanggil saat form disubmit → fetch('halaman_api.php?action=save')
6. Response dikembalikan → tampilkan notifikasi SweetAlert2

## Interaksi Server
Gunakan fetch() di dalam fungsi Alpine untuk memanggil endpoint PHP.
Asumsikan skrip PHP mengembalikan data JSON. Contoh fetch:

```
const res = await fetch('nama_halaman_api.php?action=get_header', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload)
});
if (!res.ok) {
  console.error('Error:', res.status);
}
const data = await res.json();
```

## Membuat Forms
1. Formulir menggunakan x-model untuk setiap field dan @submit.prevent="onSubmit".
2. Data formulir dapat dikirim langsung ke PHP melalui fetch.
3. Validasi harus dilakukan secara reaktif (tampilkan pesan di sebelah field).
4. Pengisian data sebelumnya bekerja dengan mengatur nilai model secara langsung (contoh: form.name = 'John').

## Angular-Like FormBuilder Pattern
Untuk memudahkan pembuatan form, sudah dibuat dua script, di assets/js, yaitu formBuilder.js dan validators.js, untuk menyerupai fitur Reactive Form angular dengan Alpinejs.
Jika field invalid, tambahkan class border-red-500 atau tampilkan pesan kecil dengan Tailwind text-red-500 text-sm.

1. Fields: { value, validators, error }
2. Functions: isValid(), getValues(), patchValue()
3. Validator functions yang ada : required, minLength, maxLength, pattern, email, numeric, positiveNumber, min, max, date, matchField
Validation and reactive updates should behave like Angular’s reactive forms.
```
onInit() {
  // build form like Angular FormBuilder
  this.form = this.$formBuilder({
    name: { value: '', validators: [Validators.required(), Validators.minLength(3)] },
    date: { value: '', validators: [Validators.required()] }
  });

  // Prefill example, real example ada handler CRUD untuk new data atau existing data
  this.form.patchValue({ name: 'John Doe' });
},

async onSubmit() {
  if (!this.form.isValid()) return;

  const data = this.form.getValues();

  const res = await fetch('https://webhook-test.com/cdb03e0a0a0c1573d9cc2e5bbd4b0e2a', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });

  if (res.ok) {
    this.message = '✅ Form submitted successfully!';
  } else {
    this.message = '❌ Failed to submit form';
  }
}
```

## Alerts
Gunakan contoh snippet ini untuk semua alert, baik sukses, gagal, dll. Sudah ada styling termasuk untuk custom alert, tidak perlu dibuat lagi.
```
Swal.fire({
  toast: true,
  position: 'top-right',
  icon: 'success', //success, error, warning, info, question
  title: 'Saved!',
  timer: 2500,
  timerProgressBar: true,
  customClass: {
    popup: 'colored-toast',
  },
  showConfirmButton: false
});
```

## Library Require PHP
Untuk semua halaman php, tambahkan require berikut 
1. /config/database.php -> berisi definisi berisi definisi $pdo
2. /lib/helpers.php -> Berisi helper h escape, require_login([]); yang harus dipanggil semua halaman, require_login_api([]); yang harus dipanggil semua file api.

### Catatan mengenai authentication
Semua request ke file *_api.php berjalan di domain yang sama, sehingga session PHP ($_SESSION) akan otomatis terbawa selama user login. Tidak perlu token khusus — autentikasi berbasis require_login_api() yang memeriksa session aktif.

## Format standar return JSON
// Format standar response JSON
```
{
  "success": true,           // boolean
  "message": "OK",           // optional string
  "data": {...}              // optional payload
}
```

# Format Output Halaman
Ketika saya mengatakan "buat halaman," generate kode dalam tiga bagian:

1. Template HTML (dengan binding Alpine, dan styling Tailwind)
2. Logika JavaScript (menggunakan Alpine.data dan/atau Alpine.store)
3. Endpoint PHP (API yang mengembalikan JSON)


# Coding Convention Summary
| Aspek                 | Standar                                            |
| --------------------- | -------------------------------------------------- |
| Struktur file halaman | `nama_halaman.php` + `nama_halaman_api.php`        |
| Format response       | `{ success, message, data }`                       |
| API param utama       | `action`                                           |
| Auth check            | `require_login()` / `require_login_api()`          |
| Frontend reactive     | Alpine.store(), x-data(), x-model()                |
| Library               | Tailwind, Alpine, Datatables, SweetAlert2, ChartJS |
| Data fetch            | `fetch('nama_api.php?action=...')`                 |
| Validasi              | Menggunakan `validators.js`, visual error merah    |
| Error display         | SweetAlert2                                        |
| Grid editable         | HTML + Alpine.js (bukan Datatables)                |