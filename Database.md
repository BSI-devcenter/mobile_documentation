# 📌 Standard dan Best Practice SQL Development

## 1. Tipe Data
- Gunakan tipe data seperlunya untuk efisiensi dan konsistensi.
- Contoh tipe data: `nchar`, `tinyint`, `bit`, `money`, dll.

## 2. Index
- Untuk tabel dengan estimasi pertumbuhan data yang cepat, **tambahkan index** pada kolom yang sering digunakan untuk filter.

## 3. Foreign Key
- Tambahkan **foreign key (FK)** untuk kolom yang berelasi.
- Hindari menyimpan kolom yang sudah ada di tabel FK, misalnya:
  > Jika sudah ada `DealerID`, **tidak perlu** menyimpan `DealerCode`.

## 4. Allow Nulls
- Sesuaikan dengan kebutuhan bisnis.
  > Misal: Kolom `Kode` tidak boleh null → **uncheck allow nulls**

## 5. Max Length
- Sesuaikan dengan estimasi kebutuhan.
  > Contoh: Kolom "Nama" cukup 50 karakter, **jangan dibuat terlalu besar**.

## 6. Default Value
- Untuk kolom tanpa FK, tambahkan default value (jika tidak konflik dengan bisnis):
  - Empty string `''`
  - `0`
  - `GETDATE()` atau `MIN(DateTime)`

## 7. Penamaan Kolom
- Gunakan nama kolom yang **informatif**.
  > Contoh: Jangan pakai `ProductSegment3`, tapi `ProductType`.

## 8. Default Column (Standard)
```sql
ID               SMALLINT/INT (PK)
RowStatus        SMALLINT
CreatedBy        VARCHAR(30)
CreatedTime      DATETIME
LastUpdatedBy    VARCHAR(30)
LastUpdatedTime  DATETIME
```

## 9. Alter Stored Procedure dengan Parameter Baru
- Tambahkan parameter baru sesuai kebutuhan.
- Sertakan metadata di awal SP (lihat Audit Object di bawah).

## 10. NOLOCK
- Gunakan `WITH (NOLOCK)` saat perlu baca data besar atau read-only query.

## 11. SELECT
- Hanya ambil data yang dibutuhkan:
  > Hindari `SELECT *`
- Gunakan `TOP` jika hanya ingin ambil data tertentu:
  ```sql
  SELECT TOP 10 * FROM TableName
  ```

## 12. UPDATE Data
- **Selalu update**:
  - `LastUpdatedBy`
  - `LastUpdatedTime`

## 13. Struktur Branch Location
- File SQL disimpan di repo:
  ```
  D-Net Service / sql / Database
  ```
- Semua perubahan dilakukan melalui **PR ke folder ini**, **bukan ke branch mobile/BE**.

## 14. Standard Code
- Sebelum membuat category baru, cek apakah **category tersebut sudah tersedia**.

## 15. Penghapusan Data Besar
Gunakan batch deletion:
```sql
DECLARE @Deleted_Rows INT;
SET @Deleted_Rows = 1;
WHILE (@Deleted_Rows > 0)
BEGIN
  DELETE TOP (1000) b 
  FROM TransactionLog a WITH (NOLOCK)
  JOIN TransactionRuntime b WITH (NOLOCK) ON a.Id = b.TransactionLogId
  WHERE a.CreatedTime < @Datemin;

  SET @Deleted_Rows = @@ROWCOUNT;
END
```

## 16. Database Version (CR)
- Gunakan database baru dari restore **production**.
- Jangan gunakan DB:
  - Lebih dari 3 bulan dari current date
  - Dari CR lain yang sudah go-live
- Hapus DB setelah release dan monitoring selesai.

## 17. Masking Data
Gunakan stored procedure masking:
- **MMKSI**:
  ```sql
  EXEC [dbo].[spDataMaskingMMKSI]
  ```
- **KTB**:
  ```sql
  EXEC [dbo].[spDataMaskingKTB]
  ```

## 18. Phone Number
- Maksimal 16 karakter atau sesuai kebutuhan bisnis.
- Validasi UI:
  - Harus diawali `62` atau `0`
  - Hanya angka, **tidak boleh** `+`, `-`, spasi, atau simbol lain.

## 19. SQL Jobs
- Hindari jadwal bentrok dengan job lain (jam genap).
- Gunakan jam unik seperti: `00:15`, `00:40`, dst.
- Job berat/daily: **setelah jam 00:30**

## 20. Execute SQL Script/Query
- Jangan tinggalkan query > 60 detik.
  - **Force cancel** dan perbaiki jika perlu.
- Jika perubahan query:
  - Lakukan **cross-check row count before/after**.
  - Pastikan gap row sesuai ekspektasi.

## 21. Metode Migrasi
- Lakukan di QA, buat table baru jika perlu.
- Persiapan sebelum release untuk efisiensi waktu.
- Gunakan **linked server** untuk migrasi dari QA ke PROD.

### Jika row > 10K
- Buat backup di DB QA:
  ```sql
  ExistingTableName_yyyymmddhhmm_Backup
  ```
  > Contoh: `ChassisMaster_202504072030_Backup`

### Jika row ≤ 10K
- PO backup manual ke Excel

### Selama Release
- Release PIC eksekusi script
- PO validasi dan konfirmasi hasil migrasi

## 22. Audit Object
Untuk semua object non-standar (function, view, SP, dll):

```sql
-- =============================================
-- Author:            <IDBSI (Nama)>
-- Create date:       <yyyy-mm-dd>
-- Last Modified By:  <IDBSI (Nama)>
-- Last Modified date:<yyyy-mm-dd>
-- Description:       <Penjelasan singkat>
-- =============================================
```

> Gunakan kode IDBSI dan nama lengkap, contoh:
```sql
-- Author: BSI80050 (Slamet)
```
