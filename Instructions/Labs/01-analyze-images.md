---
lab:
  title: Menganalisis gambar
  description: 'Gunakan Analisis Gambar Azure AI Visual untuk menganalisis gambar, menyarankan keterangan dan tag, serta mendeteksi objek dan orang.'
---

# Menganalisis gambar

Azure AI Visual adalah kemampuan kecerdasan buatan yang memungkinkan sistem perangkat lunak menginterpretasikan input visual dengan menganalisis gambar. Di Microsoft Azure, layanan Azure AI **Vision** menyediakan model bawaan untuk tugas visi komputer umum, termasuk analisis gambar untuk menyarankan keterangan dan tag, deteksi objek dan orang umum. Anda juga dapat menggunakan layanan Azure AI Visual untuk menghapus latar belakang atau membuat pemotongan gambar latar depan.

> **Catatan**: Latihan ini didasarkan pada perangkat lunak SDK pra-rilis, yang mungkin dapat berubah. Jika perlu, kami telah menggunakan versi paket tertentu; yang mungkin tidak mencerminkan versi terbaru yang tersedia. Anda mungkin mengalami beberapa perilaku, peringatan, atau kesalahan tak terduga.

Meskipun latihan ini didasarkan pada SDK Python Azure Visual, Anda dapat mengembangkan aplikasi visual menggunakan beberapa SDK khusus bahasa; termasuk:

* [Analisis Azure AI Visual untuk JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Analisis Azure AI Visual untuk Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Analisis Azure AI Visual untuk Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Latihan ini memakan waktu sekitar **30** menit.

## Menyediakan sumber daya Azure AI Visual

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya Azure AI Visual.

> **Catatan**: Dalam latihan ini, Anda akan menggunakan sumber daya **Computer Vision** mandiri. Anda juga dapat menggunakan layanan Azure AI Visual di sumber daya multilayanan *Layanan Azure AI*, baik secara langsung atau dalam proyek *Azure AI Foundry*.

1. Buka [portal Azure](https://portal.azure.com) di `https://portal.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup pesan sambutan atau tips apa pun yang ditampilkan.
1. Pilih **Buat sumber daya**.
1. Di bilah pencarian, cari `Computer Vision`, pilih **Computer Vision**, dan buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat atau pilih grup sumber daya*
    - **Wilayah**: *Pilih dari **US Timur**, **US Barat**, **Prancis Tengah**, **Korea Tengah**, **Eropa Utara**, **Asia Tenggara**, **Eropa Barat**, atau **Asia Timur**\**
    - **Nama**: *Nama yang valid untuk sumber daya Computer Vision Anda*
    - **Tingkat harga**: Gratis F0

    \*Set fitur lengkap Azure AI Vision 4.0 saat ini hanya tersedia di wilayah ini.

1. Pilih kotak centang yang diperlukan dan buat sumber daya.
1. Tunggu hingga penyebaran selesai, lalu lihat detail penyebaran.
1. Ketika sumber daya telah disebarkan, buka dan di bawah node **Manajemen sumber daya** di panel navigasi, lihat halaman **Kunci dan Titik Akhir**. Anda akan memerlukan titik akhir dan salah satu kunci dari halaman ini dalam prosedur selanjutnya.

## Mengembangkan aplikasi analisis gambar dengan SDK Azure AI Visual

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk menganalisis gambar.

### Menyiapkan konfigurasi aplikasi

1. Di portal Azure, gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell ***dengan tidak ada penyimpanan pada langganan Anda.

    Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

    > **Catatan**: Jika portal meminta Anda untuk memilih penyimpanan untuk mempertahankan file Anda, pilih **Tidak ada akun penyimpanan yang diperlukan**, pilih langganan yang Anda gunakan dan tekan **Terapkan**.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Ubah ukuran panel cloud shell sehingga Anda masih dapat melihat halaman **Kunci dan Titik Akhir** untuk sumber daya Computer Vision Anda.

    > **Tips**" Anda bisa mengubah ukuran panel dengan menyeret batas atas. Anda juga dapat menggunakan tombol minimalkan dan maksimalkan untuk beralih antara shell cloud dan antarmuka portal utama.

1. Di panel cloud shell, masukkan perintah berikut untuk mengkloning repo GitHub yang berisi file kode untuk latihan ini (ketik perintah, atau salin ke clipboard lalu klik kanan di baris perintah dan tempel sebagai teks biasa):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouputnya mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, gunakan perintah berikut untuk membuka dan melihat folder berisi file kode aplikasi:   

    ```
   cd mslearn-ai-vision/Labfiles/analyze-images/python/image-analysis
   ls -a -l
    ```

    Folder berisi konfigurasi aplikasi dan file kode untuk aplikasi Anda. Folder ini juga berisi subfolder **/images**, yang berisi beberapa file gambar untuk dianalisis aplikasi Anda.
    
1. Instal paket SDK Azure AI Visual dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi untuk aplikasi Anda:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, perbarui nilai konfigurasi di dalamnya untuk mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya Computer Vision Anda (disalin dari halaman **Kunci dan Titik Akhir** di portal Azure).
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda lalu gunakan perintah **CTRL+Q** untuk menutup editor kode sambil menjaga baris perintah cloud shell tetap terbuka.

### Menambahkan kode untuk menyarankan keterangan

1. Di baris perintah cloud shell, masukkan perintah berikut untuk membuka file kode untuk aplikasi klien:

    ```
   code image-analysis.py
    ```

    > **Tips**: Anda mungkin ingin memaksimalkan panel cloud shell dan memindahkan bilah terpisah antara konsol baris perintah dan editor kode sehingga Anda dapat melihat kode dengan lebih mudah.

1. Dalam file kode, temukan komentar **Impor namespace layanan**, dan tambahkan kode berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Visual:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat pengaturan konfigurasi dan menentukan file gambar yang akan dianalisis telah disediakan. Kemudian, temukan komentar **Autentikasi klien Azure AI Visual** dan tambahkan kode berikut untuk membuat dan mengautentikasi objek klien Azure AI Visual (pastikan untuk mempertahankan tingkat indentasi yang benar):

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Dalam fungsi **Utama**, di bawah kode yang baru saja Anda tambahkan, temukan komentar **Analisis gambar** dan tambahkan kode berikut:

    ```python
   # Analyze image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print(f'\nAnalyzing {image_file}\n')

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.DENSE_CAPTIONS,
            VisualFeatures.TAGS,
            VisualFeatures.OBJECTS,
            VisualFeatures.PEOPLE],
   )
    ```

1. Temukan komentar **Dapatkan keterangan gambar**, tambahkan kode berikut untuk menampilkan keterangan gambar dan keterangan padat:

    ```python
   # Get image captions
   if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))
    
   if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))
    ```

1. Simpan perubahan Anda (*CTRL+S*) dan ubah ukuran panel sehingga Anda dapat dengan jelas melihat konsol baris perintah sambil tetap membuka editor kode. Kemudian, masukkan perintah berikut untuk menjalankan program dengan argumen **images/street.jpg**:

    ```
   python image-analysis.py images/street.jpg
    ```

1. Amati output, yang seharusnya menyertakan keterangan yang disarankan untuk gambar **street.jpg**, yang tampak seperti berikut:

    ![Gambar jalan yang sibuk.](../media/street.jpg)

1. Jalankan program lagi, kali ini dengan argumen **images/building.jpg** untuk melihat keterangan yang dihasilkan untuk gambar **building.jpg**, yang tampak seperti berikut:

    ![Gambar sebuah bangunan.](../media/building.jpg)

1. Ulangi langkah sebelumnya untuk membuat keterangan untuk file **images/person.jpg** yang tampak seperti berikut:

    ![Gambar seseorang.](../media/person.jpg)

### Menambahkan kode untuk membuat tag yang disarankan

Terkadang berguna untuk mengidentifikasi *tag* relevan yang memberikan petunjuk tentang konten gambar.

1. Di editor kode, dalam fungsi **AnalyzeImage**, temukan komentar **Dapatkan tag gambar** dan tambahkan kode berikut:

    ```python
   # Get image tags
   if result.tags is not None:
        print("\nTags:")
        for tag in result.tags.list:
            print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
    ```

1. Simpan perubahan Anda (*CTRL+S*) dan jalankan program dengan argumen **images/street.jpg**, amati bahwa selain keterangan gambar, daftar tag yang disarankan ditampilkan.
1. Jalankan ulang program untuk file **images/building.jpg** dan **images/person.jpg**.

### Menambahkan kode untuk mendeteksi dan menemukan objek

1. Di editor kode, dalam **fungsi AnalyzeImage**, temukan komentar **Dapatkan objek dalam gambar** dan tambahkan kode berikut untuk mencantumkan objek yang terdeteksi dalam gambar, lalu panggil fungsi yang disediakan untuk membuat anotasi gambar dengan objek yang terdeteksi:

    ```python
   # Get objects in the image
   if result.objects is not None:
        print("\nObjects in image:")
        for detected_object in result.objects.list:
            # Print object tag and confidence
            print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        # Annotate objects in the image
        show_objects(image_file, result.objects.list)
    ```

1. Simpan perubahan Anda (*CTRL+S*) dan jalankan program dengan argumen **images/street.jpg**, amati bahwa selain keterangan gambar dan tag yang disarankan, file bernama **objects.jpg** dibuat.
1. Gunakan perintah **unduhan** (khusus Azure Cloud Shell) untuk mengunduh file **objects.jpg**:

    ```
   download objects.jpg
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser Anda, yang dapat Anda pilih untuk mengunduh dan membuka file. Gambar akan terlihat seperti berikut ini:

    ![Gambar dengan kotak batas objek.](../media/objects.jpg)

1. Jalankan ulang program untuk file **images/building.jpg** dan **images/person.jpg**, dengan mengunduh file objects.jpg yang dihasilkan setelah setiap eksekusi.

### Menambahkan kode untuk mendeteksi dan menemukan orang

1. Di editor kode, dalam **fungsi AnalyzeImage**, temukan komentar **Dapatkan orang dalam gambar** dan tambahkan kode berikut untuk mencantumkan orang yang terdeteksi dengan tingkat keyakinan 20% atau lebih, dan panggil fungsi yang disediakan untuk membuat anotasi dalam gambar:

    ```Python
   # Get people in the image
   if result.people is not None:
        print("\nPeople in image:")

        for detected_person in result.people.list:
            if detected_person.confidence > 0.2:
                # Print location and confidence of each person detected
                print(" {} (confidence: {:.2f}%)".format(detected_person.bounding_box, detected_person.confidence * 100))
        # Annotate people in the image
        show_people(image_file, result.people.list)
    ```

1. Simpan perubahan Anda (*CTRL+S*) dan jalankan program dengan argumen **images/street.jpg**, amati bahwa selain keterangan gambar, dan tag yang disarankan, dan file objects.jpg, daftar lokasi orang dan file bernama **people.jpg** dibuat.

1. Gunakan perintah **unduhan** (khusus Azure Cloud Shell) untuk mengunduh file **objects.jpg**:

    ```
   download people.jpg
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser Anda, yang dapat Anda pilih untuk mengunduh dan membuka file. Gambar akan terlihat seperti berikut ini:

    ![Gambar dengan kotak batas untuk orang yang terdeteksi.](../media/people.jpg)

1. Jalankan ulang program untuk file **images/building.jpg** dan **images/person.jpg**, dengan mengunduh file people.jpg yang dihasilkan setelah setiap eksekusi.

   > **Tips:** Jika Anda melihat kotak pembatas yang dikembalikan dari model yang tidak masuk akal, periksa skor keyakinan JSON dan coba tingkatkan pemfilteran skor keyakinan di aplikasi Anda.

## Membersihkan sumber daya

Setelah selesai menjelajahi Azure AI Visual, Anda harus menghapus sumber daya yang telah Anda buat dalam latihan ini untuk menghindari biaya Azure yang tidak perlu:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

1. Di bilah pencarian atas, cari *Computer Vision*, lalu pilih sumber daya Computer Vision yang Anda buat di lab ini.

1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

