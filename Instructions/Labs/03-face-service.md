---
lab:
  title: Deteksi dan analisis wajah
  description: Gunakan layanan Azure AI Vision Face untuk menerapkan solusi deteksi dan analisis wajah.
---

# Deteksi dan analisis wajah

Kemampuan untuk mendeteksi dan menganalisis wajah manusia adalah kemampuan inti AI. Dalam latihan ini, Anda akan mempelajari **layanan Face** untuk bekerja dengan wajah.

> **Catatan**: Latihan ini didasarkan pada perangkat lunak SDK pra-rilis, yang mungkin dapat berubah. Jika perlu, kami telah menggunakan versi paket tertentu; yang mungkin tidak mencerminkan versi terbaru yang tersedia. Anda mungkin mengalami beberapa perilaku, peringatan, atau kesalahan tak terduga.

Meskipun latihan ini didasarkan pada SDK Python Azure Vision Face, Anda dapat mengembangkan aplikasi visual menggunakan beberapa SDK khusus bahasa; termasuk:

* [Azure AI Vision Face untuk JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-face)
* [Azure AI Vision Face untuk Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.Face)
* [Azure AI Vision Face untuk Java](https://central.sonatype.com/artifact/com.azure/azure-ai-vision-face)

Latihan ini memakan waktu sekitar **30** menit.

> **Catatan**: Kemampuan layanan Azure AI yang menampilkan informasi pribadi dibatasi untuk pelanggan yang telah diberi [akses terbatas](https://learn.microsoft.com/legal/cognitive-services/computer-vision/limited-access-identity). Latihan ini tidak termasuk tugas pengenalan wajah, dan dapat diselesaikan tanpa meminta akses tambahan ke fitur terbatas.

## Menyediakan sumber daya Azure AI Face API

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya Azure AI Face API.

> **Catatan**: Dalam latihan ini, Anda akan menggunakan sumber daya **Face** mandiri. Anda juga dapat menggunakan layanan Azure AI Face di sumber daya multilayanan *Layanan Azure AI*, baik secara langsung atau dalam proyek *Azure AI Foundry*.

1. Buka [portal Azure](https://portal.azure.com) di `https://portal.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup pesan sambutan atau tips apa pun yang ditampilkan.
1. Pilih **Buat sumber daya**.
1. Di bilah pencarian, cari `Face`, pilih **Face**, dan buat sumber daya dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat atau pilih grup sumber daya*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Nama**: *Nama yang valid untuk sumber daya Face*
    - **Tingkat harga**: Gratis F0

1. Buat sumber daya dan tunggu hingga penyebaran selesai, lalu lihat detail penyebaran.
1. Ketika sumber daya telah disebarkan, buka dan di bawah node **Manajemen sumber daya** di panel navigasi, lihat halaman **Kunci dan Titik Akhir**. Anda akan memerlukan titik akhir dan salah satu kunci dari halaman ini dalam prosedur selanjutnya.

## Mengembangkan aplikasi analisis wajah dengan Face SDK

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan SDK Azure Face untuk mendeteksi dan menganalisis wajah manusia dalam gambar.

### Menyiapkan konfigurasi aplikasi

1. Di portal Azure, gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell ***dengan tidak ada penyimpanan pada langganan Anda.

    Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

    > **Catatan**: Jika portal meminta Anda untuk memilih penyimpanan untuk mempertahankan file Anda, pilih **Tidak ada akun penyimpanan yang diperlukan**, pilih langganan yang Anda gunakan dan tekan **Terapkan**.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Ubah ukuran panel cloud shell sehingga Anda masih dapat melihat halaman **Kunci dan Titik Akhir** untuk sumber daya Face Anda.

    > **Tips**" Anda bisa mengubah ukuran panel dengan menyeret batas atas. Anda juga dapat menggunakan tombol minimalkan dan maksimalkan untuk beralih antara shell cloud dan antarmuka portal utama.

1. Di panel cloud shell, masukkan perintah berikut untuk mengkloning repo GitHub yang berisi file kode untuk latihan ini (ketik perintah, atau salin ke clipboard lalu klik kanan di baris perintah dan tempel sebagai teks biasa):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouputnya mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, gunakan perintah berikut untuk menavigasi ke folder berisi file kode aplikasi:

    ```
   cd mslearn-ai-vision/Labfiles/face/python/face-api
   ls -a -l
    ```

    Folder berisi konfigurasi aplikasi dan file kode untuk aplikasi Anda. Folder ini juga berisi subfolder **/images**, yang berisi beberapa file gambar untuk dianalisis aplikasi Anda.

1. Instal paket SDK Azure AI Visual dan paket lain yang diperlukan dengan menjalankan perintah berikut:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-face==1.0.0b2
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi untuk aplikasi Anda:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, perbarui nilai konfigurasi di dalamnya untuk mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya Face Anda (disalin dari halaman **Kunci dan Titik Akhir** di portal Azure).
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda lalu gunakan perintah **CTRL+Q** untuk menutup editor kode sambil menjaga baris perintah cloud shell tetap terbuka.

### Menambahkan kode untuk membuat klien Face API

1. Di baris perintah cloud shell, masukkan perintah berikut untuk membuka file kode untuk aplikasi klien:

    ```
   code analyze-faces.py
    ```

    > **Tips**: Anda mungkin ingin memaksimalkan panel cloud shell dan memindahkan bilah terpisah antara konsol baris perintah dan editor kode sehingga Anda dapat melihat kode dengan lebih mudah.

1. Dalam file kode, temukan komentar **Impor namespace layanan**, dan tambahkan kode berikut untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Visual:

    ```python
   # Import namespaces
   from azure.ai.vision.face import FaceClient
   from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection01
   from azure.core.credentials import AzureKeyCredential
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat pengaturan konfigurasi dan menentukan gambar yang akan dianalisis telah disediakan. Kemudian temukan komentar **Autentikasi wajah klien** dan tambahkan kode berikut untuk membuat dan mengautentikasi objek **FaceClient**:

    ```python
   # Authenticate Face client
   face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key))
    ```

### Menambahkan kode untuk mendeteksi dan menganalisis wajah

1. Dalam file kode untuk aplikasi Anda, di fungsi **Utama**, temukan komentar **Tentukan fitur wajah yang akan diambil** dan tambahkan kode berikut:

    ```python
   # Specify facial features to be retrieved
   features = [FaceAttributeTypeDetection01.HEAD_POSE,
                FaceAttributeTypeDetection01.OCCLUSION,
                FaceAttributeTypeDetection01.ACCESSORIES]
    ```

1. **Dalam fungsi Utama**, di bawah kode yang baru saja Anda tambahkan, temukan komentar **Dapatkan wajah** dan tambahkan kode berikut untuk mencetak informasi fitur wajah dan panggil fungsi yang menganotasi gambar dengan kotak pembatas untuk setiap wajah yang terdeteksi (berdasarkan properti **face_rectangle** setiap wajah):

    ```Python
   # Get faces
   with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION01,
            recognition_model=FaceRecognitionModel.RECOGNITION01,
            return_face_id=False,
            return_face_attributes=features,
        )

   face_count = 0
   if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')
        for face in detected_faces:
    
            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))
            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Forehead occluded?: {}'.format(face.face_attributes.occlusion["foreheadOccluded"]))
            print(' - Eye occluded?: {}'.format(face.face_attributes.occlusion["eyeOccluded"]))
            print(' - Mouth occluded?: {}'.format(face.face_attributes.occlusion["mouthOccluded"]))
            print(' - Accessories:')
            for accessory in face.face_attributes.accessories:
                print('   - {}'.format(accessory.type))
            # Annotate faces in the image
            annotate_faces(image_file, detected_faces)
    ```

1. Periksa kode yang Anda tambahkan ke fungsi **Utama**. Kode ini menganalisis file gambar dan mendeteksi wajah apa pun yang ada di dalamnya, termasuk atribut untuk pose kepala, halangan, dan aksesori seperti kacamata. Selain itu, fungsi dipanggil untuk membuat anotasi gambar asli dengan kotak pembatas untuk setiap wajah yang terdeteksi.
1. Simpan perubahan Anda (*CTRL+S*) tetapi biarkan editor kode terbuka jika Anda perlu memperbaiki kesalahan ejaan apa pun.

1. Ubah ukuran panel sehingga Anda dapat melihat lebih banyak bagian konsol, lalu masukkan perintah berikut untuk menjalankan program dengan argumen *images/face1.jpg*:

    ```
   python analyze-faces.py images/face1.jpg
    ```

    Aplikasi menjalankan dan menganalisis gambar berikut:

    ![Foto patung seseorang.](../media/face1.jpg)

1. Amati output, yang seharusnya menyertakan ID dan atribut setiap wajah yang terdeteksi. 
1. Perhatikan bahwa file gambar bernama **detected_faces.jpg** juga dibuat. Gunakan perintah **unduhan** (khusus Azure Cloud Shell) untuk mengunduhnya:

    ```
   download detected_faces.jpg
    ```

    Perintah unduh membuat tautan popup di kanan bawah browser Anda, yang dapat Anda pilih untuk mengunduh dan membuka file. Gambar akan tampak seperti berikut ini:

    ![Gambar dengan wajah disorot.](../media/detected_faces1.jpg)

1. Jalankan program lagi, kali ini tentukan parameter *images/face2.jpg* untuk mengekstrak teks dari gambar berikut:

    ![Gambar orang lain.](../media/face2.jpg)

    ```
   python analyze-faces.py images/face2.jpg
    ```

1. Unduh dan lihat file **detected_faces.jpg** yang dihasilkan:

    ```
   download detected_faces.jpg
    ```

    Hasil gambar akan terlihat seperti ini:

    ![Gambar lain dengan wajah disorot.](../media/detected_faces2.jpg)

1. Jalankan program sekali lagi, kali ini dengan menentukan parameter *images/faces.jpg* untuk mengekstrak teks dari gambar ini:

    ![Foto kedua orang.](../media/faces.jpg)

    ```
   python analyze-faces.py images/faces.jpg
    ```

1. Unduh dan lihat file **detected_faces.jpg** yang dihasilkan:

    ```
   download detected_faces.jpg
    ```

    Hasil gambar akan terlihat seperti ini:

    ![Gambar lain dengan wajah disorot.](../media/detected_faces3.jpg)

## Membersihkan sumber daya

Setelah selesai menjelajahi Azure AI Visual, Anda harus menghapus sumber daya yang telah Anda buat dalam latihan ini untuk menghindari biaya Azure yang tidak perlu:

1. Buka portal Microsoft Azure di`https://portal.azure.com`, dan di bilah pencarian atas, cari sumber daya yang Anda buat di lab ini.

1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya. Atau, Anda dapat menghapus seluruh grup sumber daya untuk membersihkan semua sumber daya secara bersamaan.
