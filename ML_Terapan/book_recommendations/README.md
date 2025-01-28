# Laporan Proyek Machine Learning - Wahid Hasim Santoso

## Project Overview

Sistem rekomendasi telah menjadi komponen penting dalam membantu pengguna menemukan informasi atau
produk yang relevan di era digital. Dalam proyek ini, kami membangun sebuah sistem rekomendasi buku
menggunakan pendekatan collaborative filtering berbasis item (item-based collaborative filtering).

Sistem ini bertujuan untuk memberikan rekomendasi buku yang relevan berdasarkan pola rating pengguna
aktif.

### Pentingnya Proyek:

- Mempermudah pengguna menemukan buku yang sesuai dengan minat mereka di antara ribuan pilihan.
- Mengoptimalkan pengalaman pengguna dengan memberikan rekomendasi personal.
- Menyediakan solusi atas tantangan kelebihan informasi (information overload) di platform buku
  digital.

## Business Understanding

### Problem Statements

1. Bagaimana cara menyaring informasi dari dataset yang besar agar hanya mencakup pengguna aktif dan
   buku populer?
2. Bagaimana memberikan rekomendasi buku yang relevan berdasarkan pola rating pengguna aktif?
3. Bagaimana mengintegrasikan informasi tambahan seperti nama penulis dan gambar buku untuk
   meningkatkan pengalaman pengguna?

### Goals

1. Menyaring dataset untuk mencakup hanya pengguna aktif (memberi rating lebih dari 200 buku) dan
   buku populer (memiliki lebih dari 50 rating).
2. Membangun sistem rekomendasi buku menggunakan cosine similarity untuk mengukur kemiripan antar
   buku.
3. Menyediakan rekomendasi buku yang dilengkapi dengan informasi tambahan, seperti nama penulis dan
   gambar sampul.

### Solution Statements

1. **Collaborative Filtering**

   - Menggunakan item-based collaborative filtering untuk menghitung kesamaan antar buku berdasarkan
     pola rating pengguna.

2. **Content Enrichment**
   - Mengintegrasikan informasi tambahan, seperti gambar buku dan nama penulis, untuk meningkatkan
     daya tarik rekomendasi.

## Data Understanding

Dataset yang digunakan terdapat 3 data csv yang terdiri dari users.csv, ratings.csv, dan books.csv
masing masing berisi mengnai informasi tentang buku, pengguna, dan rating yang diberikan oleh
pengguna.

Berikut detail dari dataset yang di gunakan:

1. **Books.csv** Merupakan data buku diidentifikasi dengan ISBN masing-masing. ISBN yang tidak valid
   telah dihapus dari kumpulan data. Selain itu, beberapa informasi berbasis konten diberikan
   (Judul-Buku, Pengarang-Buku, Tahun-Terbit, Penerbit), yang diperoleh dari Amazon Web Services.
   Perhatikan bahwa jika ada beberapa pengarang, hanya pengarang pertama yang disediakan. URL yang
   menautkan ke gambar sampul juga diberikan, muncul dalam tiga rasa yang berbeda (Image-URL-S,
   Image-URL-M, Image-URL-L), yaitu kecil, sedang, besar. URL ini mengarah ke situs web Amazon.

2. **Users.csv** Merupakan data pengguna. Perhatikan bahwa ID pengguna (User-ID) telah dianonimkan
   dan dipetakan ke bilangan bulat. Data demografis disediakan (Lokasi, Usia) jika tersedia. Jika
   tidak, bidang ini berisi nilai NULL.

3. **Ratings.csv** Merupakan data rating yang berisi informasi peringkat buku. Peringkat
   (Book-Rating) dapat berupa eksplisit, dinyatakan dalam skala 1-10 (nilai yang lebih tinggi
   menunjukkan apresiasi yang lebih tinggi), atau implisit, dinyatakan dengan angka 0.

### Sumber Data

Dataset diperoleh dari
[Kaggle Book Recommendation Dataset](https://www.kaggle.com/datasets/arashnic/book-recommendation-dataset).

## Data Preparation

### Langkah-Langkah:

1. **Filter Pengguna Aktif**: Memilih pengguna yang telah memberi rating lebih dari 200 buku:

   ```python
   active_users = books_with_ratings.groupby('User-ID').count()['Book-Rating'] > 200
   active_users = active_users[active_users].index
   filtered_rating = books_with_ratings[books_with_ratings['User-ID'].isin(active_users)]
   ```

2. **Filter Buku Populer**: Memilih buku yang memiliki lebih dari 50 rating:

   ```python
   famous_books = filtered_rating.groupby('Book-Title').count()['Book-Rating'] >= 50
   famous_books = famous_books[famous_books].index
   final_ratings = filtered_rating[filtered_rating['Book-Title'].isin(famous_books)]
   ```

3. **Pivot Table**: Membuat tabel pivot untuk kemiripan antar buku:
   ```python
   pt = final_ratings.pivot_table(index='Book-Title', columns='User-ID', values='Book-Rating').fillna(0)
   ```

## Modeling

### Pendekatan: Item-Based Collaborative Filtering

1. **Cosine Similarity**: Menghitung kesamaan antar buku berdasarkan tabel pivot:

   ```python
   from sklearn.metrics.pairwise import cosine_similarity
   similarity_scores = cosine_similarity(pt)
   ```

2. **Fungsi Rekomendasi**:

   ```python
   def recommend(book_name):
       index = np.where(pt.index == book_name)[0][0]
       similar_items = sorted(list(enumerate(similarity_scores[index])), key=lambda x: x[1], reverse=True)[1:5]

       data = []
       for i in similar_items:
           item = []
           temp_df = books_df[books_df['Book-Title'] == pt.index[i[0]]]
           item.extend(list(temp_df.drop_duplicates('Book-Title')['Book-Title'].values))
           item.extend(list(temp_df.drop_duplicates('Book-Title')['Book-Author'].values))
           item.extend(list(temp_df.drop_duplicates('Book-Title')['Image-URL-M'].values))
           data.append(item)
       return data
   ```

3. **Output Rekomendasi**: Rekomendasi buku akan menampilkan:
   - Judul buku
   - Nama penulis
   - Gambar sampul buku

### Contoh Implementasi:

```python
recommend('1984')
```

## Evaluation

### Metrik Evaluasi

- **Cosine Similarity**: Menilai tingkat kemiripan antara buku yang direkomendasikan dengan buku
  yang dipilih.
- **Precision@K**: Mengukur relevansi rekomendasi terhadap buku yang diinginkan pengguna.

### Hasil Evaluasi

- Sistem mampu memberikan rekomendasi dengan kemiripan tinggi berdasarkan pola rating.
- Relevansi rekomendasi meningkat dengan mempertimbangkan pengguna aktif dan buku populer.

---

Laporan ini mencakup semua tahapan, mulai dari pemahaman bisnis hingga evaluasi model. Dengan sistem
rekomendasi ini, pengguna dapat menikmati pengalaman yang lebih personal dan efisien dalam menemukan
buku yang mereka sukai.
