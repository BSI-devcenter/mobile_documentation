# Development Process

## 1. Branching Strategy

- `Main`: Branch utama untuk kode yang sudah stabil dan siap rilis.
- `Staging`: Branch mirroring `main` untuk pull down atau menghubungkan main dengan operation.
- `Operation`: Branch untuk pengembangan fitur baru.
- `Operation-Release`: Branch untuk mengumpulkan kode yang sudah siap rilis untuk kebutuhan operation.
- `Development-Release`: Branch untuk mengumpulkan kode yang sudah siap rilis untuk kebutuhan CR Development.
- `Development`: Branch untuk melakukan test oleh tester setelah fitur dikembangkan.
- `CR/*`: Branch untuk fitur baru.  
  Contoh: `CR/leasing-submission`
- `Patch/*`: Branch untuk perbaikan bug.  
  Contoh: `Patch/Leasing-Submission/23134`

---

## 2. Commit Messages

Gunakan pesan commit yang jelas dan deskriptif. Format yang disarankan:

```
[Tipe] Deskripsi singkat
```

- `Tipe`: `feature`, `bugfix`, `refactor`, `docs`, `test`, `chore`

### Contoh:
```
feature: Add user authentication endpoint  
bugfix: Fix login error when password is incorrect
```

---

## 3. Pull Request (PR)

- **1 Ticket = 1 Branch**  
  Setelah Pull Request (PR) selesai, branch terkait harus dihapus.

- **New Ticket = New Branch**  
  Setiap ticket baru harus menggunakan branch baru berdasarkan target branch yang ditentukan (misal: `CR-1`).

- **PR Handling**  
  Jika pekerjaan belum selesai atau masih ragu, set PR sebagai `DRAFT`.  
  Jika sudah final, publish PR dan segera informasikan ke Reviewer.

- **PR Description**  
  Jelaskan dengan jelas daftar perubahan kode.  
  Sertakan link ticket atau work item.

- **SonarLint Evidence**  
  Lampirkan screenshot hasil SonarLint sebagai bukti kualitas kode.

- **Code Review Ticket**  
  Buat ticket khusus untuk keperluan code review.

---

## 4. Response Standard

- **Request**: Gunakan format `snake_case`
- **Response**: Gunakan format `snake_case`

### Contoh:
```json
{
  "id": 123,
  "user_name": "JohnDoe",
  "user_email": "Email"
}
```
## 5. Error Handling
- Gunakan kode status HTTP yang sesuai untuk setiap respons.
- Sertakan pesan error yang jelas dan informatif.

## 6. Logging
- Gunakan logging untuk mencatat informasi penting, error, dan debug.
- Pastikan log tidak mengandung informasi sensitif.