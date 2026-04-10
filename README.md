# 💢 Trigger and View - Warung Madura


### Anggota Kelompok:
- [Mochamad Naufal Hanif](www.linkedin.com/in/mochamad-naufal-hanif-652ab23a5)
- [Keiyla Adya Azzani](www.linkedin.com/in/keiyla-adya-azzani-a75ab53a5)
- [Rinzani Fauziah](www.linkedin.com/in/rinzani-fauziah-9919383a5)

**Kelas: XI RPL 1**

----------

# 📌 Deskripsi

Project ini bertujuan untuk memahami penggunaan:

-   *Trigger* (*BEFORE* & *AFTER*)
    
-   *View*
    
-   Studi kasus yang kami gunakan: sistem pembelian dan pencatatan log aktivitasnya
    

----------

# 🧱 Struktur Tabel

## 📖 Penjelasan

-   Tabel `pembelian` digunakan untuk menyimpan data transaksi
    
-   Tabel `log_pembelian` digunakan untuk mencatat aktivitas (log)
    

```sql
create table pembelian(
    id int auto_increment primary key,
    nama_barang varchar(50),
    jumlah_beli int,
    total_harga int
);

create table log_pembelian(
    id int auto_increment primary key,
    aktivitas varchar(50),
    waktu timestamp default current_timestamp
);

```

----------

# 🔧 Trigger

## 📖 Penjelasan

Trigger digunakan untuk menjalankan aksi otomatis saat terjadi:

-   *INSERT*
-   *UPDATE*
    
-   *DELETE*
    

## ⚙️ Delimiter

### 📖 Penjelasan

`delimiter` digunakan untuk mengganti tanda akhir query sementara.  
Biasanya MySQL menggunakan `;`, tetapi pada trigger digunakan `//` agar perintah di dalam `BEGIN...END` tidak dianggap selesai.

```sql
delimiter //

```

----------

## 💭 INSERT

### 📖 Penjelasan

-   *BEFORE INSERT* → menambahkan harga sebesar 1500 <- anggap saja seperti biaya adminnya
    
-   *AFTER INSERT* → mencatat aktivitas ke log
    

```sql
create trigger before_insert_pembelian
before insert on pembelian
for each row
begin
    set new.total_harga = new.total_harga + 1500;
end;
//

create trigger after_insert_pembelian
after insert on pembelian
for each row
begin
    insert into log_pembelian (aktivitas)
    values ('menambah data pembelian.');
end;
//

```

----------

## 📥 PERCOBAAN INSERT 
```sql
insert into pembelian (nama_barang, jumlah_beli, total_harga) values ('Indomie', 5, 15000);
```
---
```sql
select * from pembelian;
```

### 📊 Hasil `pembelian`

| id | nama_barang | jumlah_beli | total_harga |
|-|-------------|------------|-------------|
|  1 | Indomie     |           5 |       16500 |

---

```sql
select * from log_pembelian;
```

### 📊 Hasil `log_pembelian`

| id | aktivitas                    | waktu               |
|---|--------------------------|---------------------|
|  1 | Menambah data pembelian.         | 2026-04-08 15:14:20 |


----------

## ✏️ UPDATE

### 📖 Penjelasan

-   *BEFORE UPDATE* → mengubah nama menjadi huruf besar <- anggap saia seperti penanda data yang sudah berubah atau diupdate
    
-   *AFTER UPDATE* → mencatat perubahan ke log
    

```sql
create trigger before_update_pembelian
before update on pembelian
for each row
begin
    set new.nama_barang = upper(new.nama_barang);
end;
//

create trigger after_update_pembelian
after update on pembelian
for each row
begin
    insert into log_pembelian (aktivitas) values ('mengubah data pembelian.');
end;
//

```

----------

## ✏️ PERCOBAAN UPDATE

```sql
update pembelian
set total_harga = 19500
where id = 1;
```
---
```sql
select * from pembelian;
```

### 📊 Hasil `pembelian`

| id | nama_barang | jumlah_beli | total_harga |
|----|-------------|-------------|-------------|
|  1 | INDOMIE     |           5 |       19500 |

---

```sql
select * from log_pembelian;
```

### 📊 Hasil `log_pembelian`


| id | aktivitas                    | waktu               |
|----|------------------------------|---------------------|
|  1 | Menambah data pembelian.         | 2026-04-08 15:14:20 |
|  2 | Mengubah data pembelian.         | 2026-04-08 15:16:11 |
----------

## 🗑️ DELETE

### 📖 Penjelasan

-   *BEFORE DELETE* → mencatat sebelum data dihapus
    
-   *AFTER DELETE* → mencatat setelah data dihapus
    

```sql
create trigger before_delete_pembelian
before delete on pembelian
for each row
begin
    insert into log_pembelian (aktivitas) values ('Akan menghapus data pembelian.');
end;
//

create trigger after_delete_pembelian
after delete on pembelian
for each row
begin
    insert into log_pembelian (aktivitas) values ('Menghapus data pembelian.');
end;
//
```

----------

## 🔚 Mengembalikan Delimiter

### 📖 Penjelasan

Setelah selesai membuat trigger, delimiter dikembalikan ke default (`;`) agar query biasa dapat dijalankan kembali tanpa harus mengangkhirinya dengan (`//`).

```sql
delimiter ;
```

----------

## 🗑️ PERCOBAAN DELETE

```sql
delete from pembelian where id = 1;
```

----------

```sql
select * from pembelian;
```

### 📊 Hasil `pembelian`

```
Empty set
```

----------

```sql
select * from log_pembelian;
```

### 📊 Hasil `log_pembelian`

| id | aktivitas                        | waktu               |
|----|----------------------------------|---------------------|
|  1 | Menambah data pembelian.         | 2026-04-08 15:14:20 |
|  2 | Mengubah data pembelian.         | 2026-04-08 15:16:11 |
|  3 | Akan menghapus data pembelian.   | 2026-04-08 15:17:18 |
|  4 | Menghapus data pembelian.        | 2026-04-08 15:17:18 |

----------

# 👁️ VIEW

## 📖 Penjelasan

### View digunakan untuk menampilkan data gabungan dari beberapa tabel menggunakan JOIN.

## 🔗 VIEW

```sql
create view view_pembelian_log as
select 
    pembelian.nama_barang,
    pembelian.jumlah_beli,
    pembelian.total_harga,
    log_pembelian.aktivitas,
    log_pembelian.waktu
from pembelian
inner join log_pembelian
on pembelian.id = log_pembelian.id;
```
---
```sql
select * from view_pembelian_log;
```

### 📊 Hasil
```
Empty set
```

----------

# 🧠 Kesimpulan

-   _Trigger_ digunakan untuk memproses database secara otomatis
    
-   _BEFORE_ → sebelum aksi
    
-   _AFTER_ → setelah aksi
    
-   _View_ adalah tabel virtual dari query _SELECT_ yang mempermudah untuk melihat data
    

----------
