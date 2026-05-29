# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari CHUNK_SIZE)*

> Jawab: CHUNK_SIZE berfungsi untuk menentukan seberapa besar potongan teks dari dokumen PDF yang akan dibuat. Jadi, ketika dokumen diupload, isinya akan dibagi menjadi beberapa bagian atau chunk supaya sistem RAG bisa mencari informasi dengan lebih mudah. Nah, kalau ukuran chunk-nya kecil, teks yang dihasilkan jadi lebih spesifik

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (100–200)** | *Potongan teks yang kecil bikin pencarian jadi lebih tepat karena fokusnya lebih tajam. Tapi, ada risikonya juga, yaitu konteks penting bisa terpotong dan akhirnya jawabannya jadi kurang lengkap.* |
| **Sedang (250–400)** ⭐ Recommended | *Ukuran ini biasanya direkomendasikan karena cukup pas, nggak terlalu pendek tapi juga nggak terlalu panjang. Jadi, keseimbangan antara presisi dan kelengkapan konteks tetap terjaga.* |
| **Besar (500–1000)** | *Potongan yang besar membawa lebih banyak konteks, tapi pencarian bisa jadi kurang spesifik. Sistem juga mungkin mengambil informasi yang nggak terlalu relevan karena satu potongan mencakup terlalu banyak topik.* |

**Pertanyaan Refleksi:**

* Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?
  > Jawab: Untuk dokumen yang saya coba, ukuran CHUNK_SIZE yang sedang sebenarnya sudah cukup tepat karena isinya nggak terlalu panjang. Dengan nilai sedang ini, jawaban tetap lengkap tanpa kebanyakan informasi yang kurang relevan.

* Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: Kalau jawaban terasa terpotong atau kurang nyambung, sebaiknya coba naikkan parameter CHUNK_SIZE atau FINAL_TOP_M supaya sistem bisa mengirim konteks yang lebih utuh ke LLM

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TOP_K_INITIAL)*

> Jawab: Jadi, TOP_K_INITIAL ini berperan untuk menentukan berapa banyak kandidat chunk awal yang diambil dari hasil pencarian vektor sebelum akhirnya dipilih lagi oleh reranker. Intinya, kalau nilainya semakin besar, maka jumlah kandidat yang diperiksa juga makin banyak.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (5–10)** | *Prosesnya memang jadi lebih cepat karena kandidat yang diambil nggak terlalu banyak. Tapi, ada risiko beberapa chunk penting malah nggak kebawa.* |
| **Sedang (15–25)** ⭐ Recommended | *Rentang ini biasanya direkomendasikan karena sudah cukup untuk menangkap kandidat yang relevan, sekaligus masih efisien dari segi waktu.* |
| **Besar (30–50)** | *Recall-nya meningkat karena sistem mengambil lebih banyak kandidat. Tapi, waktu prosesnya bisa jadi lebih lama, apalagi kalau reranker-nya aktif.* |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

* Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.
  > Jawab: Top-k yang kecil sebenarnya sudah cukup kalau dokumennya pendek dan pertanyaannya juga sederhana. Tapi, kalau dokumennya panjang atau pertanyaannya butuh konteks yang lebih detail, top-k kecil bisa saja melewatkan informasi penting.

* Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL sebenarnya sangat terkait dengan penggunaan USE_RERANKER. Jadi, kalau reranker aktif, TOP_K_INITIAL ini menentukan berapa banyak kandidat awal yang nanti akan dievaluasi ulang oleh reranker tersebut. Tapi kalau reranker nggak dipakai, pengaruh TOP_K_INITIAL jadi nggak terlalu signifikan karena sistem langsung mengambil chunk teratas saja.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari FINAL_TOP_M)*

> Jawab: Jadi, FINAL_TOP_M ini menentukan berapa banyak chunk terbaik yang akhirnya dikirim ke LLM sebagai konteks untuk menjawab pertanyaan. Parameter ini cukup penting karena memengaruhi seberapa banyak informasi yang dipakai oleh model saat menyusun jawabannya.  

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Sedikit (2–3)** | *Jawaban memang jadi lebih cepat dan fokus, tapi ada kemungkinan konteks penting nggak tersampaikan ke LLM.
* |
| **Sedang (4–6)** ⭐ Recommended | * Rentang ini biasanya direkomendasikan karena memberikan konteks yang cukup tanpa bikin input jadi terlalu panjang.
* |
| **Banyak (7–10)** | *Jawabannya memang bisa jadi lebih lengkap karena konteks yang diberikan juga lebih banyak. Tapi ya, prosesnya jadi lebih lambat, dan kadang LLM malah bisa bingung kalau konteksnya terlalu banyak atau campur aduk.* |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

* Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.
  > Jawab:  Sebenarnya, tidak selalu benar kalau semakin banyak konteks itu semakin bagus. Dari beberapa percobaan, justru terlalu banyak konteks bisa bikin jawaban jadi kurang fokus karena LLM harus memproses terlalu banyak informasi sekaligus.

* Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: kalau konteks yang diberikan saling bertentangan, LLM bisa menghasilkan jawaban yang nggak konsisten atau malah mencampur informasi dari beberapa bagian dokumen

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari USE_RERANKER)*

> Jawab: USE_RERANKER berfungsi untuk mengaktifkan atau mematikan model reranker. Reranker membantu mengurutkan ulang chunk berdasarkan relevansi terhadap pertanyaan, sehingga konteks yang dikirim ke LLM menjadi lebih tepat.

**Pengaruhnya:**

| Mode | Efek |
| --- | --- |
| `True` | *Jawaban biasanya jadi lebih relevan dan tepat karena chunk-nya dicek ulang oleh reranker. Tapi, ya, prosesnya memang agak sedikit lebih lambat karena ada model tambahan yang harus dijalankan.* |
| `False` | *Prosesnya memang lebih cepat karena cuma mengandalkan cosine similarity dari pencarian vektor. Tapi, chunk yang dipilih kadang kurang relevan, apalagi kalau pertanyaannya ambigu atau dokumennya panjang.* |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

* Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.
  > Jawab: Reranker ini paling terasa manfaatnya kalau query-nya memang spesifik atau agak ambigu. Misalnya, saat saya bertanya tentang bagian tertentu dalam sebuah dokumen, reranker bakal membantu memilih chunk yang lebih tepat dan relevan dengan pertanyaan tersebut.

* Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: Saat reranker diaktifkan, memang terasa ada tambahan latency yang sedikit lebih lama dibandingkan saat dimatikan. Tapi, peningkatan relevansi jawaban yang didapat cukup membantu, jadi tetap layak dipakai terutama untuk dokumen yang butuh tingkat akurasi lebih tinggi.

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TEMPERATURE)*

> Jawab: TEMPERATURE mengatur tingkat kreativitas jawaban LLM. Nilai rendah membuat jawaban lebih konsisten dan faktual, sedangkan nilai tinggi membuat jawaban lebih bervariasi tetapi berisiko keluar dari konteks dokumen.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **0.0–0.2** ⭐ Recommended untuk RAG | *Jawaban cenderung lebih stabil dan konsisten, serta benar-benar mengikuti isi dokumen. Rentang ini cocok banget untuk RAG karena jawaban memang harus berdasarkan konteks yang ada.* |
| **0.3–0.5** | *Di rentang ini, jawaban mulai agak fleksibel, tapi masih cukup aman kalau untuk pertanyaan yang berlandaskan dokumen.* |
| **0.6–0.8** | *Jawaban jadi lebih kreatif dan beragam, tapi di rentang ini ada risiko muncul informasi yang nggak ada di dokumen asli.* |
| **0.9–1.0** | *Jawaban cenderung sangat acak dan rawan halusinasi, jadi nilai ini kurang cocok kalau dipakai untuk QA berbasis dokumen.* |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap *grounded* pada konteks yang diberikan.

**Pertanyaan Refleksi:**

* Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.
  > Jawab: Untuk QA yang berdasarkan dokumen perusahaan, sebaiknya temperature-nya sekitar 0.1 sampai 0.3 saja, karena kita butuh jawaban yang akurat, konsisten, dan pastinya sesuai dengan isi dokumen.

* Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: Kalau temperature-nya dinaikkan sampai 0.9, jawabannya memang jadi lebih panjang dan kreatif, tapi risikonya model bisa saja menambahkan informasi yang sebenarnya nggak ada dalam dokumen tersebut.

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model | Ukuran | Pengaruh terhadap Kecepatan & Kualitas |
| --- | --- | --- |
| `llama-3.1-8b-instant` (Default) | 8B parameter | *Cukup cepat dan pas banget buat pengembangan atau coba-coba awal. Kualitas jawabannya sudah lumayan oke untuk dokumen yang sederhana, tapi kalau pertanyaannya agak rumit, kadang detailnya kurang.* |
| `openai/gpt-oss-120b` | 120B parameter | *Model yang jauh lebih besar, jadi kemampuannya dalam memberikan jawaban yang lebih kuat dan detail juga lebih tinggi. Tapi ya, prosesnya biasanya sedikit lebih lama dibanding model yang kecil.* |
| `llama-3.3-70b-versatile` | 70B parameter | *Model ini cukup seimbang untuk kebutuhan yang mengutamakan kualitas tinggi. Jawaban yang diberikan biasanya lebih baik dibanding model 8B, tapi tentu saja ukurannya lebih besar dan prosesnya agak lebih lambat.* |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

* Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.
  > Jawab: Peningkatan kualitas dari model besar memang terasa terutama saat menghadapi pertanyaan yang butuh pemahaman lebih mendalam. Tapi, untuk dokumen yang pendek dan pertanyaan yang sederhana, model kecil sebenarnya sudah cukup, jadi waktu ekstra yang dibutuhkan model besar tidak selalu sebanding.

* Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: Untuk dokumen saya, model yang paling pas sebenarnya adalah llama-3.1-8b-instant karena dokumennya cukup singkat dan pertanyaannya juga sederhana. Model ini cukup cepat dan hasil jawabannya sudah cukup relevan

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter | Nilai Default | Fungsi |
| --- | --- | --- |
| `CHUNK_OVERLAP` | 60 | *Jadi, CHUNK_OVERLAP ini berfungsi untuk memberikan irisan kata antar chunk supaya informasi penting nggak hilang saat teks dipotong-potong. Kalau overlap-nya terlalu kecil, bisa-bisa konteks antar chunk jadi terputus. Tapi, kalau kebesaran, malah banyak teks yang terulang-ulang, jadi prosesnya jadi kurang efisien juga.* |
| `SCORE_THRESHOLD` | 0.30 | *Jadi, SCORE_THRESHOLD ini berfungsi sebagai batas minimum untuk skor kemiripan antara chunk dengan pertanyaan. Kalau threshold-nya terlalu tinggi, chunk yang sebenarnya relevan malah bisa terlewat. Sebaliknya, kalau terlalu rendah, chunk yang kurang relevan justru ikut terbawa ke dalam konteks.* |

---

## 📝 Tips Eksperimen

* Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
* Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
* Catat kapan sistem salah & mengapa
* Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
* Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

* Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
* Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
