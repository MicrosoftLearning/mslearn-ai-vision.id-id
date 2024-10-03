---
lab:
  title: Klasifikasikan gambar dengan model kustom Azure AI Visual
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Klasifikasikan gambar dengan model kustom Azure AI Visual

Azure AI Visual memungkinkan Anda melatih model kustom untuk mengklasifikasikan dan mendeteksi objek dengan label yang Anda tentukan. Di lab ini, kita akan membangun model klasifikasi gambar kustom untuk mengklasifikasikan gambar buah.

## Mengkloning repositori untuk kursus ini

Jika Anda belum mengkloning repositori kode **Azure AI Visual** ke lingkungan tempat Anda mengerjakan lab ini, ikuti langkah-langkah berikut untuk melakukannya. Jika tidak, buka folder kloning di Visual Studio Code.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-vision` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.
4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**. Jika Anda menerima Pesan *Mendeteksi Proyek Fungsi Azure di folder*, Anda dapat menutup pesan tersebut dengan aman.

## Penyediaan sumber daya Azure

Jika belum memilikinya dalam langganan, Anda harus menyediakan sumber daya **Layanan Azure AI**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Di bilah pencarian teratas, cari *layanan Azure AI*, pilih **Layanan Azure AI**, dan buat sumber daya akun multi-layanan layanan Azure AI dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Wilayah**: *Pilih dari US Timur, Eropa Barat, US Barat 2\**
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Standar S0

    \*Tag model kustom Azure AI Visual 4.0 saat ini hanya tersedia di wilayah ini.

3. Pilih kotak centang yang diperlukan dan buat sumber daya.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Kami juga memerlukan akun penyimpanan untuk menyimpan gambar pelatihan.

1. Di portal Microsoft Azure, cari dan pilih **Akun penyimpanan**, dan buat akun penyimpanan baru dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih grup sumber daya yang sama dengan tempat Anda membuat sumber daya Layanan Azure AI*
    - **Nama Akun Penyimpanan**: customclassifySUFFIX 
        - *catatan: ganti `SUFFIX` token dengan inisial Anda atau nilai lain untuk memastikan nama sumber daya unik secara global.*
    - **Wilayah**: *Pilih wilayah yang sama dengan yang Anda gunakan untuk sumber daya Layanan Azure AI Anda*
    - **Layanan utama**: Azure Blob Storage atau Azure Data Lake Storage Gen 2
    - **Beban kerja utama**: Lainnya
    - **Performa**: Standar
    - **Redundansi**: Penyimpanan redundan secara lokal (LRS)
1. Saat akun penyimpanan Anda sedang dibuat, buka Visual studio code, dan luaskan folder **Labfiles/02-image-classification**.
1. Di folder tersebut, pilih **replace.ps1** dan tinjau kode. Anda akan melihat bahwa itu menggantikan nama akun penyimpanan Anda untuk tempat penampung dalam file JSON (file COCO) yang kami gunakan di langkah selanjutnya. Ganti tempat penampung *hanya di baris pertama* file dengan nama akun penyimpanan Anda. Simpan file.
1. Klik kanan pada folder**klasifikasi-gambar-02** dan buka Terminal Terintegrasi. Jalankan perintah berikut.

    ```powershell
    ./replace.ps1
    ```

1. Anda dapat meninjau file COCO untuk memastikan nama akun penyimpanan Anda ada di sana. Pilih **training-images/training_labels.json** dan tampilkan beberapa entri pertama. Di bidang *absolute_url*, Anda akan melihat sesuatu yang mirip dengan *"https://myStorage.blob.core.windows.net/fruit/...*. Jika tidak melihat perubahan yang diharapkan, pastikan Anda hanya memperbarui tempat penampung pertama dalam skrip Windows PowerShell.
1. Tutup file JSON serta Windows PowerShell, dan kembali ke jendela browser Anda.
1. Akun penyimpanan Anda harus lengkap. Buka akun penyimpanan Anda.
1. Aktifkan akses publik di akun penyimpanan. Di panel kiri, navigasikan ke **Konfigurasi** di grup **Pengaturan**, dan aktifkan *Perbolehkan akses anonim Blob* Pilih **Simpan**
1. Di panel kiri, di **Penyimpanan data**, pilih **Kontainer** dan buat kontainer baru bernama `fruit`, lalu atur **Level akses anonim** ke *Kontainer (akses baca anonim untuk kontainer dan blob)*.

    > **Catatan**: Jika **hubungan privasi anonim** dinonaktifkan, refresh halaman browser.

1. Arahkan ke `fruit`, pilih **Unggah**, dan unggah gambar (dan satu file JSON) di **Labfiles/02-image-classification/training-images** ke kontainer tersebut.

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
