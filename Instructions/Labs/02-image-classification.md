---
lab:
  title: Klasifikasikan gambar dengan model kustom Azure AI Visual
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Klasifikasikan gambar dengan model kustom Azure AI Visual

Azure AI Visual memungkinkan Anda melatih model kustom untuk mengklasifikasikan dan mendeteksi objek dengan label yang Anda tentukan. Di lab ini, kita akan membangun model klasifikasi gambar kustom untuk mengklasifikasikan gambar buah.

## Memprovisikan sumber daya Penglihatan Komputer

Jika belum memilikinya dalam langganan, Anda harus menyediakan sumber daya **Computer Vision**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
1. Pilih **Buat sumber daya**.
1. Di bilah pencarian, cari *Computer Vision*, pilih **Computer Vision**, dan buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Wilayah**: *Pilih dari AS Timur, AS Barat, Prancis Tengah, Korea Tengah, Eropa Utara, Asia Tenggara, Eropa Barat, atau Asia Timur\**
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Gratis F0

    \*Set fitur lengkap Azure AI Vision 4.0 saat ini hanya tersedia di wilayah ini.

1. Pilih kotak centang yang diperlukan dan buat sumber daya.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Kami juga memerlukan akun penyimpanan untuk menyimpan gambar pelatihan.

1. Di portal Microsoft Azure, cari dan pilih **Akun penyimpanan**, dan buat akun penyimpanan baru dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih grup sumber daya yang sama dengan tempat Anda membuat sumber daya Custom Vision di *
    - **Nama Akun Penyimpanan**: customclassifySUFFIX 
        - *Catatan: ganti token `SUFFIX` dengan inisial Anda atau nilai lain untuk memastikan nama sumber daya unik secara global.*
    - **Wilayah**: *Pilih wilayah yang sama dengan yang Anda gunakan untuk sumber daya Layanan Azure AI Anda*
    - **Layanan utama**: Azure Blob Storage atau Azure Data Lake Storage Gen 2
    - **Beban kerja utama**: Lainnya
    - **Performa**: Standar
    - **Redundansi**: Penyimpanan redundan secara lokal (LRS)

1. Ketika sumber daya sudah disebarkan, pilih **Buka sumber daya**.
1. Aktifkan akses publik di akun penyimpanan. Di panel kiri, navigasikan ke **Konfigurasi** di grup **Pengaturan**, dan aktifkan *Perbolehkan akses anonim Blob* Pilih **Simpan**
1. Di panel kiri, di **Penyimpanan data**, pilih **Kontainer** dan buat kontainer baru bernama `fruit`, lalu atur **Level akses anonim** ke *Kontainer (akses baca anonim untuk kontainer dan blob)*.

    > **Catatan**: Jika **hubungan privasi anonim** dinonaktifkan, refresh halaman browser.
   
## Mengkloning repositori untuk kursus ini

File gambar untuk melatih model Anda telah disediakan dalam repositori GitHub. Anda akan mengkloning repositori dan mengunggah gambar ke akun penyimpanan menggunakan Cloud Shell dari Portal Azure. 

> **Tips**: Jika Anda sudah mengkloning repositori **mslearn-ai-vision** baru-baru ini, Anda dapat melewati tugas klon. Jika belum, ikuti langkah-langkah berikut untuk mengkloning repositori ke lingkungan pengembangan Anda.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell***. Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Di panel PowerShell, masukkan perintah berikut untuk mengkloning repositori GitHub untuk latihan ini:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Setelah repositori dikloning, buka folder yang berisi file latihan:  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. Jalankan perintah `code replace.ps1` dan tinjau kode. Anda akan melihat bahwa itu menggantikan nama akun penyimpanan Anda untuk tempat penampung dalam file JSON (file COCO) yang kami gunakan di langkah selanjutnya.
1. Ganti tempat penampung *hanya di baris pertama* file dengan nama akun penyimpanan Anda.
1. Setelah Anda mengganti tempat penampung, gunakan perintah** CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.
1. Menjalankan skrip dengan menjalankan perintah berikut:

    ```powershell
    ./replace.ps1
    ```

1. Anda dapat meninjau file COCO untuk memastikan nama akun penyimpanan Anda ada di sana. Jalankan `code training-images/training_labels.json` dan lihat beberapa entri pertama. Di bidang *absolute_url*, Anda akan melihat sesuatu yang mirip dengan *"https://myStorage.blob.core.windows.net/fruit/...*. Jika tidak melihat perubahan yang diharapkan, pastikan Anda hanya memperbarui tempat penampung pertama dalam skrip Windows PowerShell.
1. Tutup editor kode.
1. Jalankan perintah berikut, ganti `<your-storage-account>` dengan nama akun penyimpanan Anda, untuk mengunggah konten folder **training-images** ke kontainer `fruit` yang Anda buat sebelumnya.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. Buka kontainer `fruit` dan verifikasi bahwa file diunggah dengan benar.

## Membuat proyek pelatihan model kustom

Selanjutnya, Anda akan membuat proyek pelatihan baru untuk klasifikasi gambar kustom di Vision Studio.

1. Di browser web, navigasikan ke `https://portal.vision.cognitive.azure.com/` dan masuk dengan akun Microsoft tempat Anda membuat sumber daya Azure AI.
1. Pilih petak **Sesuaikan model dengan gambar** (dapat ditemukan di tab **Analisis gambar** jika tidak muncul di tampilan default Anda).
1. Pilih akun Layanan Azure AI yang telah Anda buat.
1. Di proyek Anda, pilih **Tambahkan himpunan data baru** di bagian atas. Konfigurasikan dengan pengaturan berikut:
    - **Nama himpunan data**: training_images
    - **Tipe model**: Klasifikasi gambar
    - **Pilih kontainer Azure Blob Storage**: Pilih **Pilih Kontainer**
        - **Langganan**: *Langganan Azure Anda*
        - **Storage account**: *Akun penyimpanan yang Anda buat*
        - **Kontainer blob**: fruit
    - Pilih kotak untuk "Perbolehkan Vision Studio membaca dan menulis ke penyimpanan blob Anda"
1. Pilih himpunan data **training_images**.

Pada titik ini dalam pembuatan proyek, Anda biasanya akan memilih **Buat Proyek Pelabelan Data Azure ML** dan memberi label gambar Anda, yang menghasilkan file COCO. Anda didorong untuk mencoba ini jika memiliki waktu, tetapi untuk tujuan lab ini kami telah memberi label gambar untuk Anda dan menyediakan file COCO yang dihasilkan.

1. Pilih **Tambahkan file COCO**
1. Di menu dropdown, pilih **Impor file COCO dari Kontainer Blob**
1. Karena Anda telah menyambungkan kontainer bernama `fruit`, Vision Studio mencarinya untuk file COCO. Pilih **training_labels.json** dari menu dropdown, dan tambahkan file COCO.
1. Navigasikan ke **Model kustom** di sebelah kiri, dan pilih **Latih model baru**. Gunakan pengaturan berikut:
    - **Nama model**: classifyfruit
    - **Tipe model**: Klasifikasi gambar
    - **Pilih himpunan data pelatihan**: training_images
    - Biarkan sisanya sebagai default, dan pilih **Latih model**

Pelatihan dapat memakan waktu - anggaran default hingga satu jam, namun untuk himpunan data kecil ini biasanya jauh lebih cepat dari itu. Pilih tombol **Refresh** setiap beberapa menit hingga status tugas *Berhasil*. Pilih model.

Di sini Anda dapat menampilkan performa tugas pelatihan. Tinjau presisi dan akurasi model terlatih.

## Uji model kustom Anda

Model Anda telah dilatih dan siap untuk diuji.

1. Di bagian atas halaman untuk model kustom Anda, pilih **Coba**.
1. Pilih model**classifyfruit** dari menu dropdown yang menentukan model yang ingin Anda gunakan, dan telusuri ke folder**02-image-classification\test-images**.
1. Pilih setiap gambar dan tampilkan hasilnya. Pilih tab **JSON** di kotak hasil untuk memeriksa respons JSON lengkap.

<!-- Option coding example to run-->
## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut.

1. Buka portal Microsoft Azure di`https://portal.azure.com`, dan di bilah pencarian atas, cari sumber daya yang Anda buat di lab ini.

2. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya. Atau, Anda dapat menghapus seluruh grup sumber daya untuk membersihkan semua sumber daya secara bersamaan.
