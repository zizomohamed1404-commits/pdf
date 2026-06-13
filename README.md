<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>أداة ضغط ملفات PDF</title>
    <style>
        body { 
            font-family: 'Segoe UI', Tahoma, sans-serif; 
            background: linear-gradient(135deg, #1f4037 0%, #99f2c8 100%); 
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 0;
            color: #333;
        }
        .container { 
            background: rgba(255, 255, 255, 0.95); 
            padding: 40px; 
            border-radius: 20px; 
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            text-align: center;
            width: 90%;
            max-width: 500px;
        }
        h2 { color: #1f4037; margin-bottom: 20px; }
        input[type="file"] {
            display: block;
            margin: 20px auto;
            padding: 10px;
            border: 2px dashed #1f4037;
            border-radius: 10px;
            width: 80%;
            cursor: pointer;
        }
        button {
            background-color: #1f4037;
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
            display: none;
            margin: 20px auto 0 auto;
        }
        button:hover { background-color: #142a24; transform: translateY(-2px); }
        #status { margin-top: 20px; font-weight: bold; color: #555; white-space: pre-line; }
    </style>
</head>
<body>

<div class="container">
    <h2>أداة ضغط ملفات PDF المباشرة</h2>
    <p>اختر ملف PDF من جهازك ليتم تقليل حجمه فوراً</p>
    
    <input type="file" id="uploadPDF" accept=".pdf">
    <p id="status"></p>

    <button id="downloadBtn" onclick="downloadCompressedPDF()">تحميل الـ PDF المضغوط</button>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<script>
    // إعداد مكتبة قراءة الـ PDF الخلفية
    pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

    const uploadInput = document.getElementById('uploadPDF');
    const statusText = document.getElementById('status');
    const downloadBtn = document.getElementById('downloadBtn');
    let finalPdfDoc = null;

    uploadInput.addEventListener('change', async (e) => {
        const file = e.target.files[0];
        if (!file) return;

        statusText.style.color = "#007bff";
        statusText.innerText = "جاري تفكيك ملف الـ PDF وضغطه... ⏳\n(برجاء الانتظار وعدم إغلاق الصفحة)";
        downloadBtn.style.display = "none";

        const fileReader = new FileReader();
        fileReader.onload = async function() {
            const typedarray = new Uint8Array(this.result);
            try {
                // فتح ملف الـ PDF الأصلي
                const pdf = await pdfjsLib.getDocument(typedarray).promise;
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF('p', 'mm', 'a4');
                const pdfWidth = doc.internal.pageSize.getWidth();
                const pdfHeight = doc.internal.pageSize.getHeight();

                // المرور على كل صفحة وضغطها
                for (let pageNum = 1; pageNum <= pdf.numPages; pageNum++) {
                    statusText.innerText = `جاري معالجة وضغط صفحة ${pageNum} من أصل ${pdf.numPages}... ⏳`;
                    
                    const page = await pdf.getPage(pageNum);
                    const viewport = page.getViewport({ scale: 1.2 }); // تحجيم مناسب متوازن بين الحجم والجودة
                    
                    const canvas = document.createElement('canvas');
                    const context = canvas.getContext('2d');
                    canvas.height = viewport.height;
                    canvas.width = viewport.width;

                    // رسم الصفحة على الـ Canvas
                    await page.render({ canvasContext: context, viewport: viewport }).promise;
                    
                    // تحويل الصفحة لصورة JPEG وضغط جودتها لـ 40% لتقليل المساحة تماماً
                    const imgData = canvas.toDataURL('image/jpeg', 0.4);

                    if (pageNum > 1) doc.addPage();
                    doc.addImage(imgData, 'JPEG', 0, 0, pdfWidth, pdfHeight);
                }

                finalPdfDoc = doc;
                const originalSize = (file.size / 1024 / 1024).toFixed(2);
                
                statusText.style.color = "#28a745";
                statusText.innerText = `✅ تم ضغط الملف بنجاح!\nالحجم الأصلي: ${originalSize} MB\nجاهز للتحميل الآن بجودة اقتصادية.`;
                downloadBtn.style.display = "block";

            } catch (error) {
                console.error(error);
                statusText.style.color = "#dc3545";
                statusText.innerText = "❌ حدث خطأ أثناء ضغط الملف. تأكد أن الملف غير محمي بكلمة مرور.";
            }
        };
        fileReader.readAsArrayBuffer(file);
    });

    // دالة التحميل الفعلي للملف الجديد
    function downloadCompressedPDF() {
        if (finalPdfDoc) {
            finalPdfDoc.save("Compressed_Document.pdf");
        }
    }
</script>

</body>
</html>
