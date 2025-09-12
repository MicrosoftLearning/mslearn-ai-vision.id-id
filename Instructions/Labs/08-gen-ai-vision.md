---
lab:
  title: Mengembangkan aplikasi obrolan berkemampuan visi
  description: Ggunakan Azure AI Foundry untuk membuat aplikasi AI generatif yang mendukung input gambar.
---

# Mengembangkan aplikasi obrolan berkemampuan visi

Dalam latihan ini, Anda menggunakan model AI generatif *Phi-4-multimodal-instruct* untuk menghasilkan respons terhadap perintah yang mencakup gambar. Anda akan mengembangkan aplikasi yang memberikan bantuan AI dengan produk baru di toko kelontong dengan menggunakan Azure AI Foundry dan layanan Inferensi Model Azure AI.

> **Catatan**: Latihan ini didasarkan pada perangkat lunak SDK pra-rilis, yang mungkin dapat berubah. Jika perlu, kami telah menggunakan versi paket tertentu; yang mungkin tidak mencerminkan versi terbaru yang tersedia. Anda mungkin mengalami beberapa perilaku, peringatan, atau kesalahan tak terduga.

Meskipun latihan ini didasarkan pada Azure AI Foundry Python SDK, Anda dapat mengembangkan aplikasi obrolan AI menggunakan beberapa SDK khusus bahasa; Termasuk:

- [Proyek Azure AI untuk Python](https://pypi.org/project/azure-ai-projects)
- [Proyek Azure AI untuk Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Proyek Azure AI untuk JavaScript](https://www.npmjs.com/package/@azure/ai-projects)

Latihan ini memakan waktu sekitar **30** menit.

## Buka portal Azure AI Foundry

Mari kita mulai dengan menjelajahi portal Azure AI Foundry.

1. Di browser web, buka [portal Azure AI Foundry](https://ai.azure.com) di `https://ai.azure.com` dan masuk menggunakan kredensial Azure Anda. Tutup semua tips atau panel mulai cepat yang terbuka saat pertama kali Anda masuk, dan jika perlu, gunakan logo **Azure AI Foundry** di kiri atas untuk menavigasi ke beranda, yang tampilannya mirip dengan gambar berikut (tutup panel **Bantuan** jika terbuka):

    ![Tangkapan layar portal Azure AI Foundry.](./media/ai-foundry-home.png)

1. Tinjau informasi di halaman beranda.

## Memilih model untuk memulai proyek

*Proyek* Azure AI menyediakan ruang kerja kolaboratif untuk pengembangan AI. Mari kita mulai dengan memilih model yang ingin kita kerjakan dan membuat proyek untuk menggunakannya.

> **Catatan**: Proyek AI Foundry dapat didasarkan pada *sumber daya Azure AI Foundry*yang menyediakan akses ke model AI (termasuk Azure OpenAI), layanan Azure AI, dan sumber daya lainnya untuk mengembangkan agen AI dan solusi obrolan. Alternatifnya, proyek dapat didasarkan pada sumber daya *pusat penyimpanan AI*; yang mencakup koneksi ke sumber daya Azure untuk penyimpanan, komputasi, dan alat khusus yang aman. Proyek berbasis Azure AI Foundry sangat bagus untuk pengembang yang ingin mengelola sumber daya untuk agen AI atau pengembangan aplikasi obrolan. Proyek berbasis pusat penyimpanan AI lebih cocok untuk tim pengembangan perusahaan yang mengerjakan solusi AI yang kompleks.

1. Di beranda, pada bagian **Jelajahi model dan kemampuan** , cari`Phi-4-multimodal-instruct` model; yang akan kita gunakan dalam proyek kita.

1. Dalam hasil pencarian, pilih model **Phi-4-multimodal-instruct** untuk melihat detailnya, lalu di bagian atas halaman model, pilih **Gunakan model ini**.

1. Saat diminta untuk membuat proyek, masukkan nama yang valid untuk proyek Anda dan perluas **Opsi tingkat lanjut**.

1. Pilih **Kustomisasi** dan tentukan pengaturan berikut untuk hub Anda:
    - **Sumber daya Azure AI Foundry**: *Nama yang valid untuk sumber daya Azure AI Foundry Anda*
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat atau pilih grup sumber daya*
    - **Wilayah**: *Pilih **AI Foundry yang direkomendasikan***\*

    > \* Sumber daya Azure OpenAI dibatasi oleh kuota model regional. Jika batas kuota terlampaui di kemudian hari dalam latihan, Anda mungkin perlu membuat sumber daya lain di wilayah yang berbeda.

1. Pilih **Buat** dan tunggu proyek Anda, termasuk penyebaran model Phi-4-multimodal-instruct yang Anda pilih, untuk membuatnya.

    > Catatan: Bergantung pada pilihan model Anda, Anda mungkin menerima perintah tambahan selama proses pembuatan proyek. Setujui ketentuan dan selesaikan penyebaran.

1. Saat proyek dibuat, model Anda akan ditampilkan di halaman **Model + titik akhir**:

    ![Cuplikan layar halaman penyebaran model.](./media/ai-foundry-model-deployment.png)

## Menguji model di playground

Sekarang Anda dapat menguji penyebaran model multimodal dengan perintah berbasis gambar di playground obrolan.

1. Pilih **Buka di playground** pada halaman penyebaran model.

1. Di tab browser baru, unduh [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) dari `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg` dan simpan ke folder di sistem file lokal Anda.

1. Pada halaman playground obrolan, di panel **Penyiapan**, pastikan penyebaran model **Phi-4-multimodal-instruct** Anda terpilih.

1. Di panel sesi obrolan utama, di bawah kotak input obrolan, gunakan tombol lampirkan (**&#128206;**) untuk mengunggah file gambar *mango.jpeg*, lalu tambahkan teks `What desserts could I make with this fruit?` dan kirimkan perintah.

    ![Cuplikan layar halaman playground obrolan.](../media/chat-playground-image.png)

1. Tinjau responsnya, yang diharapkan memberikan panduan yang relevan untuk hidangan penutup yang dapat Anda buat menggunakan mangga.

## Membuat aplikasi klien

Setelah menyebarkan model, Anda dapat menggunakan penyebaran dalam aplikasi klien.

### Menyiapkan konfigurasi aplikasi

1. Di portal Azure AI Foundry, lihat halaman **Gambaran Umum** untuk proyek Anda.

1. Di area **Titik akhir dan kunci**, pastikan pustaka **Azure AI Foundry** dipilih dan catat **titik akhir proyek Azure AI Foundry**. Anda akan menggunakan string koneksi ini untuk menyambungkan ke proyek Anda di aplikasi klien.

1. Buka tab browser baru (biarkan portal Azure AI Foundry tetap terbuka di tab yang sudah ada). Kemudian di tab baru, telusuri [Portal Azure](https://portal.azure.com) di `https://portal.azure.com`; masuk menggunakan kredensial Azure Anda jika diminta.

    Tutup pemberitahuan selamat datang apa pun untuk melihat halaman beranda portal Azure.

1. Gunakan tombol **[\>_]** di sebelah kanan bilah pencarian di bagian atas halaman untuk membuat Cloud Shell baru di portal Azure, dengan memilih lingkungan ***PowerShell*** dengan tidak ada penyimpanan pada langganan Anda.

    Cloud shell menyediakan antarmuka baris perintah dalam panel di bagian bawah portal Azure. Anda dapat mengubah ukuran atau memaksimalkan panel ini untuk mempermudah pekerjaan.

    > **Catatan**: Jika sebelumnya Anda telah membuat cloud shell yang menggunakan lingkungan *Bash* , alihkan ke ***PowerShell***.

    > **Catatan**: Jika portal meminta Anda untuk memilih penyimpanan untuk mempertahankan file Anda, pilih **Tidak ada akun penyimpanan yang diperlukan**, pilih langganan yang Anda gunakan dan tekan **Terapkan**.

1. Di toolbar cloud shell, di menu **Pengaturan**, pilih **Buka versi Klasik** (ini diperlukan untuk menggunakan editor kode).

    **<font color="red">Pastikan Anda telah beralih ke versi klasik cloud shell sebelum melanjutkan.</font>**

1. Di panel cloud shell, masukkan perintah berikut untuk mengkloning repo GitHub yang berisi file kode untuk latihan ini (ketik perintah, atau salin ke clipboard lalu klik kanan di baris perintah dan tempel sebagai teks biasa):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Tips**: Saat Anda menempelkan perintah ke cloudshell, ouput mungkin mengambil sejumlah besar buffer layar. Anda dapat menghapus layar dengan memasukkan `cls` perintah untuk mempermudah fokus pada setiap tugas.

1. Setelah repositori dikloning, navigasikan ke folder yang berisi file kode aplikasi obrolan:  

    ```
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. Di panel baris perintah cloud shell, masukkan perintah berikut untuk menginstal pustaka yang akan Anda gunakan:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Masukkan perintah berikut untuk mengedit file konfigurasi yang telah disediakan:

    ```
   code .env
    ```

    File dibuka dalam editor kode.

1. Dalam file kode, ganti tempat penampung **your_project_endpoint** dengan titik akhir proyek Foundry (disalin dari halaman **Gambaran umum** proyek di portal Azure AI Foundry), dan tempat penampung **your_model_deployment** dengan nama yang Anda tetapkan ke penyebaran model Phi-4-multimodal-instruct.

1. Setelah Anda mengganti tempat penampung, di editor kode,gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

### Menulis kode untuk menyambungkan ke proyek Anda dan mendapatkan klien obrolan untuk model Anda

> **Tips**: Saat Anda menambahkan kode, pastikan untuk mempertahankan indentasi yang benar.

1. Masukkan perintah berikut untuk mengedit file kode yang telah disediakan:

    ```
   code chat-app.py
    ```

1. Dalam file kode, perhatikan pernyataan yang sudah ada yang telah ditambahkan di bagian atas file untuk mengimpor namespace SDK yang diperlukan. Kemudian, temukan komentar **Tambahkan referensi**, tambahkan kode berikut untuk mereferensikan namespace di pustaka yang Anda instal sebelumnya:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. Dalam fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat string koneksi proyek dan nilai nama penyebaran model yang Anda tentukan dalam file konfigurasi.
1. Dalam fungsi **utama**, di bawah komentar **Dapatkan pengaturan konfigurasi**, perhatikan bahwa kode memuat string koneksi proyek dan nilai nama penyebaran model yang Anda tentukan dalam file konfigurasi.
1. Temukan komentar **Inisialisasi klien proyek**, lalu tambahkan kode berikut untuk menghubungkan proyek Azure AI Foundry Anda:

    > **Tips**: Berhati-hatilah untuk mempertahankan tingkat indentasi yang benar untuk kode Anda.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. Temukan komentar **Dapatkan klien obrolan**, tambahkan kode berikut untuk membuat objek klien untuk mengobrol dengan model:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Menulis kode untuk mengirimkan perintah gambar berbasis URL

1. Perhatikan bahwa kode menyertakan perulangan untuk memungkinkan pengguna memasukkan perintah hingga mereka memasukkan "berhenti". Kemudian di bagian loop, temukan komentar **Dapatkan respons terhadap input gambar**, tambahkan kode berikut untuk mengirimkan prompt yang menyertakan gambar berikut:

    ![Foto sebuah mangga.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. Gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda ke file kode - tetapi jangan ditutup dulu.

## Masuk ke Azure dan jalankan aplikasi.

1. Di panel baris perintah cloud shell, masukkan perintah berikut untuk menjalankan aplikasinya:

    ```
   az login
    ```

    **<font color="red">Anda harus masuk ke Azure - meskipun sesi cloud shell sudah diautentikasi.</font>**

    > **Catatan**: Dalam sebagian besar skenario, hanya menggunakan *login* az sudah cukup. Namun, jika Anda memiliki langganan di berbagai penyewa, Anda mungkin perlu menentukan penyewa dengan menggunakan *parameter --penyewa* . Lihat [Masuk ke Azure secara interaktif menggunakan Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) untuk detailnya.
    
1. Saat diperintahkan, ikuti instruksi untuk membuka halaman masuk di tab baru dan masukkan kode autentikasi yang diberikan dan kredensial Azure Anda. Kemudian selesaikan proses masuk di baris perintah, pilih langganan yang berisi pusat penyimpanan Azure AI Foundry jika diperintahkan.

1. Setelah Anda masuk, masukkan perintah berikut untuk menjalankan aplikasi:

    ```
   python chat-app.py
    ```

1. Saat diminta, masukkan perintah berikut:

    ```
   Suggest some recipes that include this fruit
    ```

1. Tinjau responsnya. Kemudian masukkan `quit` untuk keluar dari program.

### Mengubah kode untuk mengunggah file gambar lokal

1. Di editor kode untuk kode aplikasi Anda, di bagian loop, temukan kode yang Anda tambahkan sebelumnya di bawah komentar **Dapatkan respons terhadap input gambar**. Kemudian ubah kode sebagai berikut, untuk mengunggah file gambar lokal ini:

    ![Foto sebuah buah naga.](../media/mystery-fruit.jpeg)

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. Gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda ke file kode. Anda juga dapat menutup editor kode (**CTRL+Q**) jika mau.

1. Di panel baris perintah cloud shell di bawah editor kode, masukkan perintah berikut untuk menjalankan aplikasi:

    ```
   python chat-app.py
    ```

1. Saat diminta, masukkan perintah berikut:

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. Tinjau responsnya. Kemudian masukkan `quit` untuk keluar dari program.

    > **Catatan**: Dalam aplikasi sederhana ini, kita belum menerapkan logika untuk mempertahankan riwayat percakapan; sehingga model akan memperlakukan setiap perintah sebagai permintaan baru tanpa konteks perintah sebelumnya.

## Penghapusan

Setelah selesai menjelajahi Azure AI Foundry, Anda harus menghapus sumber daya yang telah Anda buat di latihan ini untuk menghindari biaya Azure yang tidak perlu.

1. Buka [portal Azure](https://portal.azure.com) dan lihat konten grup sumber daya tempat Anda menyebarkan sumber daya yang digunakan dalam latihan ini.
1. Pada toolbar pilih **Hapus grup sumber daya**.
1. Masukkan nama grup sumber daya untuk mengonfirmasi bahwa Anda ingin menghapusnya, dan pilih Hapus.
