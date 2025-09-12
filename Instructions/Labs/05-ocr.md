---
lab:
  title: Membaca Teks dalam Gambar
  module: Module 11 - Reading Text in Images and Documents
---

# Membaca Teks dalam Gambar

Pengenalan karakter optik (OCR) adalah bagian dari visi komputer yang berhubungan dengan membaca teks dalam gambar dan dokumen. Layanan **Azure AI Visual** menyediakan API untuk membaca teks, yang akan Anda jelajahi dalam latihan ini.

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Bersiap untuk menggunakan Azure AI Visual SDK

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk membaca teks.

> **Catatan**: Anda dapat memilih untuk menggunakan SDK baik untuk **C#** atau **Python**. Dalam langkah-langkah di bawah ini, lakukan tindakan yang sesuai untuk bahasa pilihan Anda.

1. Navigasi ke folder yang berisi file kode aplikasi untuk bahasa pilihan Anda:  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
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
1. Setelah Anda mengganti tempat penampung, gunakan perintah **CTRL+S** atau **Klik kanan > Simpan** untuk menyimpan perubahan Anda dan kemudian gunakan perintah **CTRL+Q** atau **Klik kanan > Keluar** untuk menutup editor kode sambil tetap membuka baris perintah cloud shell.

## Menggunakan Azure AI Visual SDK untuk membaca teks dari gambar

Salah satu fitur **SDK Azure AI Visual** adalah membaca teks dari gambar. Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk membaca teks dari gambar.

1. Folder **read-text** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang Anda perlukan untuk menggunakan Azure AI Visual SDK:

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

1. Dalam file kode untuk aplikasi klien Anda, dalam fungsi **Main**, kode untuk memuat pengaturan konfigurasi telah disediakan. Kemudian, temukan komentar **Mengautentikasi klien Azure AI Visual**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk membuat dan mengautentikasi objek klien Azure AI Visual:

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
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

1. Dalam fungsi **Main**, di bawah kode yang baru saja Anda tambahkan, perhatikan bahwa kode tersebut menentukan jalur ke file gambar dan kemudian meneruskan jalur gambar ke fungsi **GetTextRead**. Fungsi ini belum sepenuhnya diterapkan.

1. Mari menambahkan beberapa kode ke isi fungsi **GetTextRead**. Temukan komentar **Menggunakan fungsi Analisis gambar untuk membaca teks dalam gambar**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut, perhatikan bahwa fitur visual ditentukan saat memanggil fungsi `Analyze`:

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

1. Dalam kode yang baru saja Anda tambahkan dalam fungsi **GetTextRead**, dan di bawah komentar **Menghasilkan teks yang terdeteksi dalam gambar**, tambahkan kode berikut (kode ini mencetak teks gambar ke konsol dan menghasilkan gambar **text.jpg** yang menyoroti teks gambar):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Khusus untuk file program C#, fungsi bantuan masih diperlukan untuk menggambar poligon. Di bawah komentar **metode Bantuan untuk menggambar poligon yang diberi array SKPoints**, tambahkan kode berikut:

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. Dalam fungsi **Utama**, periksa kode yang berjalan jika pengguna memilih opsi menu **1**. Kode ini memanggil fungsi **GetTextRead**, meneruskan jalur ke file gambar *Lincoln.jpg*.

1. Simpan perubahan Anda dan tutup editor kode.

1. Di bar alat cloud shell, pilih **Unggah/Unduh file** dan pilih **Unduh**. Dalam kotak dialog baru, masukkan jalur file berikut dan pilih **Unduh**:

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. Buka gambar **Lincoln.jpg** untuk melihatnya.

1. Setelah melihat gambar yang diproses kode Anda, masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Saat diminta, masukkan **1** dan amati hasilnya, yaitu teks yang diekstrak dari gambar.

1. Di folder **read-text**, gambar **text.jpg** telah dibuat. Anda dapat mengunduhnya menggunakan jalur file `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` dan melihat ada poligon di sekeliling setiap *baris* teks.

1. Buka kembali file kode dan temukan komentar **Tampilkan kotak pembatas posisi di sekeliling setiap baris**. Kemudian, di bawah komentar ini, tambahkan kode berikut:

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

1. Simpan perubahan dan tutup editor kode, lalu masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Ketika diminta, masukkan **1** dan amati output yang dihasilkan. Output-nya harus terdiri dari setiap baris teks dalam gambar, bersama dengan posisinya masing-masing dalam gambar.

1. Buka file kode lagi dan temukan komentar **Menghasilkan setiap kata yang terdeteksi dalam gambar dan kotak pembatas posisi di sekeliling setiap kata dengan tingkat keyakinan untuk setiap kata**. Kemudian, di bawah komentar ini, tambahkan kode berikut:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Simpan perubahan dan tutup editor kode, lalu masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Ketika diminta, masukkan **1** dan amati output yang dihasilkan. Output-nya harus terdiri dari setiap kata dalam teks pada gambar, bersama dengan posisinya masing-masing dalam gambar. Perhatikan bahwa informasi yang dihasilkan juga mencakup tingkat keyakinan untuk setiap kata.

1. Unduh gambar **text.jpg** lagi dan perhatikan bahwa ada poligon di sekeliling setiap *kata*.

## Menggunakan Azure AI Visual SDK untuk membaca teks tulisan tangan dari gambar

Dalam latihan sebelumnya, Anda membaca teks yang terdefinisi dengan baik dari gambar. Namun, terkadang Anda mungkin juga ingin membaca teks dari catatan atau naskah tulisan tangan. Kabar baiknya adalah **Azure AI Visual SDK** juga dapat membaca teks tulisan tangan dengan kode yang sama persis dengan yang Anda gunakan untuk membaca teks yang terdefinisi dengan baik. Kita akan menggunakan kode yang sama dari latihan sebelumnya, tetapi kali ini kita akan menggunakan gambar yang berbeda.

1. Unduh **Note.jpg** menggunakan jalur file `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` untuk melihat gambar berikutnya yang diproses kode Anda.

1. Dalam file kode untuk aplikasi Anda, dalam fungsi **Utama**, periksa kode yang berjalan jika pengguna memilih opsi menu **2**. Kode ini memanggil fungsi **GetTextRead**, meneruskan jalur ke file gambar *Note.jpg*.

1. Dari terminal, masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Ketika diminta, masukkan **2** dan amati output yang dihasilkan. Output-nya berupa teks yang diekstrak dari gambar catatan.

1. Unduh gambar **text.jpg** lagi dan perhatikan bahwa ada poligon di sekeliling setiap *kata* pada catatan.

## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

1. Di bilah pencarian atas, cari *Computer Vision*, lalu pilih sumber daya Computer Vision yang Anda buat di lab ini.

1. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan layanan **Azure AI Visual** untuk membaca teks, lihat [dokumentasi Azure AI Visual](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
