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
return await _context.ConsentMaster.Where(a => a.ConsentCode == consentCode)
            .Where(a => a.ConsentStatus != (int)ConsentMasterEnums.Status.Destroyed)
            .OrderByDescending(a => a.LastUpdateDate).FirstOrDefaultAsync(cancellationToken);

await _context.loadStoredProcedureBuilder("sp_SFID_getCustomerList_v2")
                .AddParam("SalesmanCode", request.SalesmanCode)
                .AddParam("Search", string.IsNullOrEmpty(request.Search) ? "" : request.Search)
                .AddParam("Page", request.Page)
                .AddParam("Limit", request.Limit)
                .AddParam("SortColumn", request.SortColumn)
                .AddParam("SortType", request.SortType.ToUpper())
                .AddParam("IsSalesman", request.IsSalesman)
                .AddParam("paramDealerId", request.ParamDealerId)
                .ExecAsync(async r => retrieveCustomer = await r.ToListAsync<CustomerPage>(cancellationToken));
```

---

## 3. NoLock dengan Entity Framework
Implementasi **READUNCOMMITTED** untuk membaca data tanpa menunggu transaksi selesai (mirip `WITH (NOLOCK)` di SQL).

**Contoh**:
```csharp
public static async Task<List<T>> ToListWithNoLockAsync<T>(this IQueryable<T> query, CancellationToken cancellationToken = default)
{
    List<T> result = default;
    using (var scope = new TransactionScope(TransactionScopeOption.Suppress,
                            new TransactionOptions()
                            {
                                IsolationLevel = IsolationLevel.ReadUncommitted
                            },
                            TransactionScopeAsyncFlowOption.Enabled))
    {
        result = await query.ToListAsync(cancellationToken);
        scope.Complete();
    }
    return result;
}

public static async Task<T> FirstOrDefaultWithNoLockAsync<T>(this IQueryable<T> query, CancellationToken cancellationToken = default)
{
    T result = default;
    using (var scope = new TransactionScope(TransactionScopeOption.Suppress, // Suppress if sql option EnableRetryOnFailure is active, else required
                            new TransactionOptions()
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

public static async Task<bool> AnyWithNoLockAsync<T>(this IQueryable<T> query, CancellationToken cancellationToken = default)
{
    bool result = false;
    using (var scope = new TransactionScope(TransactionScopeOption.Suppress, // Suppress if sql option EnableRetryOnFailure is active, else required
                            new TransactionOptions()
                            {
                                IsolationLevel = IsolationLevel.ReadUncommitted
                            },
                            TransactionScopeAsyncFlowOption.Enabled))
    {
        result = await query.AnyAsync(cancellationToken);
        scope.Complete();
    }
    return result;
}

public static async Task<int> CountWithNoLockAsync<T>(this IQueryable<T> query, CancellationToken cancellationToken = default)
{
    int result = 0;
    using (var scope = new TransactionScope(TransactionScopeOption.Suppress, // Suppress if sql option EnableRetryOnFailure is active, else required
                            new TransactionOptions()
                            {
                                IsolationLevel = IsolationLevel.ReadUncommitted
                            },
                            TransactionScopeAsyncFlowOption.Enabled))
    {
        result = await query.CountAsync(cancellationToken);
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

### Kapan Harus Naik Versi API?

Versi baru **_disarankan dibuat_** apabila terjadi perubahan yang **_tidak kompatibel (breaking changes)_**, seperti:

- Perubahan struktur response (misalnya mengubah field, menghapus field lama, mengganti format data)
- Perubahan struktur request body yang tidak bisa lagi digunakan oleh aplikasi lama
- Perubahan business logic atau alur proses yang menghasilkan data/aksi berbeda dengan input yang sama
- Perubahan HTTP method (contoh: dari `POST` ke `PUT`)
- Perubahan status code yang memengaruhi logika aplikasi client

---
