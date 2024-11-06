<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Tasks - Amazing Style</title>
    <style>
        /* Styles for Amazing Style Design */
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(to right, #141e30, #243b55);
            color: #fff;
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        .nav-button {
            background: linear-gradient(135deg, #ff6a00, #ee0979);
            color: #fff;
            padding: 12px 24px;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            font-weight: bold;
            font-size: 16px;
            margin: 16px;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .nav-button:hover {
            transform: scale(1.1);
            box-shadow: 0 4px 12px rgba(238, 9, 121, 0.5);
        }
        #navContent {
            display: none;
            background: linear-gradient(135deg, #243b55, #141e30);
            padding: 24px;
            border-radius: 16px;
            margin: 16px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
        }
        .video-upload {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 16px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            padding: 16px;
        }
        .video-upload input[type="text"],
        .video-upload input[type="file"],
        .video-upload button {
            padding: 12px 20px;
            border-radius: 30px;
            border: none;
            outline: none;
            color: #333;
            font-weight: bold;
            transition: all 0.3s ease;
        }
        .video-upload input[type="text"] {
            background: #fff;
            color: #333;
            font-size: 14px;
            width: 180px;
        }
        .video-upload input[type="file"] {
            background: #fff;
            color: #333;
            cursor: pointer;
        }
        .video-upload button {
            background: linear-gradient(135deg, #ff6a00, #ee0979);
            color: #fff;
            cursor: pointer;
        }
        .video-upload button:hover {
            transform: scale(1.05);
            box-shadow: 0 4px 12px rgba(238, 9, 121, 0.5);
        }
        .video-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
            gap: 16px;
            padding: 16px;
        }
        .video-task {
            border-radius: 8px;
            overflow: hidden;
            background: #fff;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            padding: 16px;
            text-align: center;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .video-task:hover {
            transform: scale(1.05);
            box-shadow: 0 5px 5px rgba(0, 0, 0, 0.3);
        }
        .video-container {
            position: relative;
            width: 100%;
            padding-top: 56.25%; /* 16:9 Aspect Ratio */
            overflow: hidden;
            border-radius: 8px;
            background-color: #000;
        }
        video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }
        .video-title {
            font-size: 15px;
            font-weight: bold;
            margin-top: 10px;
            color: #333;
        }
    </style>
</head>
<body>
    <button class="nav-button" onclick="toggleNav()">Open Upload Section</button>

    <nav id="navContent">
        <div class="video-upload">
            <input type="text" id="codeInput" placeholder="Enter Code">
            <input type="file" id="videoFileInput" accept="video/*" multiple disabled>
            <button onclick="checkCode()">Submit Code</button>
            <button onclick="loadVideos()" disabled id="loadButton">Load Videos</button>
        </div>
    </nav>

    <!-- Video Grid outside the nav section -->
    <div id="videoGrid" class="video-grid"></div>

    <script>
        function toggleNav() {
            const navContent = document.getElementById("navContent");
            navContent.style.display = navContent.style.display === "none" ? "block" : "none";
        }

        let db;

        const request = indexedDB.open('VideoDB', 1);

        request.onupgradeneeded = function(event) {
            db = event.target.result;
            db.createObjectStore('videoFiles', { keyPath: 'id' });
        };

        request.onsuccess = function(event) {
            db = event.target.result;
            loadStoredVideos();
        };

        function checkCode() {
            const code = document.getElementById('codeInput').value;
            const videoInput = document.getElementById('videoFileInput');
            const loadButton = document.getElementById('loadButton');

            const storedCode = localStorage.getItem('videoUploadCode');

            if (storedCode === code) {
                videoInput.disabled = false;
                loadButton.disabled = false;
                alert("Code accepted! You can now upload video files.");
            } else if (code === '112233') {
                localStorage.setItem('videoUploadCode', code);
                videoInput.disabled = false;
                loadButton.disabled = false;
                alert("Code accepted! You can now upload video files.");
            } else {
                alert("Invalid code. Please enter the correct code.");
                videoInput.disabled = true;
                loadButton.disabled = true;
            }
        }

        function loadVideos() {
            const videoInput = document.getElementById('videoFileInput');
            if (videoInput.files.length > 0) {
                Array.from(videoInput.files).forEach(file => {
                    createVideoTask(file);
                    saveVideoToDB(file);
                });
            } else {
                alert("Please select video files to play.");
            }
        }

        function createVideoTask(file) {
            const videoGrid = document.getElementById('videoGrid');

            const videoTask = document.createElement('div');
            videoTask.className = 'video-task';

            const videoContainer = document.createElement('div');
            videoContainer.className = 'video-container';

            const videoPlayer = document.createElement('video');
            videoPlayer.controls = true;
            videoPlayer.src = URL.createObjectURL(file);

            videoContainer.appendChild(videoPlayer);

            const videoTitle = document.createElement('p');
            videoTitle.className = 'video-title';
            videoTitle.textContent = file.name;

            videoTask.appendChild(videoContainer);
            videoTask.appendChild(videoTitle); // Title is now below the video

            videoGrid.insertBefore(videoTask, videoGrid.firstChild);
        }

        function saveVideoToDB(file) {
            const transaction = db.transaction(['videoFiles'], 'readwrite');
            const store = transaction.objectStore('videoFiles');

            const videoData = {
                id: file.name,
                file: file
            };

            const request = store.put(videoData);

            request.onsuccess = function() {
                console.log("Video file saved to IndexedDB");
            };

            request.onerror = function() {
                console.error("Error saving video file to IndexedDB");
            };
        }

        function loadStoredVideos() {
            const transaction = db.transaction(['videoFiles'], 'readonly');
            const store = transaction.objectStore('videoFiles');
            const request = store.getAll();

            request.onsuccess = function(event) {
                const videoFiles = event.target.result.reverse();
                videoFiles.forEach(videoData => {
                    const file = new File([videoData.file], videoData.id);
                    createVideoTask(file);
                });
            };
        }
    </script>
</body>
</html>
