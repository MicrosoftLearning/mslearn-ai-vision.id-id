---
lab:
  title: Mengklasifikasikan Gambar dengan Azure AI Custom Vision
---

# Mengklasifikasikan Gambar dengan Azure AI Custom Vision

Layanan **Azure AI Custom Vision** memungkinkan Anda membuat model penglihatan komputer yang dilatih berdasarkan gambar Anda sendiri. Anda dapat menggunakannya untuk melatih model *klasifikasi gambar* dan *deteksi objek*; yang kemudian dapat Anda publikasikan dan konsumsi dari aplikasi.

Dalam latihan ini, Anda akan menggunakan layanan Custom Vision untuk melatih model klasifikasi gambar yang dapat mengidentifikasi tiga kelas buah (apel, pisang, dan jeruk).

## Buat sumber daya Custom Vision

Sebelum dapat melatih model, Anda memerlukan sumber daya Azure untuk *pelatihan* dan *prediksi*. Anda dapat membuat sumber daya **Custom Vision** untuk setiap tugas ini, atau Anda dapat membuat satu sumber daya **Layanan Azure AI** dan menggunakannya untuk salah satu (atau keduanya).

Dalam latihan ini, Anda akan membuat sumber daya **Penglihatan Khusus** untuk pelatihan dan prediksi sehingga Anda dapat mengelola akses dan biaya untuk beban kerja ini secara terpisah.

1. Di tab browser baru, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Pilih tombol **&#65291;Buat sumber daya**, cari *custom vision*, dan buat sumber daya **Custom Vision** dengan pengaturan berikut:
    - **Buat opsi**: Keduanya
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga pelatihan**: F0
    - **Tingkat harga prediksi**: F0

    > **Catatan**: Jika Anda sudah memiliki layanan custom vision F0 di langganan, pilih **S0** untuk yang satu ini.

1. Tunggu hingga sumber daya dibuat, lalu lihat detail penyebaran dan perhatikan bahwa dua sumber daya Custom Vision telah disediakan; satu untuk pelatihan, dan satu lagi untuk prediksi (dibuktikan dengan akhiran **-Prediksi**). Anda dapat melihat ini dengan menavigasi ke grup sumber daya tempat Anda membuatnya.

> **Penting**: Setiap sumber daya memiliki *titik akhir* dan *kunci* sendiri, yang digunakan untuk mengelola akses dari kode Anda. Untuk melatih model klasifikasi gambar, kode Anda harus menggunakan sumber daya *pelatihan* (dengan titik akhir dan kuncinya); dan untuk menggunakan model terlatih untuk memprediksi kelas gambar, kode Anda harus menggunakan sumber daya *prediksi* (dengan titik akhir dan kuncinya).

## Mengkloning repositori untuk kursus ini

Anda akan mengembangkan kode menggunakan Cloud Shell dari portal Azure. File kode untuk aplikasi Anda telah disediakan dalam repositori GitHub.

> **Tips**: Jika Anda sudah mengkloning repositori **mslearn-ai-vision** baru-baru ini, Anda dapat melewati tugas ini. Jika belum melakukannya, ikuti langkah-langkah berikut untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi:  

    ```
   cd mslearn-ai-vision/Labfiles/07-custom-vision-image-classification
    ```
    
## Membuat proyek Visual Kustom

Untuk melatih model klasifikasi gambar, Anda perlu membuat proyek Visi Kustom berdasarkan sumber daya pelatihan Anda. Untuk melakukannya, Anda akan menggunakan portal Custom Vision.

1. Di tab browser baru, unduh gambar pelatihan dari `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/07-custom-vision-image-classification/training-images.zip` dan ekstrak folder zip untuk melihat kontennya. Folder ini berisi sub-folder gambar apel, pisang, dan jeruk.
1. Di tab browser baru, buka portal Visi Khusus di `https://customvision.ai`. Jika diminta, masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda dan setujui ketentuan layanan.
1. Di portal Custom Vision, buat proyek baru dengan pengaturan berikut:
    - **Nama**: Klasifikasi Buah
    - **Deskripsi**: Klasifikasi gambar untuk buah
    - **Sumber daya**: *Sumber daya Custom Vision yang Anda buat sebelumnya*
    - **Jenis Proyek**: Klasifikasi
    - **Jenis Klasifikasi**: Multikelas (satu tag per gambar)
    - **Domain**: Makanan
1. Dalam proyek baru, klik **\[+\] Tambahkan gambar**, dan pilih semua file di folder **training-images/apple** yang Anda lihat sebelumnya. Kemudian unggah file gambar, tentukan tag *apel*, seperti ini:

![Unggah apel dengan tag apel](../media/upload_apples.jpg)
   
1. Ulangi langkah sebelumnya untuk mengunggah gambar di folder **pisang** dengan tag *pisang*, dan gambar di folder **jeruk** dengan tag *jeruk*.
1. Jelajahi gambar yang telah Anda unggah di proyek Custom Vision - harus ada 15 gambar dari setiap kelas, seperti ini:

![Gambar buah yang diberi tag - 15 apel, 15 pisang, dan 15 jeruk](../media/fruit.jpg)
    
1. Dalam proyek Custom Vision, di atas gambar, klik **Latih** untuk melatih model klasifikasi menggunakan gambar yang diberi tag. Pilih opsi **Pelatihan Cepat**, lalu tunggu hingga perulangan pelatihan selesai (hal ini mungkin memerlukan waktu sekitar satu menit).
1. Ketika perulangan model telah dilatih, tinjau metrik performa *Presisi*, *Pengenalan*, dan *AP* - hal ini mengukur akurasi prediksi model klasifikasi, dan semua harus tinggi.

> **Catatan**: Metrik kinerja didasarkan pada ambang probabilitas 50% untuk setiap prediksi (dengan kata lain, jika model menghitung probabilitas 50% atau lebih tinggi bahwa suatu gambar berasal dari kelas tertentu, maka kelas tersebut diprediksi). Anda dapat menyesuaikan ini di kiri atas halaman.

## Menguji model

Sekarang setelah Anda melatih modelnya, Anda dapat mengujinya.

1. Di atas metrik performa, klik **Uji Cepat**.
1. Di kotak **URL Gambar**, ketik `https://aka.ms/apple-image` dan klik &#10132;
1. Lihat prediksi yang ditampilkan oleh model Anda - skor peluang untuk *apel* harus yang tertinggi, seperti ini:

![Gambar dengan prediksi kelas apel](../media/test-apple.jpg)

1. Tutup jendela **Uji Cepat**.

## Lihat pengaturan proyek

Proyek yang Anda buat telah diberi pengidentifikasi unik, yang perlu Anda tentukan dalam kode apa pun yang berinteraksi dengannya.

1. Klik ikon *pengaturan* (&#9881;) di kanan atas halaman **Kinerja** untuk melihat pengaturan proyek.
1. Di bawah **Umum** (di sebelah kiri), perhatikan **Project Id** yang secara unik mengidentifikasi proyek ini.
1. Di sebelah kanan, di bawah **Sumber daya** perhatikan bahwa kunci dan titik akhir ditampilkan. Ini adalah detail untuk sumber daya *pelatihan* (Anda juga dapat memperoleh informasi ini dengan melihat sumber daya di portal Microsoft Azure).

## Gunakan *pelatihan* API

Portal Custom Vision menyediakan antarmuka pengguna yang nyaman yang dapat Anda gunakan untuk mengunggah dan menandai gambar, dan melatih model. Namun, dalam beberapa skenario Anda mungkin ingin mengotomatiskan pelatihan model dengan menggunakan API pelatihan Visi Kustom.

> **Catatan**: Dalam latihan ini, Anda dapat memilih untuk menggunakan API dari **C#** atau **Python** SDK. Dalam langkah-langkah di bawah ini, lakukan tindakan yang sesuai untuk bahasa pilihan Anda.

1. Kembali ke Portal Azure, jalankan perintah `cd C-Sharp/train-classifier` atau `cd Python/train-classifier` bergantung pada preferensi bahasa Anda.
1. Instal paket Pelatihan Custom Vision dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-customvision==3.1.0
    ```

1. Menggunakan perintah `ls`, Anda dapat melihat konten folder **train-classifier**, dan perhatikan bahwa folder tersebut berisi file untuk pengaturan konfigurasi:
    - **C#**: appsettings.json
    - **Python**: .env

1. Masukkan perintah berikut untuk mengedit file konfigurasi yang telah disediakan:

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, perbarui nilai konfigurasi yang ada di dalamnya untuk mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya *pelatihan* Custom Vision Anda, dan ID proyek untuk proyek deteksi objek yang Anda buat sebelumnya.
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.
1. Perhatikan bahwa folder **train-classifier** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: train-classifier.py

    Buka file kode dan tinjau kode yang ada di dalamnya, perhatikan detail berikut:
    - Ruang nama dari paket yang Anda instal diimpor
    - Fungsi **Utama** mengambil pengaturan konfigurasi, dan menggunakan kunci dan titik akhir untuk membuat **CustomVisionTrainingClient** yang diautentikasi, yang kemudian digunakan dengan ID proyek untuk membuat **Proyek** referensi ke proyek Anda.
    - Fungsi **Upload_Images** mengambil tag yang ditentukan dalam proyek Custom Vision dan kemudian mengunggah file gambar dari folder bernama terkait ke proyek, menetapkan ID tag yang sesuai.
    - Fungsi **Train_Model** membuat iterasi pelatihan baru untuk proyek dan menunggu pelatihan selesai.
1. Tutup editor kode dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python train-classifier.py
    ```
    
1. Tunggu hingga program berakhir. Kemudian kembali ke browser Anda dan lihat halaman **Gambar Pelatihan** untuk proyek Anda di portal Custom Vision (menyegarkan browser jika perlu).
1. Verifikasi bahwa beberapa gambar baru yang diberi tag telah ditambahkan ke proyek. Kemudian lihat halaman **Kinerja** dan verifikasi bahwa iterasi baru telah dibuat.

## Terbitkan model klasifikasi gambar

Sekarang Anda siap untuk memublikasikan model terlatih Anda sehingga dapat digunakan dari aplikasi klien.

1. Di portal Visi Khusus, pada halaman **Kinerja**, klik **&#128504; Publikasikan** untuk memublikasikan model terlatih dengan pengaturan berikut:
    - **Nama model**: pengklasifikasi buah
    - **Sumber Daya Prediksi**: *Sumber daya **prediksi** yang Anda buat sebelumnya yang diakhiri dengan "-Prediction" (<u>bukan</u> sumber pelatihan)*.
1. Di kiri atas halaman **Pengaturan Proyek**, klik ikon *Galeri Proyek* (&#128065;) untuk kembali ke beranda portal Custom Vision, tempat proyek Anda sekarang terdaftar.
1. Pada beranda portal Custom Vision, di kanan atas, klik ikon *pengaturan* (&#9881;) untuk melihat pengaturan layanan Custom Vision Anda. Kemudian, di bawah **Sumber Daya**, temukan sumber daya *prediksi* Anda yang diakhiri dengan "-Prediksi" (<u>bukan</u> sumber daya pelatihan) untuk menentukan **Kunci** dan nilai **Titik Akhir** (Anda juga dapat memperoleh informasi ini dengan melihat sumber daya di portal Microsoft Azure).

## Gunakan pengklasifikasi gambar dari aplikasi klien

Sekarang setelah Anda memublikasikan model klasifikasi gambar, Anda dapat menggunakannya dari aplikasi klien. Sekali lagi, Anda dapat memilih untuk menggunakan **C#** atau **Python**.

1. Kembali ke Cloud Shell, jalankan perintah `cd ../test-classifier`, kemudian masukkan perintah khusus SDK berikut untuk menginstal paket Prediksi Custom Vision:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-customvision==3.1.0
    ```

> **Catatan**: Paket Python SDK mencakup paket pelatihan dan prediksi, dan mungkin sudah diinstal.

1. Menggunakan perintah `ls`, Anda dapat melihat isi folder **test-classifier**, yang digunakan untuk menerapkan aplikasi klien pengujian untuk model klasifikasi gambar Anda.
1. Buka file konfigurasi untuk aplikasi klien Anda (*appsettings.json* untuk C# atau *.env* untuk Python) dan perbarui nilai konfigurasi yang dikandungnya untuk mencerminkan titik akhir dan kunci untuk Visi Kustom Anda *prediksi* sumber daya, ID proyek untuk proyek klasifikasi, dan nama model yang Anda terbitkan (yang seharusnya *pengklasifikasi buah*). Simpan perubahan Anda dan tutup editor kode.
1. Buka file kode untuk aplikasi klien Anda (*Program.cs* untuk C#, *test-classification.py* untuk Python) dan tinjau kode yang ada di dalamnya, perhatikan detail berikut:
    - Ruang nama dari paket yang Anda instal diimpor
    - Fungsi **Utama** mengambil pengaturan konfigurasi, dan menggunakan kunci dan titik akhir untuk membuat **CustomVisionPredictionClient** yang diautentikasi.
    - Objek klien prediksi digunakan untuk memprediksi kelas untuk setiap gambar dalam folder **test-images**, yang menetapkan ID proyek dan nama model untuk setiap permintaan. Setiap prediksi menyertakan probabilitas untuk setiap kemungkinan kelas, dan hanya tag prediksi dengan probabilitas lebih besar dari 50% yang ditampilkan.
1. Tutup editor kode dan masukkan perintah khusus SDK berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python test-classifier.py
    ```

1. Lihat label (tag) dan skor probabilitas untuk setiap prediksi. Anda dapat mengunduh gambar dalam folder **test-images** untuk memverifikasi bahwa model telah mengklasifikasikannya dengan benar.

1. Di bar alat cloud shell, pilih **Unggah/Unduh file** dan pilih **Unduh**. Dalam kotak dialog baru, masukkan jalur file berikut dan pilih **Unduh**:

    **C#**
   
    ```
   mslearn-ai-vision/Labfiles/07-custom-vision-image-classification/C-Sharp/test-classifier/test-images/IMG_TEST_1.jpg
    ```

    **Python**
   
    ```
   mslearn-ai-vision/Labfiles/07-custom-vision-image-classification/Python/test-classifier/test-images/IMG_TEST_1.jpg
    ```

   Anda juga dapat mengunduh `IMG_TEST_2.jpg` dan `IMG_TEST_3.jpg` menggunakan jalur yang sama.

## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut.

1. Buka portal Microsoft Azure di`https://portal.azure.com`, dan di bilah pencarian atas, cari sumber daya yang Anda buat di lab ini.

1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya. Atau, Anda dapat menghapus seluruh grup sumber daya untuk membersihkan semua sumber daya secara bersamaan.

## Informasi selengkapnya

Untuk informasi lebih lanjut tentang klasifikasi gambar dengan layanan Custom Vision, lihat [dokumentasi Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
