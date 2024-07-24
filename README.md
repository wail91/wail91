<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>موقع رفع الصور</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
            color: #333;
            text-align: center;
        }
        header {
            background-color: #333;
            color: #fff;
            padding: 1em 0;
        }
        .container {
            width: 80%;
            margin: 0 auto;
            padding: 2em 0;
        }
        .upload-section, .gallery {
            background-color: #fff;
            padding: 2em;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            margin-bottom: 2em;
        }
        .upload-section input[type="file"] {
            margin-bottom: 1em;
        }
        .upload-section img, .gallery img {
            max-width: 100%;
            height: auto;
            margin-top: 1em;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        footer {
            background-color: #333;
            color: #fff;
            padding: 1em 0;
            position: fixed;
            width: 100%;
            bottom: 0;
        }
        .gallery img {
            display: inline-block;
            margin: 10px;
        }
    </style>
</head>
<body>
    <header>
        <h1>موقع رفع الصور</h1>
    </header>

    <div class="container">
        <div class="upload-section">
            <h2>رفع صورة</h2>
            <input type="file" id="fileInput" accept="image/*">
            <img id="preview" src="#" alt="معاينة الصورة" style="display:none;">
            <button id="saveButton" style="display:none;">حفظ الصورة</button>
        </div>

        <div class="gallery">
            <h2>معرض الصور</h2>
            <div id="imageGallery"></div>
        </div>
    </div>

    <footer>
        <p>&copy; 2024 موقع رفع الصور. جميع الحقوق محفوظة.</p>
    </footer>

    <script>
        let db;

        function openDatabase() {
            const request = indexedDB.open('imageDatabase', 1);
            
            request.onupgradeneeded = function(event) {
                db = event.target.result;
                db.createObjectStore('images', { keyPath: 'id', autoIncrement: true });
            };

            request.onsuccess = function(event) {
                db = event.target.result;
                loadImages();
            };

            request.onerror = function(event) {
                console.error('Database error:', event.target.errorCode);
            };
        }

        function saveImage(blob) {
            const transaction = db.transaction(['images'], 'readwrite');
            const objectStore = transaction.objectStore('images');
            const image = {
                data: blob,
                timestamp: new Date()
            };
            objectStore.add(image);

            transaction.oncomplete = function() {
                alert('تم حفظ الصورة بنجاح!');
                loadImages(); // إعادة تحميل الصور بعد حفظ صورة جديدة
            };

            transaction.onerror = function(event) {
                console.error('Error saving image:', event.target.errorCode);
            };
        }

        function loadImages() {
            const transaction = db.transaction(['images'], 'readonly');
            const objectStore = transaction.objectStore('images');
            const request = objectStore.getAll();

            request.onsuccess = function(event) {
                const images = event.target.result;
                const gallery = document.getElementById('imageGallery');
                gallery.innerHTML = ''; // مسح المعرض الحالي
                images.forEach(image => {
                    const img = document.createElement('img');
                    img.src = URL.createObjectURL(image.data);
                    gallery.appendChild(img);
                });
            };

            request.onerror = function(event) {
                console.error('Error loading images:', event.target.errorCode);
            };
        }

        document.getElementById('fileInput').addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = document.getElementById('preview');
                    img.src = e.target.result;
                    img.style.display = 'block';
                    document.getElementById('saveButton').style.display = 'inline';
                }
                reader.readAsDataURL(file);
            }
        });

        document.getElementById('saveButton').addEventListener('click', function() {
            const fileInput = document.getElementById('fileInput');
            const file = fileInput.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const arrayBuffer = e.target.result;
                    const blob = new Blob([arrayBuffer], { type: file.type });
                    saveImage(blob);
                };
                reader.readAsArrayBuffer(file);
            }
        });

        openDatabase();
    </script>
</body>
</html>
