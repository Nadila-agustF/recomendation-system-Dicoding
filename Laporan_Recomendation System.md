# Laporan Proyek Sistem Rekomendasi Film - Nadila Agustiani Farhan

## Project Overview

![alt text](img/image.png)

Industri film terus saat ini terus berkembang pesat, ditandai dengan jumlah produksi film yang semakin banyak dari tahun ke tahun. Berbagai genre film diproduksi untuk menjangkau minat penonton yang sangat beragam. Berdasarkan hasil penelitian Muhammad Fajriansyah, 2021 [^1], pada tahun 2018 jumlah penonton bioskop di Indonesia telah mencapai lebih dari 50 juta orang, dengan jumlah produksi film — baik film dalam negeri maupun luar negeri — mencapai lebih dari 200 judul yang telah tayang di seluruh Indonesia.
Seiring berkembangnya teknologi digital, kebiasaan menonton film juga telah berubah. Kini, menonton film tidak lagi terbatas pada bioskop. Penonton dapat menikmati berbagai judul film di mana saja dan kapan saja melalui platform streaming digital [^2]. Platform ini menawarkan ribuan pilihan judul film dari berbagai genre dan negara.
Namun, banyaknya pilihan film seringkali membuat penonton kesulitan dalam menentukan film yang sesuai dengan preferensi mereka. Oleh karena itu, sistem rekomendasi menjadi solusi penting untuk membantu pengguna dalam memilih film yang relevan. Dengan adanya sistem ini, penonton dapat menghemat waktu dalam memilih film, sehingga pengalaman menonton menjadi lebih menyenangkan. Dengan demikian, dalam proyek ini, dikembangkan sistem rekomendasi berbasis **Content-Based Filtering** yang memanfaatkan informasi konten film seperti genre dan sutradara untuk menghasilkan rekomendasi yang relevan. Metode ini terbukti efektif dalam beberapa penelitian sebelumnya, termasuk dalam sistem rekomendasi musik oleh Putra dan Santika (2020)[^2].

## Business Understanding

### Problem Statements

1. Bagaimana membantu pengguna menemukan film yang sesuai dengan preferensi mereka?
2. Bagaimana merancang sistem rekomendasi tanpa data interaksi pengguna (seperti rating atau ulasan)?

### Goals

1. Membangun sistem yang dapat memberikan rekomendasi film secara otomatis berdasarkan konten.
2. Menyajikan film serupa berdasarkan fitur seperti genre dan sutradara, tanpa bergantung pada data eksplisit pengguna.

### Solution statements

- Untuk menjawab tantangan tersebut, solusi yang dapat dilakukan adalah membangun sistem rekomendasi film menggunakan pendekatan **Conten-Based-Filtering**. Metode ini mengukur kesamaan antar film menggunakan **cosine similarity** terhadap fitur seperti genre dan sutradara. Sistem ini cocok untuk kondisi cold-start, yaitu ketika tidak tersedia data interaksi pengguna. Dengan demikian, sistem tetap mampu memberikan rekomendasi yang relevan dan personal meskipun tanpa data historis pengguna, sehingga tetap mendukung pengalaman pengguna yang optimal.

## Data Understanding

Dataset yang digunakan dalam proyek ini berisi informasi mengenai judul film, genre, sutradara, dan beberapa data pendukung lainnya. Dataset tersebut diperoleh dari platform Kaggle. 
Sumber dataset: [Dataset](https://www.kaggle.com/datasets/arthurchongg/imdb-top-1000-movies) - Kaggle

Variabel-variabel pada IMDB-Movie dataset adalah sebagai berikut:

- Title : Judul film
- Tirector : Nama sutradara yang mengarahkan film. 
- Release_year : Tahun rilis film.
- Runtime : Durasi film, disimpan dalam bentuk teks seperti "142 min".
- Genre : Genre film, bisa terdiri dari satu atau lebih genre yang dipisahkan koma.
- Rating : Rating film berdasarkan penilaian pengguna (skala 1–10).
- Metascore : Skor film dari kritikus (skala 0–100).
- Gross : Pendapatan kotor film, dalam format string (contoh: "$28.34M").

### Melihat informasi data

```sh
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1000 entries, 0 to 999
Data columns (total 8 columns):
 #   Column        Non-Null Count  Dtype
---  ------        --------------  -----
 0   title         1000 non-null   object
 1   director      1000 non-null   object
 2   release_year  1000 non-null   object
 3   runtime       1000 non-null   object
 4   genre         1000 non-null   object
 5   rating        1000 non-null   float64
 6   metascore     1000 non-null   int64
 7   gross         1000 non-null   object
dtypes: float64(1), int64(1), object(6)
memory usage: 62.6+ KB
```

> Note:
> Dataset yang digunakan dalam proyek ini terdiri dari 1000 baris data dan 8 kolom fitur yang masing-masing merepresentasikan informasi terkait film, seperti judul, sutradara, genre, rating, hingga pendapatan kotor. > Namun, terdapat beberapa hal yang perlu dibersihkan dalam data, seperti kolom release_year yang masih bertipe objek. Kolom ini perlu diubah menjadi tipe numerik agar dapat dieksplorasi dan dianalisis dengan lebih optimal.
> 

### Memeriksa nilai 0
Ketika menampilkan dataframe, terdapat nilai 0 pada kolom metascore dan gross. 

![alt text](img/image-4.png)

Kemudian, ketika diperiksa nilai 0 di kolom metascore, ternyata terdapat nilai 0 yang cukup banyak. 
```sh # Menampilkan data yang bernilai 0
mv_df[mv_df['metascore'] == 0]
```
Hasil analisis

![alt text](img/image-3.png)

### Memeriksa missing value dan duplikasi
Untuk memastikan bahwa data yang digunakan bersih dari missing value (data yang hilang) maupun duplikasi, dilakukan pemeriksaan menggunakan kode berikut:
```sh
print("Jumlah missing value:", mv_df.isna().sum().sum())
print("Jumlah duplikasi data:", mv_df.duplicated().sum())
```
> Hasil pemeriksaan menunjukkan bahwa dataset tidak memiliki data yang hilang maupun duplikat:

```sh
Jumlah missing value: 0
Jumlah duplikasi data: 0 
```
> Dengan demikian, tidak diperlukan proses tambahan untuk penanganan data hilang atau penghapusan data ganda.
> 
### Melihat Statistik Deskriptif

Untuk melihat statistik deskriptif data film digunakan perintah berikut:
```sh
mv_df.describe()
```
Hasilnya akan menampilkan ringkasan statistik untuk kolom numerik dalam dataset, seperti berikut:
```sh
      rating	metascore
count	1000.00000	1000.000000
mean	7.96870	66.653000
std	0.27562	30.712829
min	7.60000	0.000000
25%	7.80000	64.750000
50%	7.90000	77.000000
75%	8.10000	86.000000
max	9.30000	100.000000
```

> Karena fungsi .describe() hanya menganalisis kolom dengan tipe data numerik, maka yang ditampilkan hanyalah kolom rating dan metascore yang bertipe float. Berdasarkan hasil tersebut:
> - Rating film memiliki nilai minimum sebesar 7.6 dan maksimum 9.3, dengan rata-rata sekitar 7.97.
> - Metascore berkisar antara 0 hingga 100, dengan rata-rata sekitar 66.65.
> Statistik ini memberikan gambaran umum mengenai persebaran nilai rating dan metascore dari film-film dalam dataset.

### Melihat jumlah nilai unique data

```sh
Jumlah unique value setiap kolom:
title 994
director 560
release_year 123
runtime 142
genre 195
rating 17
metascore 61
gross 709
dtype: int64
```

> Terdapat 994 judul film yang unik, dengan jumlah 560 sutradara menunjukkan bahwa sebagian besar film disutradarai oleh orang yang berbeda-beda. Selain itu, terdapat 195 kombinasi genre yang berbeda, yang mengindikasikan bahwa genre merupakan fitur yang cukup variatif dan dapat menjadi informasi penting dalam sistem rekomendasi berbasis konten.

### Melihat top 10 Sutradara Film

![alt text](img/image-1.png)
Dari total 560 sutradara yang terdapat dalam dataset, grafik di atas menampilkan 10 sutradara teratas yang paling banyak menyutradarai film dalam daftar. Hal ini memberikan gambaran siapa saja tokoh penting di balik sebagian besar film top yang ada dalam dataset.

### Melihat distribusi rating film

![alt text](img/image-2.png)

Grafik di atas menunjukkan distribusi rating film dalam dataset. Terlihat bahwa:
- Sebagian besar film memiliki rating antara 7.6 hingga 8.2, yang berarti mayoritas film mendapat penilaian cukup tinggi namun tidak ekstrem.
- Jumlah film paling banyak berada di sekitar rating 7.7–8.0, yang membentuk puncak distribusi (modus).
- Distribusi bersifat miring ke kanan (right-skewed), menandakan hanya sedikit film yang mendapatkan rating sangat tinggi di atas 8.5 hingga maksimal 9.3.

## Data Preparation

Dalam mempersiapkan data agar dapat menghasilkan sistem rekomendasi yang akurat, diperlukan beberapa tahapan data preparation, antara lain:

1. Pembersihan Data
Dataset sudah dipastikan tidak memiliki missing value atau data duplikat, sehingga tidak diperlukan proses imputasi atau penghapusan data ganda. Namun pada kolom **release_year** yang masih bertipe objek, terdapat string berupa tanda kurung dan string lainnya. Maka perlu menghapus string sebelum mengubah tipe data. 
```sh
# Menghapus string dan tanda baca pada kolom release_year
mv_df['release_year'] = mv_df['release_year'].str.replace(r'\D', '', regex=True)
```

2. Mengubah tipe data
Setelah dibersihkan baru ubah data ke numerik, berupa integer. `...mv_df['release_year'].astype(int)`` 
Tipe data berhasil diubah
``2   release_year  1000 non-null   int64``

3. Pemilihan fitur
Dalam pendekatan ini, fitur yang relevan adalah genre dan director karena kedua atribut ini mencerminkan gaya dan jenis film yang dapat mempengaruhi preferensi pengguna. Oleh karena itu, kolom metascore dan gross dapat dihapus karena kurang relevan selain itu mengandung nilai 0 yang cukup banyak. 

4. Feature Engineering
Menggabungkan kolom genre dan director ke dalam satu kolom **combined_features** untuk mempermudah proses ekstraksi fitur dan penghitungan kemiripan antar film. 

```sh
mv_df['combined_features'] = mv_df['director'] + ": " + mv_df['genre']
mv_df[['title', 'combined_features']].head()
```

Sehingga hasilnya seperti berikut: 

| title	| combined_features |
| ----- | ----------------- | 
| The Shawshank Redemption | Frank Darabont: Drama |
| The Godfather	| Francis Ford Coppola: Crime, Drama |
| The Dark Knight	| Christopher Nolan: Action, Crime, Drama |
| Schindler's List |	Steven Spielberg: Biography, Drama, History |
|	12 Angry Men |	Sidney Lumet: Crime, Drama |

## Modeling

Sistem rekomendasi ini menggunakan pendekatan Content-Based Filtering yang mengukur kemiripan antar film berdasarkan fitur kontennya (genre dan sutradara) menggunakan Cosine Similarity.
Langkah-langkah Model:

### TF-IDF Vectorization
Fitur yang sudah digabungkan akan diproses dengan teknik TF-IDF (Term Frequency - Inverse Document Frequency) untuk mengubah teks menjadi vektor numerik.
```sh
tfidf = TfidfVectorizer(stop_words='english')
tfidf_matrix = tfidf.fit_transform(mv_df['combined_features'])
```

### Menghitung Cosine Similarity
```sh
# Hitung cosine similarity antar film
cosine_sim = cosine_similarity(tfidf_matrix, tfidf_matrix)
```

### Membuat Fungsi Rekomendasi

Fungsi berikut digunakan untuk menghasilkan 10 film teratas yang paling mirip dengan film input berdasarkan nilai cosine similarity:

```sh
def get_recommendations(title, cosine_sim=cosine_sim):
    if title not in indices:
        return f"Judul '{title}' tidak ditemukan dalam dataset."
    
    idx = indices[title]
    sim_scores = list(enumerate(cosine_sim[idx]))
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    sim_scores = sim_scores[1:11]
```
- **Penjelasan Parameter dan Alur Fungsi:**
> - **title:** Judul film yang dijadikan referensi untuk mencari film-film serupa.
> - **cosine_sim**: Matriks cosine similarity yang telah dihitung sebelumnya.
> - **indices:** Mapping antara judul film dan index baris pada DataFrame.
> - Jika judul tidak ditemukan dalam indices, fungsi akan mengembalikan pesan error.
> - **sim_scores = sorted(..., reverse=True)**: Mengurutkan daftar film berdasarkan nilai kemiripan dari yang paling tinggi ke paling rendah.
> - **sim_scores[1:11]:** Mengambil 10 film teratas, dimulai dari index ke-1 karena index ke-0 adalah film itu sendiri.

- **Contoh hasil rekomendasi**
Dengan menggunakan judul film "The Dark Knight" sebagai input, maka sistem akan menghasilkan 10 rekomendasi film serupa beserta nilai metrik cosine similarity yang diurutkan dari yang tertinggi. 

```sh
Title	Similarity Score
0	Batman Begins	1.0000
1	The Dark Knight Rises	0.9039
2	Dunkirk	0.8627
3	Memento	0.7886
4	Inception	0.7756
5	Interstellar	0.7394
6	The Prestige	0.7265
7	Mission: Impossible - Fallout	0.4157
8	Vikram Vedha	0.2062
9	Key Largo	0.2033
```
### Kelebihan & Kekurangan menggunakan Conten Based Filtering

**Kelebihan:**
- Tidak memerlukan data interaksi pengguna sehingga cocok untuk menangani cold-start problem, seperti pengguna baru.
- Rekomendasi yang dihasikan cenderung relevan karena berdasarkan kemiripan konten film.

**Kekurangan:**
- Tidak mampu merekomendasikan film di luar preferensi yang telah diketahui, sehingga kurang mampu memperluas cakupan rekomendasi.
- Pemilihan dan perancangan fitur konten sangat krusial, karena secara langsung memengaruhi akurasi sistem rekomendasi.

## Evaluation

Untuk mengukur performa sistem rekomendasi berbasis Content-Based Filtering, digunakan fungsi **evaluate_recommendation_precision()**. Fungsi ini mengevaluasi seberapa relevan film-film yang direkomendasikan terhadap film asal (film yang dijadikan referensi), berdasarkan dua aspek utama, genre dan sutradara dengan menggunakan metrik Genre similarity, director match, precision, dan recal.

1. Kemiripan Genre (Genre Similarity)
- Parameter yang digunakan: *Intersection over Union (IoU)* antara genre film asal dan genre film yang direkomendasikan. Untuk mengukur seberapa banyak genre yang sama yang dimiliki film rekomendasi dibandingkan dengan genre keseluruhan dari kedua film.

```sh 
# Hitung kesamaan genre (intersection over union)
genre_intersection = len(original_genre.intersection(rec_genre))
genre_union = len(original_genre.union(rec_genre))
genre_similarity = genre_intersection / genre_union if genre_union > 0 else 0
```
> Semakin tinggi skor genre similarity (mendekati 1), maka semakin besar kesamaan genre antara film rekomendasi dan film input. 

2. Director Match 
Kemiripan sutradara dievaluasi secara kualitatif menggunakan perbandingan boolean. Untuk menilai apakah film yang direkomendasikan disutradarai oleh orang yang sama, yang biasanya mengindikasikan kesamaan gaya atau kualitas film. Apabila nama sutradara dari film input dan film rekomendasi sama, maka dianggap cocok (match).

```sh
# Memeriksa apakah sutradara sama
director_match = original_director == rec_director
```
> Nilai True berarti film direkomendasikan oleh sutradara yang sama dengan film asal, dan False jika hasilnya sebaliknya.

- Contoh evaluasi dengan judul film **The Godfather**
```sh
# Contoh evaluasi untuk "The Godfather"
example_title = "The Godfather"
recommended_movies = recommend(example_title)
```
Fungsi ini akan membantu mengukur seberapa efektif rekomendasi sistem dalam menyarankan film yang benar-benar serupa secara konten, bukan hanya berdasarkan judul atau popularitas semata.

- Maka hasil untuk rekomendasi dari film The Godfather dengan pendekatan genre similarity dan Director Match sebagai berikut:

```sh
--- Evaluasi Rekomendasi untuk 'The Godfather' ---

Film Rekomendasi: The Godfather Part II
  Genre Asli: Crime, Drama
  Genre Rekomendasi: Crime, Drama
  Director Asli: Francis Ford Coppola
  Director Rekomendasi: Francis Ford Coppola
  Genre Similarity (IoU): 1.0
  Director Match: True

Film Rekomendasi: The Conversation
  Genre Asli: Crime, Drama
  Genre Rekomendasi: Drama, Mystery, Thriller
  Director Asli: Francis Ford Coppola
  Director Rekomendasi: Francis Ford Coppola
  Genre Similarity (IoU): 0.0
  Director Match: True

Film Rekomendasi: Apocalypse Now
  Genre Asli: Crime, Drama
  Genre Rekomendasi: Drama, Mystery, War
  Director Asli: Francis Ford Coppola
  Director Rekomendasi: Francis Ford Coppola
  Genre Similarity (IoU): 0.0
  Director Match: True

Film Rekomendasi: The Grapes of Wrath
  Genre Asli: Crime, Drama
  Genre Rekomendasi: Drama
  Director Asli: Francis Ford Coppola
  Director Rekomendasi: John Ford
  Genre Similarity (IoU): 0.0
  Director Match: False

Film Rekomendasi: The Quiet Man
  Genre Asli: Crime, Drama
  Genre Rekomendasi: Comedy, Drama, Romance
  Director Asli: Francis Ford Coppola
  Director Rekomendasi: John Ford
  Genre Similarity (IoU): 0.25
  Director Match: False
```
3. Precision@K 
Precision@K digunakan untuk mengukur proporsi film relevan yang berhasil masuk ke dalam Top-K rekomendasi. Relevansi ditentukan berdasarkan kesamaan genre. Proses perhitungan Precision adalah sebagai berikut:

```sh
relevant_in_rec = 0          # untuk iInisialisasi jumlah rekomendasi relevan.
  for rec_title in recommended_titles.head(k):        # Akan mengambil Top-K film teratas dari rekomendasi.
    rec_movie_row = df[df['title'] == rec_title]
    if not rec_movie_row.empty:           # akan melanjutkan hanya jika film ditemukan di data.
        rec_genre = set(rec_movie_row.iloc[0]['genre'].split(','))        # Ambil dan ubah genre film jadi format set.
        # Item dianggap relevan jika ada irisan genre
        if len(original_genre.intersection(rec_genre)) > 0:         # Jika genre film input dan rekomendasi beririsan, hitung sebagai relevan.
            relevant_in_rec += 1
```
Kemudian menghitung precision
```sh
precision = relevant_in_rec / k if k > 0 else 0.0 
```
> Cara menghitung precision menggunakan rumus
> - Precision@K= Jumlah Rekomendasi yang Relevan ÷ K
> - Jika k = 0, maka precision diberi nilai 0.0 agar tidak terjadi pembagian dengan nol.

Contoh hasil evaluasi untuk film The Dark Knight
```sh
Precision@5: 0.8000
```
**Note:** Hal ini berarti dari 5 film yang direkomendasikan, 4 film diantaranya memiliki genre yang serupa dengan The Dark Knight. Artinya, sebagian besar rekomendasi cukup relevan.

4. Recall@K
Recall@K mengukur sejauh mana sistem berhasil merekomendasikan seluruh film relevan yang ada di dataset. Semakin banyak film relevan yang berhasil masuk ke daftar rekomendasi, semakin tinggi nilai recall.

```sh
# Total item relevan dalam dataset (semua film non-input yang memiliki irisan genre)
relevant_items = df[df['title'] != title].copy()
```
Mengambil semua film selain film input sebagai kandidat item relevan.

```sh
relevant_items['genre_match'] = relevant_items['genre'].apply(
  lambda g: len(original_genre.intersection(set(g.split(',')))) > 0
  )
```
> **Note** 
> - Tandai film yang genre-nya beririsan dengan film input → dianggap relevan.
> - Hasilnya: kolom genre_match berisi True jika relevan.

```sh 
total_relevant = relevant_items['genre_match'].sum()
```
> **Note:** Hitung jumlah total film relevan di seluruh dataset (di luar input).

```sh 
recall = relevant_in_rec / total_relevant if total_relevant > 0 else 0.0
```
> - Menghitung recal dengan rumus Recall@K = (jumlah rekomendasi relevan di Top-K) ÷ (jumlah total film relevan).
> - Jika tidak ada film relevan sama sekali, recall = 0.0.

Contoh hasil untuk The Dark Knight
`Recall@5: 0.0073`

Artinya, dari seluruh film yang memiliki genre serupa dengan The Dark Knight di dataset, hanya sekitar 0.73% yang berhasil muncul dalam 5 rekomendasi teratas. Nilai ini rendah, yang wajar karena jumlah total film relevan bisa sangat banyak, sementara Top-K hanya terbatas pada lima.

### Evaluasi Sistem Rekomendasi
Dalam sistem rekomendasi berbasis konten, evaluasi menggunakan pendekatan atribut, bukan perilaku pengguna, karena tidak tersedia data eksplisit seperti histori penonton atau rating personal. Oleh karena itu, metrik seperti precision dan recall digunakan untuk mengukur relevansi konten.

1. Evaluasi Kesamaan Konten
Sistem dievaluasi berdasarkan seberapa mirip genre dan sutradara film yang direkomendasikan dengan film input. Hasil menunjukkan bahwa sistem mampu memberikan rekomendasi yang relevan secara konten.

2. Cosine Similarity
Cosine similarity mengukur tingkat kemiripan antar film berdasarkan representasi vektornya. Nilainya antara 0 hingga 1. Semakin tinggi skor cosine similarity, semakin besar kesamaan kontennya.

## Kesimpulan

Sistem rekomendasi film berbasis **Content-Based Filtering** yang dikembangkan dalam proyek ini terbukti mampu memberikan rekomendasi yang relevan dan berkualitas tinggi berdasarkan kemiripan konten antar film, khususnya dalam aspek genre dan sutradara. Evaluasi terhadap contoh film seperti *The Godfather* menunjukkan bahwa film-film rekomendasi memiliki karakteristik konten yang serupa, baik dari segi genre maupun sutradaranya.

Meskipun sistem ini tidak menggunakan data pengguna atau rating eksplisit, hasil evaluasi internal menunjukkan bahwa model mampu memberikan rekomendasi yang konsisten dan relevan secara konten. Film yang direkomendasikan cenderung memiliki genre yang serupa atau disutradarai oleh orang yang sama, yang menjadi indikator presisi dari sistem berbasis konten ini.

Evaluasi dilakukan dengan menggunakan indikator seperti:
- **Director Match** – untuk memeriksa apakah film rekomendasi disutradarai oleh orang yang sama.
- **Genre Similarity (IoU)** – untuk mengukur seberapa besar irisan genre antara film yang direkomendasikan dan film input.
- **Precision@K:** Mengukur proporsi rekomendasi relevan dalam top-K rekomendasi. Misalnya, Precision@5 = 0.8 menunjukkan bahwa 4 dari 5 film yang direkomendasikan memiliki genre yang sesuai.
- **Recall@K:** Mengukur cakupan sistem dalam menemukan semua film relevan di dataset. Misalnya, Recall@5 = 0.0073 berarti hanya 0.73% dari semua film segenre yang berhasil direkomendasikan.

Nilai Precision@5 yang tinggi menunjukkan bahwa sistem cukup tepat sasaran dalam memilih film yang mirip, sedangkan nilai Recall@5 yang rendah merupakan hal yang umum pada pendekatan Content-Based Filtering, karena sistem hanya mengambil beberapa film paling mirip dari banyaknya data yang ada. Sistem ini sangat bermanfaat terutama dalam konteks awal implementasi atau jika data pengguna masih terbatas. Tentunya sistem ini dapat dikembangkan lebih lanjut dengan mempertimbangkan fitur tambahan atau pendekatan hybrid untuk cakupan yang lebih luas.

## Referensi
[^1] Muhammad Fajriansyah, et.all, Sistem Rekomendasi Film Menggunakan ContentBasedFiltering, jurnal ptiik.ub 2021 (https://j-ptiik.ub.ac.id/index.php/j-ptiik/article/view/9163/4159)
[^2] Jeremia Maheswara A.S, et al, Sistem Rekomendasi Film pada Platform Streaming Menggunakan Metode Content-Based Filtering, jurnals.usm, 2024 (https://journals.usm.ac.id/index.php/transformatika/)
