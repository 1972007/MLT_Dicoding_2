# Laporan Proyek Machine Learning - Joseph Setiawan

## Project Overview
Banyak aplikasi yang terdaftar dalam Google Play Store. Proyek ini dibuat untuk menciptakan suatu sistem rekomendasi aplikasi yang sesuai dengan keinginan pengguna. Sistem rekomendasi aplikasi dapat membantu pengguna menemukan aplikasi mobile yang mereka butuhkan[1]. Dengan data yang diambil dari Google Play Store, proyek ini bertujuan membantu pengguna melalui sistem rekomendasi berdasarkan kategori.

## Business Understanding
Proyek ini berisi pembuatan rekomendasi aplikasi. Rekomendasi yang akan dipakai adalah rekomendasi berdasarkan kategori, karena sepertinya fitur data lain merupakan data detail aplikasi saja.

### Problem Statements
Masalah yang diselesaikan :
- Apa algoritma sistem rekomendasi yang diperlukan untuk memberikan rekomendasi berdasarkan kategori?

### Goals
Tujuan yang ingin dicapai :
- Penggunaan algoritma sistem rekomendasi untuk memberikan rekomendasi berdasarkan kategori

## Data Understanding
Data aplikasi yang digunakan diambil dari Kaggle. Data-data ini diammbil dengan teknik scrapping terhadap Google Play Store
[Google Play Store Apps](https://www.kaggle.com/datasets/lava18/google-play-store-apps/code?datasetId=49864&searchQuery=rec)
\
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
- Android Ver, yaitu versi android yang bisa menjalankan aplikasi<br>
![image](https://user-images.githubusercontent.com/56014054/181737259-2b2322e4-bcc6-49a5-9278-f1ed4b8c63d9.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181738636-c5529710-c63b-4b41-a790-27ce445e5ed7.png)

## Data Preparation
- Pengecekan deskripsi dan _missing value_ fail, barangkali ditemukan. Sebelum menghilangkan semua _missing value_, kita perlu melihat apakah memang perlu dihilangkan atau ada cara lain yang tidak banyak menghilangkan data<br>
![image](https://user-images.githubusercontent.com/56014054/181757449-a811aaa4-e006-4d3d-87d4-6fbd1a39d34a.png)<br>

![image](https://user-images.githubusercontent.com/56014054/181738683-d1eeb568-901b-4a93-8830-fac5a9952096.png)

- Kita menyaring rating yang tidak lebih dari 5. Ketika melihat graf, ada rating yang melebihi 5, yang merupakan data tidak wajar. Data Rating yang wajar tidak lebih besar dari 5. Maka itu data tersebut dihilangkan beserta data tanpa rating<br>
![image](https://user-images.githubusercontent.com/56014054/181738737-788782b1-f407-404a-b30e-26bb03b2bf15.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181738838-2f92cc28-52b3-4b14-bec4-55aedb822f16.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181738967-e7a04af2-0bc3-47b2-b11c-cbac7f29ed0c.png)

- Kita mengganti tipe data Review menjadi angka, karena asalnya data tersebut asalnya bertipe object<br>
![image](https://user-images.githubusercontent.com/56014054/181757605-28f83c60-8b12-4283-99cc-5fad6a2ff5cf.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181739149-4d36da21-cb32-4206-8da6-fe771bfa387f.png)

- Kita memonitor missing value. Kita menemukan Android Ver dan Current Version memilik nilai NaN. Namun, kita tidak perlu menghilangkan nilai NaN sekarang.<br>
![image](https://user-images.githubusercontent.com/56014054/181757894-14d9186c-c626-4306-a76e-2f2902020ac9.png)

- Kita melihat data kategori untuk mendapat gambaran data. Data Family adalah jenis aplikasi yang paling sering dijumpai, diikuti data kategori Game.\
![image](https://user-images.githubusercontent.com/56014054/181739407-35025bcb-3bf9-43f2-ad90-7a0285510c78.png)

- Kita menghilangkan kolom yang tidak digunakan untuk pembuatan sistem. Last Updated dan Current Ver bisa dipakai sebagai database saja. Genres sudah diwakilkkan Category secara garis besarnya, sehingga Genres dihilangkan untuk mengurangi dimensi<br>
```python
for i in ['Last Updated','Current Ver', "Genres"]:
  try:
    apps.drop(i,axis=1,inplace=True)
  except:
    continue
```

- Sekarang kita dapat menghilangkan data dengan missing value, karena dengan data yang tanpa missing value pun, data yang tersisa masih berjumlah 9364, cukup banyak untuk proyek ini\
![image](https://user-images.githubusercontent.com/56014054/181740203-b7e03ded-ab84-4c44-9836-c2935b092b8d.png)

- Kita melihat nilai unik data Size. Hanya bagian string paling belakang yang dilihat karena dalam data Size, nilai belakangnya pasti berbentuk M, k atau Varies with device (dengan string terakhir e)\
![image](https://user-images.githubusercontent.com/56014054/181740326-57cc6c1d-093b-41df-879d-ab6ab2a96939.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181740384-e96a44b2-fa30-40b2-aa61-d9b455505559.png)

- Kita mengubah isi data kolom size menjadi data angka dengan patokan satuan (MB) agar angkanya tidak terlampau besar.
```python
apps_sizeChange["Size"] = apps_sizeChange['Size'].map(lambda x: x[:-1] if "M" in x else x)
apps_sizeChange["Size"] = apps_sizeChange['Size'].map(lambda x: float(x[:-1])*0.001 if "k" in x else x)
```

- Data size dengan Varies with device diganti dengan rata-rata. Hasil modus dan median terlalu kecil dan opsi rata-rata lebih baik daripada kehilangan banyak data. Berikut kode pengambilan Mean:<br>
```python
app_size_number = apps_sizeChange.loc[apps_sizeChange['Size'].str[-1]!="e"]
app_size_number["Size"] = app_size_number["Size"].astype(float)

apps_sizeChange.loc[apps_sizeChange['Size']=="Varies with device", "Size"] = app_size_number["Size"].mean()
```

![image](https://user-images.githubusercontent.com/56014054/181740579-fc29907a-77b1-4eff-8ad8-51750fe306b4.png)

- Kita mengubah tipe data kolom Size menjadi float. Semua data Size sudah sesuai satuan MB, maka dari itu kita dapat mengubah tipe data Size menjadi float<br>
![image](https://user-images.githubusercontent.com/56014054/181759893-e8ad8dbc-559c-4668-a279-4dc23693eb20.png)

- Missing Value sudah tidak terdeteksi.<br>
![image](https://user-images.githubusercontent.com/56014054/181760178-bfad2815-db1b-4fa8-a669-6ca7729b5e36.png)

-Kita menyatukan data unrated dan Adults only ke dalam Mature 17+. Hal ini didasarkan dengan asumsi 17 tahun itu cukup dewasa dan aplikasi Unrated bisa saja berisi konten dewasa (Mature)\
![image](https://user-images.githubusercontent.com/56014054/181740665-459f4eed-91da-4819-b898-843bfbb1b499.png)

- Kita mengubah data Android Ver agar jumlah versi android bisa lebih sedikit. Angka 240 dipilih agar versi android yang tercatat lebih sedikit, dan data-data lain bisa dikategorikan "Others"\
![image](https://user-images.githubusercontent.com/56014054/181741126-770dff6e-8c38-499e-ad84-85fb3f23b4cf.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181741149-f783f11a-07bf-4c7d-a944-56eeaaa10e3a.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181741434-4323309b-1f23-44da-a0e9-acedf2e569e5.png)<br>

- Kita menghilangkan duplikat nama aplikasi untuk kemudahan indexing. Data masih berjumlah 8194 sehingga tidak apa-apa kita hilangkan data duplikatnya\
![image](https://user-images.githubusercontent.com/56014054/181741359-b2b27918-4c60-4488-b78b-d3d2f692cd27.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181741323-bdcf95c4-6975-42dc-917f-698df3e25c85.png)

- Berdasarkan Correlation Matrix yang terbuat, fitur numerik Review lebih berpengaruh daripada Size\
![image](https://user-images.githubusercontent.com/56014054/181741478-15a240c9-c571-4aac-a1d7-f51187ac1f38.png)

- Kita mengubah data Installs dan Price menjadi angka (Integer dan Float) dengan menghilangkan tanda $ dan +\
![image](https://user-images.githubusercontent.com/56014054/181741757-b38e9b42-59e7-450b-bd56-c5ffd8c0e795.png)

## Modeling
Disini kita akan memakai Content-Based filtering berdasarkan kategori. Hal ini dikarenakan rancunya data Review, Installs dan Rating. Tidak semua orang yang Install aplikasi mereview dan memberi rating pada aplikasi.

- Pertama, kita ubah kategori menjadi angka dengan TfidfVectorizer()
- Kedua,  kita menghitung cosine_similarity dari data hasil TfidfVectorizer()\
![image](https://user-images.githubusercontent.com/56014054/181741804-a99c1830-4e3b-4069-a99d-08c0ccf9dc2b.png)
- Ketiga, kita menggabungkan data Cosine dengan nama aplikasi menjadi satu dataframe\
![image](https://user-images.githubusercontent.com/56014054/181741917-c3584ffa-719b-4412-8125-b922452af7f3.png)

- Keempat, kita membuat fungsi yang menggunakan nama aplikasi yang ada di data\
![image](https://user-images.githubusercontent.com/56014054/181742066-c43b844c-94a5-4e2d-9998-a21e54ee761e.png)

Hasil :

![image](https://user-images.githubusercontent.com/56014054/181742186-3aef78c1-52e2-498f-a9c3-af0cd77b54ca.png)<br>
![image](https://user-images.githubusercontent.com/56014054/181742235-c6d8863c-5c56-4bd3-9667-4ff30fc4d3bf.png)


## Evaluation
Model dapat memberi rekomendasi dengan kategori Art and Design dan Books and Reference
![image](https://user-images.githubusercontent.com/56014054/181756446-622382a6-9805-44e4-9c40-7991bda3a9cb.png)

Dari kedua test dengan nama aplikasi "Coloring book moana" dan "The SCP Foundation DB fr nn5n", program dapat memberikan 20 aplikasi hasil rekomendasi dengan kategori yang sama. Presisi yang dihasilkan selama ini adalah 100%, karena 20 aplikasi yang diberikan memiliki kategori yang sama dengan aplikasi yang diinput. Namun, saat program perlu memberikan 40 rekomendasi aplikasi, presisi kategori "ART_AND_DESIGN tetap. Namun, presisi dalam hal kategori "BOOKS_AND_REFERENCE" menurun menjadi 0.5 (50%). Hal ini bisa saja disebabkan oleh tidak seimbangnya data, seperti kurangnya data aplikasi setiap kategori.

## Conclusion
Proyek pembuatan sistem rekomendasi aplikasi berhasil dibuat. Hanya saja permasalahan yang ditemukan adalah kurangnya variasi kategori data sehingga precision kategori bisa berbeda. 

## Reference
[1]. J. He, X. Fang, H. Liu, and X. Li, “Mobile app recommendation: An involvement-enhanced approach,” SSRN Electronic Journal, 2018. 
