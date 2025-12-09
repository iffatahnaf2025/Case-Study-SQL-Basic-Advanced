# 📘 **CASE STUDY SQL: BASIC — ADVANCE**

Dokumentasi ini merupakan case study SQL yang menggabungkan kemampuan basic hingga advanced SQL menggunakan PostgreSQL pada database `dvdrental` dan skema custom `dibimbing`.  
Seluruh tugas, query, dan insight dirancang untuk menunjukkan kemampuan analisis data, manipulasi dataset, serta penggunaan teknik SQL tingkat lanjut yang relevan untuk kebutuhan seorang **Data Analyst**.

## **Tools yang Digunakan**
- **DBeaver:** Sebagai _database management tool_ untuk menjalankan query SQL dan mengelola skema database.
- **PostgreSQL (`dvdrental`):** Sebagai database utama untuk case study.

---

# 🧩 **Case Study 1: Basic SQL Queries**

Proyek ini mendokumentasikan serangkaian query SQL dasar yang dibuat untuk praktik manipulasi dan pengambilan data dari dua skema database: `dibimbing` (pembuatan skema baru) dan `dvdrental` (kueri pada data yang sudah ada).

Case study ini mencakup pembuatan tabel (DDL), penyisipan data (DML), fungsi agregasi, logika kondisional (CASE), penggabungan tabel (JOIN), dan operasi set (UNION).

---

## **1. Membuat Database `dibimbing` & Tabel `students`**

### 1.1 Create Database
Membuat database baru bernama `dibimbing` sebagai tempat untuk menyimpan tabel custom.
```sql
CREATE DATABASE dibimbing;
```

### 1.2 Create Table
Membuat tabel `students` dengan kolom biodata mahasiswa. Kolom `id` dijadikan primary key untuk memastikan nilai unik.
```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    nama VARCHAR(100),
    institute VARCHAR(100),
    berat_badan FLOAT,
    tinggi_badan FLOAT
);
```

### 1.3 Insert Data
Mengisi tabel dengan 5 record data mahasiswa sebagai dataset awal untuk eksplorasi.
```sql
INSERT INTO students (id, nama, institute, berat_badan, tinggi_badan)
VALUES
(1, 'Ayman Ahnaf', 'Universitas Muhammadiyah Yogyakarta', 65.5, 172.0),
(2, 'Nadia Putri', 'Universitas Indonesia', 54.2, 160.3),
(3, 'Rizky Pratama', 'Institut Teknologi Bandung', 70.8, 178.4),
(4, 'Siti Nurhaliza', 'Universitas Airlangga', 49.7, 158.9),
(5, 'Dimas Saputra', 'Universitas Gadjah Mada', 80.1, 182.5);
```

## **2. Query Dasar pada Database `dvdrental`**

### 2.1 Filter Actor Berdasarkan Nama
Menampilkan daftar aktor yang memiliki `first_name` tertentu menggunakan operator `IN` untuk mempersingkat multiple OR
```sql
SELECT first_name, last_name
FROM actor
WHERE first_name IN ('Jennifer', 'Nick', 'Ed');
```

### 2.2 Summary Pembayaran Amount >5.99
- Mengambil record pembayaran dengan amount > 5.99.
- Menampilkan total amount per payment_id.
- Dua subquery digunakan untuk menghitung total transaksi dan total nominal secara keseluruhan.
- Menunjukkan penggunaan kombinasi agregasi & subquery.
``` sql
SELECT
    p.payment_id,
    SUM(p.amount) AS "total_amount (>5.99)",
    (SELECT COUNT(*) FROM payment WHERE amount > 5.99) AS total_transactions,
    (SELECT SUM(amount) FROM payment WHERE amount > 5.99) AS total_payment
FROM payment p
WHERE p.amount > 5.99
GROUP BY p.payment_id;
```

### 2.3 Klasifikasi Film Berdasarkan Durasi
Menggunakan `CASE` untuk melakukan segmentasi durasi film menjadi beberapa kategori. Teknik ini umum digunakan untuk segmentasi analitis.
```sql
SELECT
    title,
    length,
    CASE
        WHEN length > 100 THEN 'Over 100 menit'
        WHEN length BETWEEN 87 AND 100 THEN '87-100 menit'
        WHEN length BETWEEN 72 AND 86 THEN '72-86 menit'
        WHEN length < 72 THEN 'Under 72 menit'
        ELSE 'Unknown'
    END AS kategori_durasi
FROM film;
```

### 2.4. `JOIN` Rental & Payment
Menggabungkan tabel `rental` dan `payment` berdasarkan foreign key `rental_id`. Hal ini berguna untuk melihat hubungan antara aktivitas rental dan pembayaran pelanggan.
```sql
SELECT
    r.rental_id,
    r.rental_date,
    p.payment_id,
    p.amount
FROM rental r
JOIN payment p ON r.rental_id = p.rental_id
ORDER BY p.amount ASC;
```

### 2.5 `UNION` Address dengan City Tertentu
Menggabungkan dua query dengan `UNION` untuk mendapatkan daftar alamat dari dua kota berbeda.
`UNION` otomatis menghapus duplikasi (berbeda dengan `UNION ALL`).
```sql
SELECT address_id, address, city_id
FROM address
WHERE city_id = 42

UNION

SELECT address_id, address, city_id
FROM address
WHERE city_id = 300;
```

---

# **🧠 Case Study 2: Intermediate & Advance SQL Queries**

Case study ini menyajikan rangkaian *intermediate* hingga *advancce* SQL queries yang berfokus pada analisis data transaksi pada database `dvdrental`. Studi ini dirancang untuk memperkuat kemampuan dalam memanipulasi data relasional menggunakan teknik SQL tingkat menengah, termasuk penggunaan JOIN, subqueries, window functions, aggregation, data transformation, serta conditional logic untuk menghasilkan insight yang lebih mendalam.

---

## **1. Subqueries**

### 1.1 Customer dengan Average Payment > Global Average
- Menghitung rata-rata pembayaran setiap customer.
- HAVING digunakan untuk memfilter customer yang rata-ratanya lebih besar dari rata-rata global pembayaran (subquery).
- Berguna untuk mengidentifikasi high-paying customers.
```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    ROUND(AVG(p.amount), 2) AS avg_customer_payment
FROM payment p
JOIN customer c ON p.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING AVG(p.amount) > (SELECT AVG(amount) FROM payment)
ORDER BY avg_customer_payment DESC;
```

### 1.2 Film Berdurasi di Atas Rata-rata
Menampilkan film-film yang memiliki durasi lebih panjang dari rata-rata keseluruhan film. Contoh penggunaan subquery scalar baik pada `SELECT` maupun `WHERE`.
```sql
SELECT
    film_id,
    title,
    length,
    (SELECT ROUND(AVG(length), 2) FROM film) AS avg_length
FROM film
WHERE length > (SELECT AVG(length) FROM film)
ORDER BY length ASC;
```

### 1.3 Actor yang Membintangi Tepat 1 Film
Subquery correlated digunakan untuk menghitung jumlah film per aktor. Query ini mencari aktor yang hanya membintangi satu film — namun pada dataset dvdrental tidak ada aktor yang memenuhi kondisi ini.
```sql
SELECT
    a.actor_id,
    CONCAT(a.first_name, ' ', a.last_name) AS full_name,
    (SELECT COUNT(*) FROM film_actor fa WHERE fa.actor_id = a.actor_id) AS count_film
FROM actor a
WHERE (
    SELECT COUNT(*) FROM film_actor fa WHERE fa.actor_id = a.actor_id
) = 1;
```
*Catatan: Pada dataset dvdrental, tidak ada aktor yang hanya membintangi satu film.*

## **2. Window Functions**

### 2.1 Ranking Film Berdasarkan Rental Rate (`RANK`)
`RANK()` memberikan peringkat dengan _gaps_ jika ada nilai rental_rate yang sama. Hal ini digunakan untuk melihat film dengan biaya rental tertinggi.
```sql
SELECT
    film_id,
    title,
    rental_rate,
    RANK() OVER (ORDER BY rental_rate DESC) AS rank_rental_rate
FROM film
ORDER BY rank_rental_rate;
```

### 2.2 Ranking Customer Berdasarkan Total Payment (`DENSE_RANK`)
`DENSE_RANK()` memberikan peringkat tanpa gap. Hal ini digunakan untuk mengidentifikasi pelanggan dengan kontribusi pembayaran terbesar.
```sql
SELECT
    customer_id,
    SUM(amount) AS total_payment,
    DENSE_RANK() OVER (ORDER BY SUM(amount) DESC) AS rank_customer
FROM payment
GROUP BY customer_id
ORDER BY rank_customer;
```

### 2.3 Row Number Berdasarkan Tahun Rilis
Menambahkan nomor urut unik untuk setiap baris berdasarkan tahun rilis. Cocok untuk kebutuhan indexing dan pagination.
```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY release_year ASC) AS row_num,
    film_id,
    title,
    release_year
FROM film;
```

## **3. Common Table Expressions (CTE)**

### 3.1 Frequent Customers (>10 Transactions)
Menggunakan CTE untuk menyederhanakan query kompleks. Menampilkan pelanggan yang melakukan lebih dari 10 transaksi pembayaran dan menambahkan nama pelanggan.
```sql
WITH frequent_customers AS (
    SELECT
        customer_id,
        COUNT(payment_id) AS total_transactions
    FROM payment
    GROUP BY customer_id
    HAVING COUNT(payment_id) > 10
)
SELECT
    fc.customer_id,
    fc.total_transactions,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name
FROM frequent_customers fc
JOIN customer c ON c.customer_id = fc.customer_id
ORDER BY total_transactions DESC;
```

## 3.2 Film dengan Rental Terbanyak
Mengidentifikasi film dengan jumlah rental tertinggi menggunakan CTE. Melibatkan join antar tabel: `film`, `inventory`, dan `rental`.
```sql
WITH film_rental_count AS (
    SELECT
        f.film_id,
        f.title,
        COUNT(r.rental_id) AS total_rentals
    FROM film f
    JOIN inventory i ON f.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
    GROUP BY f.film_id, f.title
)
SELECT *
FROM film_rental_count
ORDER BY total_rentals DESC;
```

## **4. Case-Based Classification**

### 4.1 Kategori Film Berdasarkan Rental Rate
Melakukan segmentasi film berdasarkan biaya rental. Kategori ini membantu pemodelan pricing dan analisis produk.
```sql
SELECT
    film_id,
    title,
    rental_rate,
    CASE
        WHEN rental_rate > 4 THEN 'Premium'
        WHEN rental_rate BETWEEN 2 AND 4 THEN 'Regular'
        WHEN rental_rate < 2 THEN 'Budget'
        ELSE 'Unknown'
    END AS category
FROM film
ORDER BY rental_rate DESC;
```

### 4.2 Customer Segmentation Berdasarkan Total Payment
Segmentasi pelanggan berdasarkan total pembayaran yang dilakukan. Teknik ini umum digunakan untuk analisis customer value dan strategi marketing.
```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(p.amount) AS total_payment,
    CASE
        WHEN SUM(p.amount) > 100 THEN 'High Value Customer'
        WHEN SUM(p.amount) BETWEEN 50 AND 100 THEN 'Medium Value Customer'
        WHEN SUM(p.amount) < 50 THEN 'Low Value Customer'
        ELSE 'No Transactions'
    END AS customer_category
FROM customer c
LEFT JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_payment DESC;
```
