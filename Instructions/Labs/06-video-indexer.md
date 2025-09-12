---
lab:
  title: Analisis video
  description: Gunakan Azure AI Video Indexer untuk menganalisis video.
---

# Analisis video

Sebagian besar data yang dibuat dan digunakan saat ini dalam format video. **Azure AI Video Indexer** adalah layanan yang didukung AI yang dapat Anda gunakan untuk mengindeks video dan mengekstrak wawasan dari video tersebut.

> **Catatan**: Mulai 21 Juni 2022, kemampuan layanan Azure AI yang mengembalikan informasi pribadi dibatasi untuk pelanggan yang telah diberi [akses terbatas](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Tanpa mendapatkan persetujuan akses terbatas, kemampuan untuk mengenali orang dan selebritas dengan Video Indexer untuk lab ini tidak tersedia. Untuk detail selengkapnya tentang perubahan yang telah dilakukan Microsoft, dan mengapa - lihat [Investasi dan perlindungan AI yang bertanggung jawab untuk pengenalan wajah](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Mengunggah video ke Video Indexer

Pertama, Anda harus masuk ke portal Video Indexer dan mengunggah video.

1. Di browser Anda, buka [portal Video Indexer](https://www.videoindexer.ai) di `https://www.videoindexer.ai`.
1. Jika Anda sudah memiliki akun Video Indexer, lanjutkan untuk masuk. Jika belum, daftar untuk mendapatkan akun gratis dan masuk menggunakan akun Microsoft Anda (atau jenis akun valid lainnya). Jika Anda mengalami kesulitan masuk, coba buka sesi browser privat.

    > Catatan: Jika baru pertama kali masuk, Anda mungkin melihat formulir pop-up yang meminta Anda untuk memverifikasi bagaimana Anda akan menggunakan layanan. 

1. Di tab baru, unduh video AI yang Bertanggung Jawab dengan mengunjungi `https://aka.ms/responsible-ai-video`. Simpan file.
1. Di Video Indexer, pilih opsi **Unggah**. Kemudian, pilih opsi **Telusuri file**, pilih video yang diunduh, dan klik **Tambahkan**. Ubah teks di bidang **Nama file** menjadi **AI yang bertanggung jawab**. Pilih **Tinjau + unggah**, tinjau gambaran umum ringkasan, pilih kotak centang untuk memverifikasi kepatuhan terhadap kebijakan Microsoft untuk pengenalan wajah, dan unggah file.
1. Setelah file diunggah, tunggu beberapa menit sementara Video Indexer secara otomatis mengindeksnya.

> **Catatan**: Dalam latihan ini, kami menggunakan video ini untuk menjelajahi fungsionalitas Video Indexer; tetapi Anda harus meluangkan waktu untuk menontonnya hingga selesai setelah menyelesaikan latihan karena video ini berisi informasi dan panduan yang berguna untuk mengembangkan aplikasi berkemampuan AI secara bertanggung jawab! 

## Meninjau wawasan video

Proses pengindeksan mengekstrak wawasan dari video, yang dapat Anda lihat di portal.

1. Di portal Video Indexer, saat video diindeks, pilih untuk menampilkannya. Anda akan melihat pemutar video di samping panel yang menunjukkan wawasan yang diambil dari video.

    > **Catatan**: Karena kebijakan akses terbatas untuk melindungi identitas individu, Anda mungkin tidak melihat nama saat mengindeks video.

    ![Video Indexer dengan pemutar video dan panel Wawasan](../media/video-indexer-insights.png)

1. Saat video diputar, pilih tab **Garis Waktu** untuk melihat transkrip audio video.

    ![Video Indexer dengan pemutar video dan panel Garis Waktu yang menampilkan transkrip video.](../media/video-indexer-transcript.png)

1. Di kanan atas portal, pilih simbol **Tampilan** (yang terlihat mirip dengan &#128455;), dan di daftar wawasan, selain **Transkrip**, pilih **OCR** dan **Pembicara**.

    ![Menu tampilan Video Indexer dengan Transkrip, OCR, dan Pembicara dipilih](../media/video-indexer-view-menu.png)

1. Perhatikan bahwa panel **Garis Waktu** sekarang menyertakan:
    - Transkrip narasi audio.
    - Teks terlihat dalam video.
    - Indikasi pembicara yang muncul dalam video. Beberapa orang terkenal secara otomatis dikenali dengan nama, yang lain ditunjukkan dengan nomor (misalnya *Pembicara #1*).
1. Beralih kembali ke panel **Wawasan** dan lihat tampilan wawasan di sana. Meliputi:
    - Masing-masing orang yang muncul dalam video.
    - Topik yang dibahas dalam video.
    - Label untuk objek yang muncul di video.
    - Entitas bernama, seperti orang dan merek yang muncul dalam video.
    - Adegan kunci.
1. Dengan panel **Wawasan** terlihat, pilih simbol **Tampilan** lagi, dan dalam daftar wawasan, tambahkan **Kata Kunci** dan **Sentimen** ke panel.

    Wawasan yang ditemukan dapat membantu Anda menentukan tema utama dalam video. Misalnya, **topik** untuk video ini menunjukkan bahwa video ini jelas tentang teknologi, tanggung jawab sosial, dan etika.

## Cari wawasan

Anda dapat menggunakan Video Indexer untuk mencari video guna mendapatkan wawasan.

1. Di panel **Wawasan**, di kotak **Pencarian**, masukkan *Lebah*. Anda mungkin perlu menggulir ke bawah di panel Wawasan untuk melihat hasil semua jenis wawasan.
1. Amati bahwa satu *label* yang cocok ditemukan, dengan lokasinya dalam video yang ditunjukkan di bawahnya.
1. Pilih awal bagian tempat keberadaan lebah ditunjukkan, dan lihat video pada saat itu (Anda mungkin perlu menjeda video dan memilih dengan hati-hati - lebah hanya muncul sebentar!)

    ![Hasil pencarian Video Indexer untuk Lebah](../media/video-indexer-search.png)

1. Kosongkan kotak **Pencarian** guna menampilkan semua wawasan untuk video tersebut.

## Menggunakan REST API Video Indexer

Video Indexer menyediakan REST API yang dapat Anda gunakan untuk mengunggah dan mengelola video di akun Anda.

1. Di tab browser baru, buka [portal Azure](https://portal.azure.com) di `https://portal.azure.com` dan masuk menggunakan kredensial Azure Anda. Biarkan tab yang sudah ada dengan portal Video Indexer terbuka.
1. Di portal Azure, gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell ***dengan tidak ada penyimpanan pada langganan Anda.

    Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

    > **Catatan**: Jika portal meminta Anda untuk memilih penyimpanan untuk mempertahankan file Anda, pilih **Tidak ada akun penyimpanan yang diperlukan**, pilih langganan yang Anda gunakan dan tekan **Terapkan**.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Ubah ukuran panel cloud shell sehingga Anda dapat melihat lebih banyak.

    > **Tips**" Anda bisa mengubah ukuran panel dengan menyeret batas atas. Anda juga dapat menggunakan tombol minimalkan dan maksimalkan untuk beralih antara shell cloud dan antarmuka portal utama.

1. Di panel cloud shell, masukkan perintah berikut untuk mengkloning repo GitHub yang berisi file kode untuk latihan ini (ketik perintah, atau salin ke clipboard lalu klik kanan di baris perintah dan tempel sebagai teks biasa):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, buka folder yang berisi file kode aplikasi untuk latihan ini:  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### Mendapatkan detail API Anda

Untuk menggunakan API Video Indexer, Anda memerlukan beberapa informasi untuk mengautentikasi permintaan:

1. Di portal Video Indexer, luaskan panel kiri dan pilih halaman **Pengaturan akun**.
1. Catat **ID Akun** di halaman ini - Anda akan membutuhkannya nanti.
1. Buka tab browser baru dan buka [portal pengembang Video Indexer](https://api-portal.videoindexer.ai) di https://api-portal.videoindexer.ai, masuk menggunakan kredensial Azure Anda.
1. Pada halaman **Profil**, lihat **Langganan** yang terkait dengan profil Anda.
1. Pada halaman dengan langganan Anda, perhatikan bahwa Anda telah diberi dua kunci (primer dan sekunder) untuk setiap langganan. Kemudian pilih **Tampilkan** salah satu tombol untuk melihatnya. Anda akan membutuhkan kunci ini segera.

### Menggunakan REST API

Sekarang setelah Anda memiliki ID akun dan kunci API, Anda dapat menggunakan REST API untuk bekerja dengan video di akun Anda. Dalam prosedur ini, Anda akan menggunakan skrip PowerShell untuk melakukan panggilan REST; tetapi prinsip yang sama berlaku dengan utilitas HTTP seperti cURL atau Postman, atau bahasa pemrograman apa pun yang mampu mengirim dan menerima JSON melalui HTTP.

Semua interaksi dengan REST API Video Indexer mengikuti pola yang sama:

- Permintaan awal ke metode **AccessToken** dengan kunci API di header digunakan untuk mendapatkan token akses.
- Permintaan berikutnya menggunakan token akses untuk mengautentikasi saat memanggil metode REST agar berfungsi dengan video.

1. Di Cloud Shell, gunakan perintah berikut untuk membuka skrip PowerShell:

    ```
   code get-videos.ps1
    ```
    
1. Dalam skrip PowerShell, ganti tempat penampung **YOUR_ACCOUNT_ID** dan **YOUR_API_KEY** dengan ID akun dan nilai kunci API yang Anda identifikasi sebelumnya.
1. Perhatikan bahwa *lokasi* untuk akun gratis adalah "percobaan". Jika Anda sudah membuat akun Video Indexer yang tidak dibatasi (dengan sumber daya Azure terkait), Anda dapat mengubahnya ke lokasi tempat sumber daya Azure Anda disediakan (misalnya "eastus").
1. Tinjau kode dalam skrip, perhatikan bahwa memanggil dua metode REST: satu untuk mendapatkan token akses, dan satu lagi untuk membuat daftar video di akun Anda.
1. Simpan perubahan Anda (tekan *CTRL+S*), tutup editor kode (tekan *CTRL+Q*) lalu jalankan perintah berikut untuk menjalankan skrip:

    ```
   ./get-videos.ps1
    ```
    
1. Lihat respons JSON dari layanan REST, yang seharusnya berisi detail video **AI yang Bertanggung Jawab** yang Anda indeks sebelumnya.

## Menggunakan widget Video Indexer

Portal Video Indexer adalah antarmuka yang berguna untuk mengelola proyek pengindeksan video. Namun, ada kalanya Anda ingin berbagi video dan wawasannya dengan orang lain yang tidak memiliki akses ke akun Video Indexer Anda. Video Indexer menyediakan widget yang dapat Anda sematkan di halaman web untuk tujuan ini.

1. Gunakan perintah `ls` untuk melihat konten dari folder **video-indexer**. Perhatikan bahwa file tersebut berisi file **analyze-video.html**. Ini adalah halaman HTML dasar tempat Anda akan menambahkan widget **Pemutar** dan **Wawasan** Video Indexer.
1. Masukkan perintah berikut untuk mengedit file:

    ```
   code analyze-video.html
    ```

    File dibuka dalam editor kode.
   
1. Perhatikan referensi ke skrip **vb.widgets.mediator.js** di header - skrip ini memungkinkan beberapa widget Video Indexer di halaman untuk berinteraksi satu sama lain.
1. Di portal Video Indexer, kembali ke halaman **File media** dan buka video **AI yang Bertanggung Jawab**.
1. Di bagian pemutar video, pilih **&lt;/&gt; Sematkan** guna melihat kode iframe HTML untuk menyematkan widget.
1. Di kotak dialog **Bagikan dan Sematkan**, pilih widget **Pemutar**, atur ukuran video ke 560 x 315, lalu salin kode semat ke clipboard.
1. Di portal Azure Cloud Shell, di editor kode untuk file **analyze-video.html**, tempel kode yang disalin di bagian komentar **&lt;-- Widget pemutar di sini -- &gt;**.
1. Kembali ke portal Video Indexer, di kotak dialog **Bagikan dan Sematkan**, pilih widget **Insight**, lalu salin kode semat ke clipboard. Kemudian tutup kotak dialog **Bagikan dan Sematkan**, alihkan kembali ke portal Azure, dan tempel kode yang disalin di bagian komentar **&lt;-- Widget Insight di sini -- &gt;**.
1. Setelah mengedit file, dalam editor kode, simpan perubahan (*CTRL+S*) dan tutup editor kode (*CTRL+Q*) untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.
1. Di bar alat cloud shell, masukkan perintah berikut (Khusus cloud Shell) untuk mengunduh file HTML yang Anda edit:

    ```
    download analyze-video.html
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser, yang dapat Anda pilih untuk mengunduh dan membuka file. Tampilannya seperti berikut ini:

    ![Widget Video Indexer di halaman web](../media/video-indexer-widgets.png)

1. Bereksperimenlah dengan widget, menggunakan widget **Wawasan** untuk mencari wawasan dan membukanya di video.

