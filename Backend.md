# Dokumentasi Teknis Singkat

## 1. `AsNoTracking()`
Digunakan di Entity Framework untuk membaca data **tanpa tracking** oleh context.

- Cocok untuk operasi **read-only**
- Meningkatkan performa dan mengurangi penggunaan memori

**Contoh**:
```csharp
var data = dbContext.Users.AsNoTracking().ToList();
```

---

## 2. `CancellationToken`
Digunakan untuk **membatalkan operasi asynchronous** jika tidak diperlukan lagi.

- Umumnya digunakan saat pemanggilan async method
- Dapat menghindari eksekusi tidak perlu dan menghemat resource

**Contoh**:
```csharp
await Task.Delay(5000, cancellationToken); // Dapat dibatalkan
```

---

## 3. NoLock dengan Entity Framework
Implementasi **READUNCOMMITTED** untuk membaca data tanpa menunggu transaksi selesai (mirip `WITH (NOLOCK)` di SQL).

- Memungkinkan membaca data yang sedang diubah
- Dapat meningkatkan performa, tapi **berisiko data tidak akurat**
- Cocok untuk laporan atau kebutuhan non-kritis

**Contoh**:
```csharp
public static async Task<T> FirstOrDefaultWithNoLockAsync<T>(
    this IQueryable<T> query,
    CancellationToken cancellationToken = default)
{
    T result = default;

    using (var scope = new TransactionScope(
        TransactionScopeOption.Suppress,
        new TransactionOptions
        {
            IsolationLevel = IsolationLevel.ReadUncommitted
        },
        TransactionScopeAsyncFlowOption.Enabled))
    {
        result = await query.FirstOrDefaultAsync(cancellationToken);
        scope.Complete();
    }

    return result;
}
```

---

## 4. Versioning API
Strategi untuk **menentukan versi API** agar perubahan tidak mengganggu aplikasi yang menggunakan versi lama.

- Contoh versi API:
  ```
  /work-order/v2/time-register
  ```

**Keuntungan**:
- Versi lama tetap bisa digunakan
- Versi baru bisa dikembangkan dengan aman

---
