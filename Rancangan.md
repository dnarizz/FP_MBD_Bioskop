## Final Project Manajemen Basis Data

---

## 1. Conceptual Data Model (CDM)

### 1.1 Daftar Entitas

| Entitas | Deskripsi |
|---|---|
| Film | Data film yang ditayangkan |
| Studio | Data ruang studio bioskop |
| Kursi | Master kursi fisik milik suatu studio |
| Jadwal Tayang | Jadwal penayangan film pada studio dan waktu tertentu |
| Kursi Jadwal | Status ketersediaan kursi untuk jadwal tayang tertentu (entitas penghubung) |
| Pelanggan | Data pelanggan yang melakukan pemesanan |
| Pemesanan | Transaksi pemesanan tiket oleh pelanggan |
| Detail Pemesanan | Rincian kursi yang dipesan dalam satu pemesanan |
| Pembayaran | Pencatatan pembayaran atas suatu pemesanan |

### 1.2 Relasi Antar Entitas

| Relasi | Kardinalitas | Keterangan |
|---|---|---|
| Studio – Kursi | 1 : N | Satu studio memiliki banyak kursi |
| Studio – Jadwal Tayang | 1 : N | Satu studio digunakan untuk banyak jadwal |
| Film – Jadwal Tayang | 1 : N | Satu film memiliki banyak jadwal tayang |
| Jadwal Tayang – Kursi Jadwal | 1 : N | Satu jadwal menghasilkan banyak baris status kursi |
| Kursi – Kursi Jadwal | 1 : N | Satu kursi fisik muncul pada banyak jadwal berbeda |
| Pelanggan – Pemesanan | 1 : N | Satu pelanggan dapat melakukan banyak pemesanan |
| Jadwal Tayang – Pemesanan | 1 : N | Satu jadwal dapat dipesan oleh banyak pemesanan berbeda |
| Pemesanan – Detail Pemesanan | 1 : N | Satu pemesanan dapat mencakup beberapa kursi (mendukung TR3) |
| Kursi Jadwal – Detail Pemesanan | 1 : 1 | Satu slot kursi-jadwal hanya boleh muncul pada satu detail pemesanan aktif |
| Pemesanan – Pembayaran | 1 : 1 | Satu pemesanan memiliki satu catatan pembayaran |

---
```
CREATE TABLE film (
    id_film         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    judul_film      VARCHAR(150) NOT NULL,
    genre           VARCHAR(20) NOT NULL,
    durasi_menit    INTEGER NOT NULL,
    sutradara       VARCHAR(30) NOT NULL,
    deskripsi       TEXT NOT NULL
);

-- 2. studio
CREATE TABLE studio (
    id_studio       INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nama_studio     VARCHAR(50) NOT NULL,
    tipe_studio     VARCHAR(20) NOT NULL CHECK (tipe_studio IN ('reguler', 'vip', '3d')),
    kapasitas       INTEGER NOT NULL
);

-- 3. kursi
CREATE TABLE kursi (
    id_kursi        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_studio       INTEGER NOT NULL REFERENCES studio(id_studio),
    nomor_kursi     VARCHAR(5) NOT NULL,
    CONSTRAINT uq_kursi_studio_nomor UNIQUE (id_studio, nomor_kursi)
);

-- 4. jadwal_tayang
CREATE TABLE jadwal_tayang (
    id_jadwal       INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_film         INTEGER NOT NULL REFERENCES film(id_film),
    id_studio       INTEGER NOT NULL REFERENCES studio(id_studio),
    tanggal_tayang  DATE NOT NULL,
    jam_tayang      TIME NOT NULL,
    harga_tiket     NUMERIC(10,2) NOT NULL
);

-- 5. kursi_jadwal
CREATE TABLE kursi_jadwal (
    id_kursi_jadwal INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_jadwal       INTEGER NOT NULL REFERENCES jadwal_tayang(id_jadwal),
    id_kursi        INTEGER NOT NULL REFERENCES kursi(id_kursi),
    status_kursi    VARCHAR(20) NOT NULL DEFAULT 'available' CHECK (status_kursi IN ('available', 'booked')),
    CONSTRAINT uq_kursijadwal_jadwal_kursi UNIQUE (id_jadwal, id_kursi)
);

-- 6. pelanggan
CREATE TABLE pelanggan (
    id_pelanggan    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nama_pelanggan  VARCHAR(50) NOT NULL,
    email           VARCHAR(100) NOT NULL UNIQUE,
    no_hp           VARCHAR(20) NOT NULL
);

-- 7. pemesanan
CREATE TABLE pemesanan (
    id_pemesanan     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_pelanggan     INTEGER NOT NULL REFERENCES pelanggan(id_pelanggan),
    id_jadwal        INTEGER NOT NULL REFERENCES jadwal_tayang(id_jadwal),
    tanggal_pesan    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status_pemesanan VARCHAR(20) NOT NULL DEFAULT 'pending' CHECK (status_pemesanan IN ('pending', 'confirmed', 'cancelled')),
    total_harga      NUMERIC(10,2) NOT NULL DEFAULT 0
);

-- 8. detail_pemesanan
CREATE TABLE detail_pemesanan (
    id_detail        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_pemesanan     INTEGER NOT NULL REFERENCES pemesanan(id_pemesanan),
    id_kursi_jadwal  INTEGER NOT NULL UNIQUE REFERENCES kursi_jadwal(id_kursi_jadwal),
    harga            NUMERIC(10,2) NOT NULL
);

-- 9. pembayaran
CREATE TABLE pembayaran (
    id_pembayaran     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_pemesanan      INTEGER NOT NULL UNIQUE REFERENCES pemesanan(id_pemesanan),
    metode_pembayaran VARCHAR(30) NOT NULL CHECK (metode_pembayaran IN ('cash', 'debit', 'credit', 'e-wallet')),
    status_pembayaran VARCHAR(30) NOT NULL DEFAULT 'pending' CHECK (status_pembayaran IN ('success', 'failed')),
    tanggal_bayar     TIMESTAMP NULL
);
```
## 2. Physical Data Model (PDM)

### 2.1 Tabel `film`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_film | INT | PK, AUTO_INCREMENT |
| judul_film | VARCHAR(150) | NOT NULL  |
| genre | VARCHAR(20) | |
| durasi_menit | INT | |
| sutradara | VARCHAR(30) | |
| deskripsi | TEXT | |

### 2.2 Tabel `studio`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_studio | INT | PK, AUTO_INCREMENT |
| nama_studio | VARCHAR(50) | NOT NULL |
| tipe_studio | VARCHAR(20)('reguler','vip','3d') | NOT NULL |
| kapasitas | INT | NOT NULL |

### 2.3 Tabel `kursi`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_kursi | INT | PK, AUTO_INCREMENT |
| id_studio | INT | FK → studio.id_studio, NOT NULL |
| nomor_kursi | VARCHAR(5) | NOT NULL, mis. A1, B5 |
| | | UNIQUE(id_studio, nomor_kursi) |

### 2.4 Tabel `jadwal_tayang`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_jadwal | INT | PK, AUTO_INCREMENT |
| id_film | INT | FK → film.id_film, NOT NULL |
| id_studio | INT | FK → studio.id_studio, NOT NULL |
| tanggal_tayang | DATE | NOT NULL |
| jam_tayang | TIME | NOT NULL |
| harga_tiket | DECIMAL(10,2) | NOT NULL |

### 2.5 Tabel `kursi_jadwal`  
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_kursi_jadwal | INT | PK, AUTO_INCREMENT |
| id_jadwal | INT | FK → jadwal_tayang.id_jadwal, NOT NULL |
| id_kursi | INT | FK → kursi.id_kursi, NOT NULL |
| status_kursi | ENUM('available','booked') | DEFAULT 'available' |
| | | UNIQUE(id_jadwal, id_kursi) |

### 2.6 Tabel `pelanggan`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_pelanggan | INT | PK, AUTO_INCREMENT |
| nama_pelanggan | VARCHAR(50) | NOT NULL |
| email | VARCHAR(100) | UNIQUE, NOT NULL |
| no_hp | VARCHAR(20) | |

### 2.7 Tabel `pemesanan`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_pemesanan | INT | PK, AUTO_INCREMENT |
| id_pelanggan | INT | FK → pelanggan.id_pelanggan, NOT NULL |
| id_jadwal | INT | FK → jadwal_tayang.id_jadwal, NOT NULL |
| tanggal_pesan | DATETIME | DEFAULT CURRENT_TIMESTAMP |
| status_pemesanan | ENUM('pending','confirmed','cancelled') | DEFAULT 'pending' |
| total_harga | DECIMAL(10,2) | DEFAULT 0 |

### 2.8 Tabel `detail_pemesanan`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_detail | INT | PK, AUTO_INCREMENT |
| id_pemesanan | INT | FK → pemesanan.id_pemesanan, NOT NULL |
| id_kursi_jadwal | INT | FK → kursi_jadwal.id_kursi_jadwal, NOT NULL |
| harga | DECIMAL(10,2) | NOT NULL  |


### 2.9 Tabel `pembayaran`
| Kolom | Tipe Data | Constraint |
|---|---|---|
| id_pembayaran | INT | PK, AUTO_INCREMENT |
| id_pemesanan | INT | FK → pemesanan.id_pemesanan, NOT NULL, UNIQUE |
| metode_pembayaran | ENUM('cash','debit','credit','e-wallet') | NOT NULL |
| status_pembayaran | ENUM('success','failed') |
| tanggal_bayar | DATETIME | |

---
### CDM
<img width="921" height="743" alt="Bioskop_CDM-2026-06-23_19-01" src="https://github.com/user-attachments/assets/9a542f01-281a-47fe-a162-7067c852ccd1" />



### PDM
<img width="921" height="754" alt="Bioskop_CDM_Physical_Export_3-2026-06-23_19-04" src="https://github.com/user-attachments/assets/9976193c-d1ef-4a1e-81df-5184ad69b62a" />



## 3. Pemetaan Komponen Penilaian

### 3.1 Trigger (T1–T4)
| Kode | Nama Trigger | Tabel Sumber | Aksi |
|---|---|---|---|
| T1 | Update status kursi → booked | `detail_pemesanan` (AFTER INSERT) | UPDATE `kursi_jadwal.status_kursi` = 'booked' |
| T2 | Kembalikan status kursi → available | `pemesanan` / `pembayaran` (AFTER UPDATE, saat status = 'cancelled'/'failed') | UPDATE `kursi_jadwal.status_kursi` = 'available' |
| T3 | Validasi kursi belum terisi | `detail_pemesanan` (BEFORE INSERT) | Cek `kursi_jadwal.status_kursi`, tolak (RAISE ERROR) jika sudah 'booked' |
| T4 | Hitung total harga otomatis | `detail_pemesanan` (AFTER INSERT) | UPDATE `pemesanan.total_harga` = SUM(harga) dari seluruh detail terkait |
| T5 | Otomatisasi data dari kursi dan jadwal tayang | `kursi_jadwal` (AFTER INSERT) | INSERT `jadwal_tayang` | 
T5 — Otomatisasi data kursi_jadwal setalah menambahkan data jadwal_tayang baru
-- AFTER INSERT jadwal_tayang
-- Aktif ketika INSERT jadwal_tayang


```
-- ============================================================
-- T3 — Validasi kursi belum terisi
-- `detail_pemesanan` (BEFORE INSERT)
-- ============================================================

CREATE OR REPLACE FUNCTION fn_validasi_kursi_tersedia()
RETURNS TRIGGER AS $$
DECLARE
    v_status kursi_jadwal.status_kursi%TYPE;
BEGIN
    SELECT status_kursi INTO v_status
    FROM kursi_jadwal
    WHERE id_kursi_jadwal = NEW.id_kursi_jadwal
    FOR UPDATE;

    IF v_status IS NULL THEN
        RAISE EXCEPTION 'Kursi-jadwal dengan id % tidak ditemukan', NEW.id_kursi_jadwal;
    END IF;

    IF v_status <> 'available' THEN
        RAISE EXCEPTION 'Kursi tidak tersedia (status: %). Tidak dapat memesan kursi yang sama.', v_status;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_t3_validasi_kursi
BEFORE INSERT ON detail_pemesanan
FOR EACH ROW
EXECUTE FUNCTION fn_validasi_kursi_tersedia();


-- ============================================================
-- T1 — Update status kursi menjadi 'booked'
-- AFTER INSERT pada detail_pemesanan
-- ============================================================
CREATE OR REPLACE FUNCTION fn_update_status_kursi_booked()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE kursi_jadwal
    SET status_kursi = 'booked'
    WHERE id_kursi_jadwal = NEW.id_kursi_jadwal;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_t1_update_status_kursi
AFTER INSERT ON detail_pemesanan
FOR EACH ROW
EXECUTE FUNCTION fn_update_status_kursi_booked();


-- ============================================================
-- T4 — Hitung ulang total_harga pada pemesanan secara otomatis
-- AFTER INSERT pada detail_pemesanan
-- ============================================================
CREATE OR REPLACE FUNCTION fn_hitung_total_harga()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE pemesanan
    SET total_harga = (
        SELECT COALESCE(SUM(harga), 0)
        FROM detail_pemesanan
        WHERE id_pemesanan = NEW.id_pemesanan
    )
    WHERE id_pemesanan = NEW.id_pemesanan;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_t4_hitung_total_harga
AFTER INSERT ON detail_pemesanan
FOR EACH ROW
EXECUTE FUNCTION fn_hitung_total_harga();



-- ============================================================
-- T2 — Kembalikan status kursi menjadi 'available'
-- AFTER UPDATE OF status_pemesanan ON pemesanan
-- Aktif ketika status_pemesanan berubah MENJADI 'cancelled'
-- ============================================================
CREATE OR REPLACE FUNCTION fn_kembalikan_status_kursi_dari_pemesanan()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status_pemesanan = 'cancelled' AND OLD.status_pemesanan IS DISTINCT FROM 'cancelled' THEN
        UPDATE kursi_jadwal kj
        SET status_kursi = 'available'
        FROM detail_pemesanan dp
        WHERE dp.id_pemesanan = NEW.id_pemesanan
          AND kj.id_kursi_jadwal = dp.id_kursi_jadwal;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_t2_kembalikan_status_kursi
AFTER UPDATE OF status_pemesanan ON pemesanan
FOR EACH ROW
EXECUTE FUNCTION fn_kembalikan_status_kursi_dari_pemesanan();


-- ============================================================
-- T5 — Otomatisasi data kursi_jadwal setalah menambahkan data jadwal_tayang baru
-- AFTER INSERT jadwal_tayang
-- Aktif ketika INSERT jadwal_tayang
-- ============================================================
CREATE OR REPLACE FUNCTION fn_buka_kursi_jadwal()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO kursi_jadwal (id_jadwal, id_kursi, status_kursi)
    SELECT NEW.id_jadwal, id_kursi, 'available'
    FROM kursi
    WHERE id_studio = NEW.id_studio;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_buka_kursi_otomatis
AFTER INSERT ON jadwal_tayang
FOR EACH ROW
EXECUTE FUNCTION fn_buka_kursi_jadwal();

```



### 3.2 Indexing (I1, I2, I3, I6)
| Kode | Tabel | Kolom | Jenis |
|---|---|---|---|
| I1 | film | judul_film | Single-column |
| I2 | jadwal_tayang | tanggal_tayang | Single-column |
| I3 | kursi_jadwal | (id_jadwal, id_kursi) | Composite |
| I6 | jadwal_tayang | (id_studio, tanggal_tayang) | Composite |


```
-- I1 — Single-column index untuk pencarian film berdasarkan judul
CREATE INDEX idx_film_judul ON film (judul_film);

-- I2 — Single-column index untuk filter jadwal berdasarkan tanggal tayang
CREATE INDEX idx_jadwal_tanggal ON jadwal_tayang (tanggal_tayang);

-- I3 — Composite index untuk pengecekan ketersediaan kursi per jadwal
-- Dibuat sebagai UNIQUE INDEX agar sekaligus mengembalikan aturan bisnis
-- "satu slot kursi-jadwal hanya boleh muncul satu kali" yang sebelumnya
-- diberlakukan melalui constraint uq_kursijadwal_jadwal_kursi
CREATE UNIQUE INDEX idx_kursijadwal_jadwal_kursi ON kursi_jadwal (id_jadwal, id_kursi);

-- I6 — Composite index untuk filter jadwal berdasarkan studio dan tanggal
CREATE INDEX idx_jadwal_studio_tanggal ON jadwal_tayang (id_studio, tanggal_tayang);
```

### 3.3 Role & Privilege (R1, R2, R3)
| Kode | Role | Hak Akses (rencana awal) |
|---|---|---|
| R1 | Admin | ALL PRIVILEGES pada seluruh tabel |
| R2 | Kasir | SELECT, INSERT, UPDATE pada `pemesanan`, `detail_pemesanan`, `pembayaran`, `kursi_jadwal`; SELECT pada `film`, `jadwal_tayang`, `kursi`, `studio` |
| R3 | Customer Service | SELECT (read-only) pada `pemesanan`, `detail_pemesanan`, `pelanggan`, `pembayaran` |

> Catatan: R1–R3 diimplementasikan sebagai MySQL user/role (`CREATE USER`, `GRANT`) pada tahap Administrasi Database, bukan sebagai tabel aplikasi. Tidak menambah entitas baru pada CDM/PDM.

```
-- ============================================================
-- R1 — ADMIN
-- Hak akses penuh terhadap seluruh tabel
-- ============================================================
CREATE ROLE admin_bioskop WITH LOGIN PASSWORD 'Admin#2026';

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin_bioskop;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO admin_bioskop;


-- ============================================================
-- R2 — KASIR
-- - SELECT, INSERT, UPDATE pada: pemesanan, detail_pemesanan,
--   pembayaran, kursi_jadwal (operasional transaksi penjualan)
-- - SELECT, INSERT pada: pelanggan (registrasi pelanggan baru
--   di loket — penyesuaian dari rancangan awal yang sebelumnya
--   belum mencantumkan akses ini)
-- - SELECT pada: film, jadwal_tayang, kursi, studio (data referensi)
-- - TIDAK ada hak DELETE maupun akses ke tabel master film/studio
-- ============================================================
CREATE ROLE kasir_bioskop WITH LOGIN PASSWORD 'Kasir#2026';

GRANT SELECT, INSERT, UPDATE
    ON pemesanan, detail_pemesanan, pembayaran, kursi_jadwal
    TO kasir_bioskop;

GRANT SELECT, INSERT
    ON pelanggan
    TO kasir_bioskop;

GRANT SELECT
    ON film, jadwal_tayang, kursi, studio
    TO kasir_bioskop;


-- ============================================================
-- R3 — CUSTOMER SERVICE
-- - Hanya SELECT (read-only) pada: pemesanan, detail_pemesanan,
--   pelanggan, pembayaran — untuk keperluan penanganan komplain
--   dan verifikasi data pemesanan
-- - TIDAK ada hak INSERT, UPDATE, maupun DELETE pada tabel manapun
-- ============================================================
CREATE ROLE cs_bioskop WITH LOGIN PASSWORD 'CSBioskop#2026';

GRANT SELECT
    ON pemesanan, detail_pemesanan, pelanggan, pembayaran
    TO cs_bioskop;
```

### 3.4 Transaksi Database
- **TR1 (wajib)**: Pemesanan + Pembayaran dalam satu transaksi — melibatkan INSERT ke `pemesanan`, `detail_pemesanan`, `pembayaran`, serta trigger T1/T3/T4. Struktur skema di atas sudah mendukung skenario ini sepenuhnya.
- **Skenario kedua**: masih menunggu kesepakatan kelompok (lihat Bagian 4).

---

## 4. Catatan Revisi — Tergantung Pilihan Skenario Transaksi Kedua

| Jika kelompok memilih | Dampak terhadap Skema |
|---|---|
| **TR2 – Pembatalan pemesanan** | Tidak perlu tabel baru. `status_pemesanan` dan `status_pembayaran` sudah mengakomodasi nilai 'cancelled'/'failed'. Opsional: tambah kolom `alasan_pembatalan VARCHAR(150)` pada `pemesanan` jika ingin dicatat. |
| **TR3 – Pemesanan multi-kursi** | Tidak ada perubahan — sudah didukung penuh oleh relasi 1:N antara `pemesanan` dan `detail_pemesanan`. |
| **TR4 – Reschedule jadwal** | **Memerlukan tabel tambahan**, misal `riwayat_reschedule` (id_riwayat PK, id_pemesanan FK, id_jadwal_lama FK, id_jadwal_baru FK, tanggal_reschedule DATETIME). Ini akan menambah satu entitas baru pada CDM/PDM. |

Apabila kelompok memilih kombinasi TR1 + TR2 atau TR1 + TR3, **skema pada dokumen ini sudah final dan dapat langsung dilanjutkan ke tahap normalisasi formal serta penulisan DDL.** Apabila TR4 dipilih, akan dilakukan revisi penambahan satu tabel sebelum melanjutkan ke DDL.

