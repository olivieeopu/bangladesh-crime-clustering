# bangladesh-crime-clustering
Analisis eksploratif catatan kriminalitas Bangladesh menggunakan K-Means, Elbow Method, Silhouette Score, visualisasi PCA, dan cluster profiling.

# Bangladesh Crime Pattern Analysis with K-Means

Proyek akademik untuk mengeksplorasi pengelompokan catatan insiden kriminalitas di Bangladesh berdasarkan karakteristik waktu, lokasi, demografi, dan fasilitas wilayah. Analisis menggunakan **K-Means clustering**, **Elbow Method**, **Silhouette Score**, serta **PCA** untuk visualisasi dua dimensi.

## Tujuan

- Mengeksplorasi kualitas data dan menangani missing values serta beberapa anomali.
- Membandingkan jumlah cluster dari K = 2 sampai K = 7.
- Menyusun profil tiga cluster berdasarkan karakteristik numerik dan komposisi kategori insiden.

Unit analisis adalah **baris catatan insiden**, bukan individu atau satu baris per wilayah. Hasil merupakan segmentasi eksploratif, bukan prediksi pelaku, prediksi kejadian mendatang, atau ukuran tingkat kriminalitas per kapita.

## Dataset

Notebook menggunakan `Bangladesh_Crime_Dataset_A.csv`, yang diberikan untuk tugas akademik. Asal publik dan periode cakupan dataset belum dicantumkan dalam notebook.

| Keterangan | Nilai |
|---|---:|
| Baris awal | 6.574 |
| Kolom awal, termasuk indeks `Unnamed: 0` | 26 |
| Baris identik yang dihapus setelah preprocessing | 341 |
| Baris untuk clustering | 6.233 |
| Fitur setelah encoding | 91 |
| Jumlah cluster akhir | 3 |

Variabel mencakup waktu insiden, distrik/divisi, cuaca, populasi, literasi, fasilitas wilayah, dan kategori `crime`. Kategori insiden yang tercatat meliputi `murder`, `rape`, `assault`, `bodyfound`, `kidnap`, dan `robbery`.

## Persiapan Data

1. Menghapus kolom indeks `Unnamed: 0`.
2. Mengisi missing values `incident_month`, `part_of_the_day`, dan `incident_district` menggunakan modus.
3. Mengisi `literacy_rate` menggunakan mean.
4. Menyeragamkan huruf kecil dan spasi pada `incident_division`.
5. Mengganti satu nilai negatif `police_station` dengan missing value, kemudian mengimputasinya menggunakan median.
6. Memeriksa outlier menggunakan IQR. Pemeriksaan ini tidak berarti semua outlier kemudian dihapus.
7. Menghapus kolom `gender_ration` dan menghapus baris identik setelah preprocessing.

Missing values awal ditemukan pada `incident_month` (657), `part_of_the_day` (117), `incident_district` (328), dan `literacy_rate` (328).

## Eksplorasi Asosiasi dan Pemilihan Fitur

Chi-square dalam notebook menunjukkan asosiasi dengan kategori insiden untuk waktu dalam sehari, distrik, divisi, dan musim. `incident_weekday` memiliki p-value sekitar **0,4934** dan kemudian dikeluarkan.

Notebook juga memetakan kategori `crime` menjadi angka 0–5 untuk korelasi Pearson, lalu menghapus `visibility`, `density_per_kmsq`, `precip`, dan `heatindex`. Karena kategori tersebut bersifat nominal, urutan angka ini tidak memiliki makna kuantitatif. Korelasi tersebut tidak digunakan di README ini sebagai bukti kuat relevansi fitur; pemilihan fiturnya perlu dievaluasi ulang.

`crime` dan `crime_tmp` tidak dimasukkan ke input K-Means. Namun, informasi `crime` telah digunakan pada tahap pemilihan fitur, sehingga proses persiapan fitur tidak sepenuhnya bebas dari label.

## Encoding dan Scaling

- One-hot encoding diterapkan pada `part_of_the_day`, `incident_district`, `incident_division`, dan `season`.
- Encoder menggunakan `drop='first'`, `handle_unknown='ignore'`, dan output dense.
- Fitur numerik dan hasil encoding digabung menjadi **91 fitur**.
- StandardScaler diterapkan pada seluruh fitur sebelum clustering.

## Pemilihan Jumlah Cluster

K-Means dicoba pada **K = 2–7** dengan `random_state=42` menggunakan dua ukuran:

| Ukuran | Interpretasi |
|---|---|
| Inertia / Elbow Method | Mengukur jumlah kuadrat jarak observasi ke centroid; penambahan cluster biasanya menurunkannya |
| Silhouette Score | Mengukur kedekatan observasi dengan cluster sendiri dibanding cluster lain; lebih tinggi menunjukkan pemisahan lebih baik menurut ukuran ini |

<img width="566" height="390" alt="Screenshot 2026-09-29 at 15 19 59" src="https://github.com/user-attachments/assets/8c203f60-3389-4218-9b4e-2c6eebb1476c" />
<img width="566" height="386" alt="Screenshot 2026-09-29 at 15 20 16" src="https://github.com/user-attachments/assets/b5189885-9e72-410c-80bd-914603748098" />

Notebook menetapkan **K = 3** sebagai kompromi untuk interpretasi. Pilihan ini tidak dilaporkan sebagai K yang memaksimalkan silhouette. Narasi notebook menyebut elbow sekitar K = 5 dan silhouette cenderung meningkat pada K yang lebih besar. Tabel nilai metrik per K belum disimpan sebagai output numerik.

## Visualisasi Cluster

PCA digunakan untuk memproyeksikan 91 fitur yang sudah distandardisasi ke dua komponen untuk visualisasi. K-Means tetap dilatih pada fitur hasil scaling, bukan hanya dua komponen PCA.

![Uploading Screenshot 2026-09-29 at 15.20.32.png…]()

Jarak dan overlap pada proyeksi dua dimensi tidak menggambarkan seluruh struktur ruang fitur. Proporsi explained variance PCA belum dilaporkan dalam notebook.

## Profil Setiap Cluster

Nilai berikut merupakan **rata-rata atribut wilayah pada catatan insiden dalam cluster**, bukan jumlah penduduk cluster dan bukan rata-rata setiap wilayah dengan bobot yang sama.

| Karakteristik | Cluster 0 | Cluster 1 | Cluster 2 |
|---|---:|---:|---:|
| Total population | 547.118 | 266.322 | 8.906.039 |
| Average household size | 4,44 | 4,36 | 8,42 |
| Literacy rate | 53,07 | 53,71 | 73,73 |
| Playground | 33,46 | 75,70 | 99,00 |
| Park | 1,89 | 0,86 | 17,00 |
| Police station | 4,46 | 2,16 | 59,93 |
| School | 50,17 | 44,23 | 242,00 |
| College | 8,05 | 9,98 | 64,00 |

### Cluster 0 — Profil Populasi Menengah dalam Perbandingan Ini

Rata-rata populasi wilayah lebih tinggi daripada Cluster 1, tetapi jauh di bawah Cluster 2. Kategori terbanyak adalah **murder (24,42%)**, diikuti **bodyfound (23,01%)**. Proporsi tersebut menggambarkan komposisi catatan di cluster, bukan risiko terjadinya insiden bagi penduduknya.

### Cluster 1 — Profil Populasi Lebih Kecil

Memiliki rata-rata populasi sekitar 266 ribu dan lebih sedikit kantor polisi dibanding dua cluster lain. Namun, jumlah playground rata-rata lebih tinggi daripada Cluster 0, sehingga tidak tepat menyatakan seluruh fasilitasnya lebih sedikit. Kategori terbanyak adalah **rape (26,53%)** dan **assault (25,51%)**.

### Cluster 2 — Profil Populasi dan Fasilitas Lebih Besar

Memiliki rata-rata populasi, literasi, dan sejumlah fasilitas yang paling tinggi dalam perbandingan ini. Kategori terbanyak adalah **bodyfound (27,79%)**, diikuti **robbery (18,45%)**. Penamaan metropolitan memerlukan verifikasi geografis tambahan, sehingga dipakai deskripsi berdasarkan atribut yang terukur.

### Komposisi Kategori Insiden

| Kategori | Cluster 0 | Cluster 1 | Cluster 2 |
|---|---:|---:|---:|
| Assault | 16,53% | 25,51% | 17,84% |
| Bodyfound | 23,01% | 15,31% | 27,79% |
| Kidnap | 10,09% | 10,20% | 7,77% |
| Murder | 24,42% | 16,33% | 14,32% |
| Rape | 18,38% | 26,53% | 13,83% |
| Robbery | 7,57% | 6,12% | 18,45% |

Persentase dihitung di dalam masing-masing cluster dan dapat berbeda tipis dari 100% akibat pembulatan.

## Temuan Utama

Tiga cluster menunjukkan perbedaan profil populasi, fasilitas wilayah, dan komposisi kategori insiden. Cluster 2 memiliki atribut populasi dan sejumlah fasilitas yang lebih besar, sementara Cluster 0 dan Cluster 1 memiliki komposisi kategori insiden yang berbeda. Temuan ini mendeskripsikan struktur dataset; tidak membuktikan bahwa karakteristik demografi atau fasilitas menyebabkan kriminalitas.


## Tools

Python, pandas, NumPy, Matplotlib, SciPy, dan scikit-learn (OneHotEncoder, StandardScaler, KMeans, Silhouette Score, PCA).


