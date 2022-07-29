# Laporan Proyek Machine Learning - Joseph Setiawan

## Project Overview
Banyak aplikasi yang terdaftar dalam Google Play Store. Proyek ini dibuat untuk menciptakan suatu sistem rekomendasi aplikasi yang sesuai dengan keinginan pengguna.


Sumber yang digunakan untuk referensi data preparation proyek ini :
[Android-App-Recommendation](https://www.kaggle.com/code/nandalald/android-app-recommendation)

Dataset yang digunakan :
[Google Play Store Apps](https://www.kaggle.com/datasets/lava18/google-play-store-apps/code?datasetId=49864&searchQuery=rec)
## Business Understanding
Proyek ini berisi pembuatan rekomendasi aplikasi.

### Problem Statements
Masalah yang diselesaikan :
- Bagaimana membersihkan data yang dibutuhkan?
- Apa algoritma sistem rekomendasi yang diperlukan untuk memberikan rekomendasi ?

### Goals
Tujuan yang ingin dicapai :
- Pembersihkan data yang digunakan untuk sistem rekomendasi
- Penggunaan algoritma sistem rekomendasi 


## Data Understanding
Berikut deskripsi kolom data yang digunakan:
- App, yaitu nama aplikasi
- Category, yaitu kategori aplikasi
- Rating, yaitu penilaian dari pengguna
- Reviews, yaitu jumlah review dari pengguna
- Size, yaitu ukuran aplikasi 
- Installs, yaitu jumlah unduh atau pasang aplikasi
- Type, yaitu jenis aplikasi, apakah berbayar atau gratis
- Price, yaitu harga untuk menggunakan aplikasi
- Content Rating, yaitu rating umur pengguna yang sesuai dengan aplikasi
- Genres, yaitu bagian detail dari kategori yang mendaftarkan semua genre yang cocok dengan aplikasi
- Last Updated, yaitu kapan aplikasi ini terakhir diperbaharui
- Current Ver, yaitu angka versi yang disematkan pembua aplikasi
- Android Ver, yaitu versi android yang bisa menjalankan aplikasi

## Data Preparation
- Pengecekan deskripsi dan missing value file
- Menyaring rating yang tidak lebih dari 5. Ketika melihat graf, ada rating yang melebihi 5, yang merupakan data tidak wajar. Maka itu data tersebut dihilangkan beserta data tanpa rating.
- Mengganti tipe data Review menjadi angka
- Melihat nilai-nilai yang hilang
- Menghilangkan kolom yang tidak digunakan untuk pembuatan sistem. Last Updated dan Current Ver bisa dipakai sebagai database saja. Genres sudah diwakilkkan Category secara garis besarnya, sehingga Genres dihilangkan untuk mengurangi dimensi
- Menghilangkan data dengan missing value, karena dengan data yang tanpa missing value pun, data yang tersisa masih berjumlah 8194, cukup banyak untuk proyek ini
- Melihat nilai unik setiap data
- Mengubah isi data kolom size menjadi tipe data float dengan satuan (MB) agar angkanya tidak terlampau besar
- Data size dengan Varies with device diganti dengan rata-rata
- Menyatukan data unrated dan Adults only kedalam Mature 17+. Hal ini didasarkan dengan asumsi 17 tahun itu cukup dewasa dan data Unrated bisa saja berisi konten dewasa (Mature).
- Mengubah data Android Ver agar jumlah versi android bisa lebih sedikit
- Berdasarkan Correlation Matrix yang terbuat, fitur numerik Review lebih berpengaruh daripada Size
- Mengubah data Installs dan Price menjadi angka (Integer dan Float) dengan menghilangkan tanda $ dan +

## Modeling
Disini kita akan memakai Content-Based filtering berdasarkan kategori. Hal ini dikarenakan rancunya data Review, Installs dan Rating. Tidak semua orang yang Install aplikasi mereview dan memberi rating pada aplikasi.

- Pertama, kita ubah kategori menjadi angka dengan TfidfVectorizer()
- Kedua,  kita menghitung cosine_similarity dari data Kolom Category.
- Ketiga, kita menggabungkan data Cosine dengan nama aplikasi menjadi satu dataframe
- Keempat, kita membuat fungsi yang menggunakan nama aplikasi yang ada di data, dan
## Evaluation
Model berhasil memberi rekomendasi dengan kategori Art and Design dan Books andd Reference

Dari kedua test dengan nama aplikasi "Coloring book moana" dan "The SCP Foundation DB fr nn5n", program dapat memberikan 20 aplikasi hasil rekomendasi dengan kategori yang sama. Presisi yang dihasilkan selama ini adalah 100%, karena 20 aplikasi yang diberikan memiliki kategori yang sama dengan aplikasi yang diinput. Hal ini dapat disebabkan jumlah datanya yang banyak sehingga mesin cukup mengetahui rekomendasi yang tepat.