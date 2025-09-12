---
lab:
  title: Deteksi dan Analisis Wajah
  module: Module 4 - Detecting and Analyze Faces
---

# Deteksi dan Analisis Wajah

Kemampuan untuk mendeteksi dan menganalisis wajah manusia adalah kemampuan inti AI. Dalam latihan ini, Anda akan menjelajahi dua Layanan Azure AI yang dapat Anda gunakan untuk bekerja dengan wajah dalam gambar: layanan ** Azure AI Vision**, dan layanan **Face**.

> **Penting**: Lab ini dapat diselesaikan tanpa meminta akses tambahan ke fitur terbatas.

> **Catatan**: Mulai 21 Juni 2022, kemampuan layanan Azure AI yang mengembalikan informasi identitas pribadi dibatasi untuk pelanggan yang telah diberikan [akses terbatas](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Selain itu, kemampuan yang menyimpulkan keadaan emosional tidak lagi tersedia. Untuk detail selengkapnya tentang perubahan yang telah dilakukan Microsoft, dan mengapa - lihat [Investasi dan perlindungan AI yang bertanggung jawab untuk pengenalan wajah](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

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
1. Tunggu hingga penyebaran selesai, lalu lihat detail penyebaran.
1. Saat sumber daya telah diterapkan, buka dan lihat halaman **Kunci dan Titik Akhir**. Anda akan memerlukan titik akhir dan salah satu kunci dari halaman ini dalam prosedur selanjutnya.

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
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## Bersiap untuk menggunakan Azure AI Visual SDK

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk menganalisis gambar.

> **Catatan**: Anda dapat memilih untuk menggunakan SDK baik untuk **C#** atau **Python**. Dalam langkah-langkah di bawah ini, lakukan tindakan yang sesuai untuk bahasa pilihan Anda.

1. Navigasi ke folder yang berisi file kode aplikasi untuk bahasa pilihan Anda:  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. Instal paket SDK Azure AI Visual dan dependensi yang diperlukan dengan menjalankan perintah sesuai preferensi bahasa Anda:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. Dengan menggunakan perintah `ls`, Anda dapat melihat konten folder **computer-vision**. Perhatikan bahwa ini berisi file untuk pengaturan konfigurasi:

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

1. Dalam file kode, perbarui nilai konfigurasi di dalamnya untuk mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya Computer Vision Anda.
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** untuk menyimpan perubahan Anda lalu gunakan perintah **CTRL+Q** untuk menutup editor kode sambil menjaga baris perintah cloud shell tetap terbuka.

## Melihat gambar yang akan Anda analisis

Dalam latihan ini, Anda akan menggunakan layanan Azure AI Vision untuk menganalisis gambar orang.

1. Di bar alat cloud shell, pilih **Unggah/Unduh file** dan pilih **Unduh**. Dalam kotak dialog baru, masukkan jalur file berikut dan pilih **Unduh**:

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. Buka gambar **people.jpg** untuk melihatnya.

## Mendeteksi wajah dalam gambar

Sekarang Anda siap menggunakan SDK untuk memanggil layanan Visi dan mendeteksi wajah dalam gambar.

1. Perhatikan bahwa folder **computer-vision** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: detect-people.py

1. Masukkan perintah berikut untuk membuka file kode untuk aplikasi klien:

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. Tambahkan kode khusus bahasa berikut di bawah komentar **Impor namespace layanan** untuk mengimpor namespace layanan yang akan Anda perlukan untuk menggunakan SDK Azure AI Visual:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat pengaturan konfigurasi telah disediakan. Kemudian temukan komentar **Autentikasi klien Azure AI Visual**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk membuat dan mengautentikasi objek klien Azure AI Visual:

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

1. Dalam fungsi **Utama**, di bawah kode yang baru saja Anda tambahkan, perhatikan bahwa kode menentukan jalur ke file gambar lalu meneruskan jalur gambar ke fungsi bernama **AnalyzeImage**. Fungsi ini belum sepenuhnya diimplementasikan.

1. Dalam fungsi **AnalyzeImage** di bawah komentar **Dapatkan hasil dengan menentukan fitur yang akan diambil (PEOPLE)**, tambahkan kode berikut:

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

1. Dalam fungsi **AnalyzeImage** di bawah komentar **Gambar kotak batas di sekitar orang yang terdeteksi**, tambahkan kode berikut:

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

1. Simpan perubahan, tutup editor kode, dan masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. Amati output, yang harus menunjukkan jumlah wajah yang terdeteksi.
7. Unduh file **people.jpg** yang dihasilkan di folder yang sama seperti file kode Anda untuk melihat wajah yang dianotasi. Dalam hal ini, kode Anda telah menggunakan atribut wajah untuk melabeli lokasi di kiri atas kotak, dan koordinat kotak pembatas untuk menggambar persegi panjang di sekitar setiap wajah.

Jika Anda ingin melihat skor keyakinan semua orang yang terdeteksi layanan, Anda dapat membatalkan komentar baris kode di bawah komentar `Return the confidence of the person detected` dan menjalankan ulang kode.

## Bersiap untuk menggunakan Face SDK

Meskipun layanan **Azure AI Vision** menawarkan deteksi wajah dasar (bersama dengan banyak kemampuan analisis gambar lainnya), layanan **Face** menyediakan fungsionalitas yang lebih komprehensif untuk analisis dan pengenalan wajah.

1. Navigasikan ke folder **face-api** dengan menjalankan perintah `cd ../face-api`. Kemudian instal paket Face SDK dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. Lihat konten folder **face-api**, dan perhatikan bahwa folder tersebut berisi file untuk setelan konfigurasi:
    - **C#**: appsettings.json
    - **Python**: .env

1. Buka file konfigurasi dan perbarui nilai konfigurasi di dalamnya agar mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya layanan Azure AI Anda. Simpan perubahan Anda.

1. Perhatikan bahwa folder **face-api** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

1. Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace, Anda harus menggunakan Vision SDK:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. Dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat pengaturan konfigurasi telah disediakan. Kemudian temukan komentar **Autentikasi klien Wajah**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk membuat dan mengautentikasi objek **FaceClient** :

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

1. Dalam fungsi **Utama**, di bawah kode yang baru saja Anda tambahkan, perhatikan bahwa kode menampilkan menu yang memungkinkan Anda memanggil fungsi dalam kode Anda untuk menjelajahi kemampuan layanan Face. Anda akan menerapkan fungsi-fungsi ini di sisa latihan ini.

## Deteksi dan analisis wajah

Salah satu kemampuan paling mendasar dari layanan Wajah adalah mendeteksi wajah dalam gambar, dan menentukan atributnya, seperti pose kepala, buram, keberadaan topeng, dan sebagainya.

1. Dalam file kode untuk aplikasi Anda, dalam fungsi **Utama**, periksa kode yang berjalan jika pengguna memilih opsi menu **1**. Kode ini memanggil fungsi **DetectFaces**, meneruskan jalur ke file gambar.
1. Temukan fungsi **DetectFaces** dalam file kode, dan di bawah komentar **Tentukan fitur wajah yang akan diambil**, tambahkan kode berikut:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

1. Dalam fungsi **DetectFaces**, di bawah kode yang baru saja Anda tambahkan, temukan komentar **Dapatkan wajah** dan tambahkan kode berikut:

    **C#**

    ```C#
    // Get faces
    using (var imageData = File.OpenRead(imageFile))
    {    
        var response = await faceClient.DetectAsync(
            BinaryData.FromStream(imageData),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: false,
            returnFaceAttributes: features);
        IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

        if (detected_faces.Count() > 0)
        {
            Console.WriteLine($"{detected_faces.Count()} faces detected.");

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

            int faceCount=0;

            // Draw and annotate each face
            foreach (var face in detected_faces)
            {
                faceCount++;
                Console.WriteLine($"\nFace number {faceCount}");
            
                // Get face properties
                Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
                Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
                Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
                Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
                Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

                // Draw and annotate face
                var r = face.FaceRectangle;

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
        }
    }
    ```

    **Python**

    ```Python
    # Get faces
    with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=False,
            return_face_attributes=features,
        )

        if len(detected_faces) > 0:
            print(len(detected_faces), 'faces detected.')

            # Prepare image for drawing
            fig = plt.figure(figsize=(8, 6))
            plt.axis('off')
            image = Image.open(image_file)
            draw = ImageDraw.Draw(image)
            color = 'lightgreen'
            face_count = 0

            # Draw and annotate each face
            for face in detected_faces:
    
                # Get face properties
                face_count += 1
                print('\nFace number {}'.format(face_count))

                print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
                print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
                print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
                print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
                print(' - Mask: {}'.format(face.face_attributes.mask.type))

                # Draw and annotate face
                r = face.face_rectangle
                bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
                draw = ImageDraw.Draw(image)
                draw.rectangle(bounding_box, outline=color, width=5)
                annotation = 'Face number {}'.format(face_count)
                plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

            # Save annotated image
            plt.imshow(image)
            outputfile = 'detected_faces.jpg'
            fig.savefig(outputfile)

            print('\nResults saved in', outputfile)
    ```

1. Periksa kode yang Anda tambahkan ke fungsi **DetectFaces**. Ini menganalisis file gambar dan mendeteksi wajah apa pun yang ada di dalamnya, termasuk atribut untuk pose kepala, buram, dan keberadaan topeng. Detail setiap wajah ditampilkan, termasuk pengidentifikasi wajah unik yang ditetapkan untuk setiap wajah; dan lokasi wajah ditunjukkan pada gambar menggunakan kotak pembatas.
1. Simpan perubahan dan tutup editor kode, lalu masukkan perintah berikut untuk menjalankan program:

    **C#**

    ```
    dotnet run
    ```

    *Output C# mungkin menampilkan peringatan tentang fungsi asinkron yang sekarang menggunakan operator **menunggu** Anda dapat mengabaikan ini.*

    **Python**

    ```
    python analyze-faces.py
    ```

1. Saat diminta, masukkan **1** dan amati output, yang harus menyertakan ID dan atribut setiap wajah yang terdeteksi.
1. Unduh file **detected_faces.jpg** yang dihasilkan di folder yang sama seperti file kode Anda untuk melihat wajah yang dianotasi.

## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut.

1. Buka portal Microsoft Azure di`https://portal.azure.com`, dan di bilah pencarian atas, cari sumber daya yang Anda buat di lab ini.

1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya. Atau, Anda dapat menghapus seluruh grup sumber daya untuk membersihkan semua sumber daya secara bersamaan.

## Informasi selengkapnya

Ada beberapa fitur tambahan yang tersedia dalam layanan **Face**, tetapi mengikuti [Standar AI yang Bertanggung Jawab](https://aka.ms/aah91ff) yang dibatasi di balik kebijakan Akses Terbatas. Fitur-fitur ini termasuk mengidentifikasi, memverifikasi, dan membuat model pengenalan wajah. Untuk mempelajari selengkapnya dan mengajukan permohonan akses, lihat [Akses Terbatas untuk Layanan Azure AI](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Untuk informasi selengkapnya tentang menggunakan layanan **Azure AI Vision** untuk deteksi wajah, lihat [dokumentasi Azure AI Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Untuk mempelajari lebih lanjut tentang layanan **Wajah**, lihat [dokumentasi Wajah](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity).
