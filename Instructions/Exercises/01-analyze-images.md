---
lab:
  title: Menganalisis Gambar dengan Azure AI Visual
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Menganalisis Gambar dengan Azure AI Visual

Azure AI Visual adalah kemampuan kecerdasan buatan yang memungkinkan sistem perangkat lunak menginterpretasikan input visual dengan menganalisis gambar. Di Microsoft Azure, layanan Azure AI **Vision** menyediakan model bawaan untuk tugas visi komputer umum, termasuk analisis gambar untuk menyarankan keterangan dan tag, deteksi objek dan orang umum. Anda juga dapat menggunakan layanan Azure AI Visual untuk menghapus latar belakang atau membuat pemotongan gambar latar depan.

## Mengkloning repositori untuk kursus ini

Jika Anda belum mengkloning repositori kode **Azure AI Visual** ke lingkungan tempat Anda mengerjakan lab ini, ikuti langkah-langkah berikut untuk melakukannya. Jika tidak, buka folder kloning di Visual Studio Code.

1. Memulai Visual Studio Code.
2. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-ai-vision` ke folder lokal (tidak masalah folder mana).
3. Setelah repositori dikloning, buka folder di Visual Studio Code.
4. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**. Jika Anda menerima Pesan *Terdeteksi Proyek Azure Function di folder*, Anda dapat menutup pesan tersebut dengan aman.

## Menyediakan sumber daya Layanan Azure AI

Jika Belum memilikinya di langganan, Anda harus menyediakan sumber daya **Layanan Azure AI**.

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Di bilah pencarian teratas, cari *layanan Azure AI*, pilih **Layanan Azure AI**, dan buat sumber daya akun multi-layanan layanan Azure AI dengan pengaturan berikut:
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

Dalam latihan ini, Anda akan menyelesaikan aplikasi klien yang diimplementasikan sebagian yang menggunakan Azure AI Visual SDK untuk menganalisis gambar.

> **Catatan**: Anda dapat memilih untuk menggunakan SDK baik untuk **C#** atau **Python**. Dalam langkah-langkah di bawah ini, lakukan tindakan yang sesuai untuk bahasa pilihan Anda.

1. Di Visual Studio Code, di panel **Explorer**, telusuri folder **Labfiles/01-analyze-images** dan perluas folder **C-Sharp** atau **Python** tergantung pada preferensi bahasa Anda.
2. Klik kanan folder **image-analysis** dan buka terminal terintegrasi. Kemudian instal paket Azure AI Visual SDK dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Catatan**: Jika diminta untuk menginstal ekstensi kit pengembangan, Anda dapat menutup pesan tersebut dengan aman.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```
    
3. Lihat konten folder **image-analysis**, dan perhatikan bahwa folder tersebut berisi file untuk pengaturan konfigurasi:
    - **C#**: appsettings.json
    - **Python**: .env

    Buka file konfigurasi dan perbarui nilai konfigurasi di dalamnya agar mencerminkan **titik akhir** dan **kunci** autentikasi untuk sumber daya layanan Azure AI Anda. Simpan perubahan Anda.
4. Perhatikan bahwa folder **image-analysis** berisi file kode untuk aplikasi klien:

    - **C#**: Program.cs
    - **Python**: image-analysis.py

    Buka file kode dan di bagian atas, di bawah referensi namespace yang ada, temukan komentar **Impor namespace**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk mengimpor namespace layanan, Anda harus menggunakan Azure AI Visual SDK:

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
    
## Lihat gambar yang akan Anda analisis

Dalam latihan ini, Anda akan menggunakan layanan Azure AI Visual untuk menganalisis beberapa gambar.

1. Di Visual Studio Code, luaskan folder **image-analysis** dan folder **gambar** yang ada di dalamnya.
2. Pilih masing-masing file gambar secara bergantian untuk melihatnya dalam Visual Studio Code.

## Analisis gambar untuk menyarankan keterangan

Sekarang Anda siap menggunakan SDK untuk memanggil layanan Visi dan menganalisis gambar.

1. Dalam file kode untuk aplikasi klien Anda (**Program.cs** atau **image-analysis.py**), dalam fungsi **Utama**, perhatikan bahwa kode untuk memuat pengaturan konfigurasi telah disediakan. Kemudian temukan komentar **Autentikasi klien Azure AI Visual**. Kemudian, di bawah komentar ini, tambahkan kode khusus bahasa berikut untuk membuat dan mengautentikasi objek klien Azure AI Visual:

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

2. Dalam fungsi **Utama**, di bawah kode yang baru saja Anda tambahkan, perhatikan bahwa kode menentukan jalur ke file gambar lalu meneruskan jalur gambar ke dua fungsi lain (**AnalyzeImage** dan **BackgroundForeground**). Fungsi-fungsi ini belum sepenuhnya dilaksanakan.

3. Dalam fungsi **AnalyzeImage**, di bawah komentar **Dapatkan hasil dengan menentukan fitur yang akan diambil**, tambahkan kode berikut:

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
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
    
4. Dalam fungsi **AnalyzeImage**, di bawah komentar **Tampilkan hasil analisis**, tambahkan kode berikut (termasuk komentar yang menunjukkan di mana Anda akan menambahkan lebih banyak kode nanti.):

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. Simpan perubahan Anda dan kembali ke terminal terintegrasi untuk folder **image-analysis**, dan masukkan perintah berikut untuk menjalankan program dengan argumen **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Amati output, yang harus menyertakan keterangan yang disarankan untuk gambar **street.jpg**.
7. Jalankan program lagi, kali ini dengan argumen **images/building.jpg** untuk melihat keterangan yang dihasilkan untuk gambar **building.jpg**.
8. Ulangi langkah sebelumnya untuk membuat keterangan untuk file **images/person.jpg**.

## Dapatkan tag yang disarankan untuk sebuah gambar

Terkadang berguna untuk mengidentifikasi *tag* relevan yang memberikan petunjuk tentang konten gambar.

1. Dalam fungsi **AnalyzeImage**, di bawah komentar **Dapatkan tag gambar**, tambahkan kode berikut:

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Simpan perubahan Anda dan jalankan program sekali untuk setiap file gambar dalam folder **gambar**, perhatikan bahwa selain keterangan gambar, daftar tag yang disarankan akan ditampilkan.

## Mendeteksi dan menemukan objek dalam gambar

*Deteksi objek* adalah bentuk visi komputer tertentu di mana objek individual dalam gambar diidentifikasi dan lokasinya ditunjukkan oleh kotak pembatas..

1. Dalam fungsi **AnalyzeImage**, di bawah komentar **Dapatkan objek dalam gambar**, tambahkan kode berikut:

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. Simpan perubahan Anda dan jalankan program sekali untuk setiap file gambar di folder **gambar**, mengamati objek apa pun yang terdeteksi. Setelah setiap kali dijalankan, lihat file **objects.jpg** yang dibuat dalam folder yang sama dengan file kode Anda untuk melihat objek yang dianotasi.

## Mendeteksi dan menemukan orang dalam gambar

*Deteksi orang* adalah bentuk spesifik visi komputer di mana individu orang dalam gambar diidentifikasi dan lokasinya ditunjukkan oleh kotak pembatas.

1. Dalam fungsi **AnalyzeImage**, di bawah komentar **Dapatkan orang dalam gambar**, tambahkan kode berikut:

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (Opsional) Batalkan komentar perintah **Console.Writeline** di bawah bagian **Mengembalikan keyakinan orang yang terdeteksi** untuk meninjau tingkat keyakinan yang mengembalikan bahwa seseorang terdeteksi pada posisi gambar tertentu.

3. Simpan perubahan Anda dan jalankan program sekali untuk setiap file gambar di folder **gambar**, mengamati objek apa pun yang terdeteksi. Setelah setiap kali dijalankan, lihat file **objects.jpg** yang dibuat dalam folder yang sama dengan file kode Anda untuk melihat objek yang dianotasi.

> **Catatan**: Dalam tugas sebelumnya, Anda menggunakan satu metode untuk menganalisis gambar, lalu menambahkan kode secara bertahap untuk mengurai dan menampilkan hasilnya. SDK juga menyediakan metode individual untuk menyarankan keterangan, mengidentifikasi tag, mendeteksi objek, dan sebagainya - artinya Anda dapat menggunakan metode yang paling tepat untuk mengembalikan hanya informasi yang Anda butuhkan, mengurangi ukuran muatan data yang perlu dikembalikan. Lihat dokumentasi [.NET SDK](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) atau [dokumentasi Python SDK](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) untuk detail selengkapnya.

## Menghapus latar belakang atau menghasilkan potongan latar depan gambar

Dalam beberapa kasus, Anda mungkin perlu membuat hapus latar belakang gambar atau mungkin ingin membuat potongan latar depan gambar tersebut. Mari kita mulai dengan penghapusan latar belakang.

1. Dalam file kode Anda, temukan fungsi ** BackgroundForeground**; dan di bawah komentar **Hapus latar belakang dari gambar atau hasilkan potongan latar depan**, tambahkan kode berikut:

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("background.png", "wb") as file:
    file.write(image)
print('  Results saved in background.png \n')
```
    
2. Simpan perubahan Anda dan jalankan program sekali untuk setiap file gambar di folder **gambar**, membuka file **background.png** yang dihasilkan di folder yang sama dengan file kode Anda untuk setiap gambar.  Perhatikan bagaimana latar belakang telah dihapus dari setiap gambar.

Sekarang mari kita hasilkan potongan latar depan untuk gambar kita.

3. Dalam file kode Anda, temukan fungsi **BackgroundForeground**; dan di bawah komentar **Tentukan versi API dan mode**, ubah variabel mode menjadi `foregroundMatting`.

4. Simpan perubahan Anda dan jalankan program sekali untuk setiap file gambar di folder **gambar**, membuka file **background.png** yang dihasilkan di folder yang sama dengan file kode Anda untuk setiap gambar.  Perhatikan bagaimana potongan latar depan telah dihasilkan untuk gambar Anda.

## Membersihkan sumber daya

Jika Anda tidak menggunakan sumber daya Azure yang dibuat di lab ini untuk modul pelatihan lainnya, Anda dapat menghapusnya untuk menghindari dikenakan biaya lebih lanjut. Berikut caranya:

1. Buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.

2. Di bilah pencarian teratas, cari *akun multi-layanan layanan Azure AI*, dan pilih sumber daya akun multi-layanan layanan Azure AI yang Anda buat di lab ini.

3. Pada halaman sumber daya, pilih **Hapus** dan ikuti instruksi untuk menghapus sumber daya.

## Informasi selengkapnya

Dalam latihan ini, Anda menjelajahi beberapa analisis gambar dan kemampuan manipulasi layanan Azure AI Visual. Layanan ini juga mencakup kemampuan untuk mendeteksi objek dan orang, dan tugas visi komputer lainnya.

Untuk informasi selengkapnya tentang menggunakan layanan **Azure AI Visual**, lihat [dokumentasi Azure AI Visual](https://learn.microsoft.com/azure/ai-services/computer-vision/).
