# QRcodematching
实验4
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>双二维码验证系统</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            box-sizing: border-box;
        }

        #camera-container {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            position: relative;
            border: 2px solid #333;
            border-radius: 10px;
            overflow: hidden;
        }

        #video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        #status {
            margin: 20px 0;
            font-size: 1.2em;
            text-align: center;
        }

        #result {
            padding: 15px;
            background: #f0f0f0;
            border-radius: 8px;
            margin: 10px 0;
            width: 100%;
            max-width: 600px;
            word-break: break-all;
        }

        #confirm-btn {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            padding: 15px 30px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 25px;
            font-size: 1.1em;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            z-index: 100;
        }

        #confirm-btn:disabled {
            background: #cccccc;
            cursor: not-allowed;
        }

        .hidden {
            display: none !important;
        }
    </style>
</head>
<body>
    <div id="camera-container">
        <video id="video" playsinline></video>
    </div>
    <div id="status">请扫描第一个二维码</div>
    <div id="result" class="hidden"></div>
    <button id="confirm-btn" class="hidden" onclick="startSecondScan()">确认并扫描第二个二维码</button>

    <script>
        let video = document.getElementById('video');
        let resultDiv = document.getElementById('result');
        let statusDiv = document.getElementById('status');
        let confirmBtn = document.getElementById('confirm-btn');
        let firstCode = null;
        let secondCode = null;
        let scanning = false;

        // 初始化摄像头
        async function initCamera() {
            try {
                let stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "environment" } 
                });
                video.srcObject = stream;
                await video.play();
                requestAnimationFrame(tick);
            } catch (e) {
                alert('无法访问摄像头，请确保已授予摄像头权限');
            }
        }

        // 二维码扫描处理
        function tick() {
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                let canvas = document.createElement('canvas');
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
                let ctx = canvas.getContext('2d');
                ctx.drawImage(video, 0, 0);
                
                let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                let code = jsQR(imageData.data, imageData.width, imageData.height);

                if (code) {
                    if (!firstCode) {
                        handleFirstCode(code.data);
                    } else if (!secondCode) {
                        handleSecondCode(code.data);
                    }
                }
            }
            
            if (scanning) {
                requestAnimationFrame(tick);
            }
        }

        function handleFirstCode(data) {
            firstCode = data;
            scanning = false;
            video.classList.add('hidden');
            resultDiv.textContent = `第一个二维码内容：${data}`;
            resultDiv.classList.remove('hidden');
            statusDiv.textContent = "请确认第一个二维码内容";
            confirmBtn.classList.remove('hidden');
        }

        function startSecondScan() {
            confirmBtn.classList.add('hidden');
            resultDiv.classList.add('hidden');
            video.classList.remove('hidden');
            statusDiv.textContent = "请扫描第二个二维码";
            scanning = true;
            requestAnimationFrame(tick);
        }

        function handleSecondCode(data) {
            secondCode = data;
            scanning = false;
            video.classList.add('hidden');
            
            let result = {firstCode} === {secondCode} ? "匹配成功" : "匹配失败";
            resultDiv.innerHTML = `
                <p>第一个二维码：${firstCode}</p>
                <p>第二个二维码：${secondCode}</p>
                <h2>${result}</h2>
            `;
            resultDiv.classList.remove('hidden');
            statusDiv.textContent = "扫描结果";
        }
            
          
    </script>
</body>
</html>
