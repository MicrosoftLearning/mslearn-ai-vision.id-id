---
lab:
  title: Membaca Teks dalam Gambar
  module: Module 11 - Reading Text in Images and Documents
---

# Membaca Teks dalam Gambar

Pengenalan karakter optik (OCR) adalah bagian dari visi komputer yang berhubungan dengan membaca teks dalam gambar dan dokumen. Layanan **Azure AI Visual** menyediakan API untuk membaca teks, yang akan Anda jelajahi dalam latihan ini.

## Mengkloning repositori untuk kursus ini

Jika Anda belum melakukannya, Anda harus mengkloning repositori kode untuk kursus ini:

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-vision` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.
4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**. Jika Anda menerima Pesan *Terdeteksi Proyek Azure Function di folder*, Anda dapat menutup pesan tersebut dengan aman.

## Menyediakan sumber daya Layanan Azure AI

Jika belum memilikinya di langganan, Anda harus menyediakan sumber daya **Layanan Azure AI**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Pada bilah pencarian di bagian atas, cari *layanan Azure AI*, pilih **Layanan Azure AI**, lalu buat sumber daya akun multi-layanan layanan Azure AI dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Wilayah**: *Pilih dari Asia Tenggara, Eropa Barat, Eropa Utara, Korea Tengah, Prancis Tengah, US Barat, US Timur, atau Asia Timur\**
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Standar S0

    \*Fitur Azure AI Visual 4.0 saat ini hanya tersedia di wilayah ini.

3. Pilih kotak centang yang diperlukan dan buat sumber daya.
4. Tunggu hingga penyebaran selesai, lalu lihat detail penyebaran.
5. Saat sumber daya telah diterapkan, buka dan lihat halaman **Kunci dan Titik Akhir**. Anda akan memerlukan titik akhir dan salah satu kunci dari halaman ini dalam prosedur selanjutnya.

## Bersiap untuk menggunakan Azure AI Visual SDK

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk membaca teks.

> **Catatan**: Anda dapat memilih untuk menggunakan SDK baik untuk **C#** atau **Python**. Pada langkah-langkah berikut, lakukan tindakan yang sesuai dengan bahasa pilihan Anda.

1. Di Visual Studio Code, di panel **Explorer**, telusuri folder **Labfiles\05-ocr** dan luaskan folder **C-Sharp** atau **Python**, tergantung pada preferensi bahasa Anda.
2. Klik kanan folder **read-text** dan buka terminal terintegrasi. Kemudian, instal paket Azure AI Visual SDK dengan menjalankan perintah yang sesuai dengan preferensi bahasa Anda:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Catatan**: Jika diminta untuk menginstal ekstensi kit pengembangan, Anda dapat menutup pesan tersebut dengan aman.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. Lihat konten folder **read-text**, dan perhatikan bahwa folder tersebut berisi file untuk pengaturan konfigurasi:

    - **C#**: appsettings.json
    - **Python**: .env

    Buka file konfigurasi dan perbarui nilai konfigurasi di dalamnya agar mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya layanan Azure AI Anda. Simpan perubahan Anda.


## Menggunakan Azure AI Visual SDK untuk membaca teks dari gambar

Salah satu fitur **Azure AI Visual SDK** adalah membaca teks dari gambar. Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk membaca teks dari gambar.

1. Folder **read-text** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan yang Anda perlukan untuk menggunakan Azure AI Visual SDK:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. Dalam file kode untuk aplikasi klien Anda, dalam fungsi **Main**, kode untuk memuat pengaturan konfigurasi telah disediakan. Kemudian, temukan komentar **Mengautentikasi klien Azure AI Visual**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk membuat dan mengautentikasi objek klien Azure AI Visual:

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

3. Dalam fungsi **Main**, di bawah kode yang baru saja Anda tambahkan, perhatikan bahwa kode tersebut menentukan jalur ke file gambar dan kemudian meneruskan jalur gambar ke fungsi **GetTextRead**. Fungsi ini belum sepenuhnya diterapkan.

4. Mari menambahkan beberapa kode ke isi fungsi **GetTextRead**. Temukan komentar **Menggunakan fungsi Analisis gambar untuk membaca teks dalam gambar**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut, perhatikan bahwa fitur visual ditentukan saat memanggil fungsi `Analyze`:

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
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
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

5. Dalam kode yang baru saja Anda tambahkan dalam fungsi **GetTextRead**, dan di bawah komentar **Menghasilkan teks yang terdeteksi dalam gambar**, tambahkan kode berikut (kode ini mencetak teks gambar ke konsol dan menghasilkan gambar **text.jpg** yang menyoroti teks gambar):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

6. Dalam folder **read-text/images**, pilih **Lincoln.jpg** untuk menampilkan file yang diproses kode Anda.

7. Dalam file kode untuk aplikasi Anda, dalam fungsi **Utama**, periksa kode yang berjalan jika pengguna memilih opsi menu **1**. Kode ini memanggil fungsi **GetTextRead**, meneruskan jalur ke file gambar *Lincoln.jpg*.

8. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **baca-teks**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. Saat diminta, masukkan **1** dan amati hasilnya, yaitu teks yang diekstrak dari gambar.

10. Dalam folder **read-text**, pilih gambar **text.jpg** dan perhatikan keberadaan poligon di sekeliling setiap *baris* teks.

11. Kembali ke file kode di Visual Studio Code, dan temukan komentar **Menghasilkan kotak pembatas posisi di sekeliling setiap baris**. Kemudian, di bawah komentar ini, tambahkan kode berikut:

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

12. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **baca-teks**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. Ketika diminta, masukkan **1** dan amati output yang dihasilkan. Output-nya harus terdiri dari setiap baris teks dalam gambar, bersama dengan posisinya masing-masing dalam gambar.


14. Kembali ke file kode di Visual Studio Code, dan temukan komentar **Menghasilkan setiap kata yang terdeteksi dalam gambar dan kotak pembatas posisi di sekeliling setiap kata dengan tingkat keyakinan untuk setiap kata**. Kemudian, di bawah komentar ini, tambahkan kode berikut:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

15. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **baca-teks**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. Ketika diminta, masukkan **1** dan amati output yang dihasilkan. Output-nya harus terdiri dari setiap kata dalam teks pada gambar, bersama dengan posisinya masing-masing dalam gambar. Perhatikan bahwa informasi yang dihasilkan juga mencakup tingkat keyakinan untuk setiap kata.

17. Dalam folder **read-text**, pilih gambar **text.jpg** dan perhatikan keberadaan poligon di sekeliling setiap *kata*.

## Menggunakan Azure AI Visual SDK untuk membaca teks tulisan tangan dari gambar

Dalam latihan sebelumnya, Anda membaca teks yang terdefinisi dengan baik dari gambar. Namun, terkadang Anda mungkin juga ingin membaca teks dari catatan atau naskah tulisan tangan. Kabar baiknya adalah **Azure AI Visual SDK** juga dapat membaca teks tulisan tangan dengan kode yang sama persis dengan yang Anda gunakan untuk membaca teks yang terdefinisi dengan baik. Kita akan menggunakan kode yang sama dari latihan sebelumnya, tetapi kali ini kita akan menggunakan gambar yang berbeda.

1. Dalam folder **read-text/images**, pilih **Note.jpg** untuk menampilkan file yang diproses kode Anda.

2. Dalam file kode untuk aplikasi Anda, dalam fungsi **Utama**, periksa kode yang berjalan jika pengguna memilih opsi menu **2**. Kode ini memanggil fungsi **GetTextRead**, meneruskan jalur ke file gambar *Note.jpg*.

3. Dari terminal terintegrasi untuk folder **read-text**, masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. Ketika diminta, masukkan **2** dan amati output yang dihasilkan. Output-nya berupa teks yang diekstrak dari gambar catatan.

5. Dalam folder **read-text**, pilih gambar **text.jpg** dan perhatikan keberadaan poligon di sekeliling setiap *kata* dalam catatan.

## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

2. Pada bilah pencarian di bagian atas, cari *akun multi-layanan layanan Azure AI*, dan pilih sumber daya akun multi-layanan layanan Azure AI yang Anda buat di lab ini

3. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Untuk informasi selengkapnya tentang menggunakan layanan **Azure AI Visual** untuk membaca teks, lihat [dokumentasi Azure AI Visual](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
