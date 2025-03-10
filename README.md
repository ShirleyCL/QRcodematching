# QRcodematching
QRcode matching 
<!DOCTYPE html>
<html>
<head>
    <title>二维码对比扫描器</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        body { text-align: center; font-family: Arial; }
        #video-container { position: relative; margin: 20px auto; }
        video { width: 300px; border: 2px solid #333; }
        .status { color: #666; margin: 10px; }
        .result { padding: 15px; margin: 20px; border-radius: 5px; }
        .success { background: #dfffdf; color: #2a752a; }
        .error { background: #ffe5e5; color: #cc0000; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 5px; cursor: pointer; }
    </style>
</head>
<body>
    <h2>二维码对比扫描器</h2>
    <div id="video-container">
        <video id="video" playsinline></video>
    </div>
    <div id="status" class="status">点击开始扫描第一个二维码</div>
    <button onclick="startNewScan()">重新开始</button>
    <div id="result"></div>

<script>
let video = document.getElementById('video');
let statusDiv = document.getElementById('status');
let resultDiv = document.getElementById('result');
let qrData = { first: null, second: null };
let scanning = false;

// 启动摄像头
async function startCamera() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } });
        video.srcObject = stream;
        video.play();
        requestAnimationFrame(tick);
    } catch (e) {
        statusDiv.innerHTML = "无法访问摄像头: " + e.message;
    }
}

// 视频帧处理
function tick() {
    if (!scanning && video.readyState === video.HAVE_ENOUGH_DATA) {
        scanning = true;
        
        const canvas = document.createElement('canvas');
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
        
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const code = jsQR(imageData.data, imageData.width, imageData.height);

        if (code) {
            handleScannedCode(code.data);
        }
        scanning = false;
    }
    requestAnimationFrame(tick);
}

// 处理扫描结果
function handleScannedCode(data) {
    if (!qrData.first) {
        qrData.first = data;
        statusDiv.innerHTML = "第一个二维码扫描成功！请扫描第二个二维码";
        resultDiv.innerHTML = `<div class="result success">第一个内容：${data}</div>`;
    } else {
        qrData.second = data;
        video.srcObject.getTracks().forEach(track => track.stop());
        compareResults();
    }
}

// 对比结果
function compareResults() {
    const isMatch = qrData.first === qrData.second;
    statusDiv.innerHTML = "扫描完成";
    resultDiv.innerHTML += `
        <div class="result ${isMatch ? 'success' : 'error'}">
            第二个内容：${qrData.second}<br>
            对比结果：${isMatch ? '✅ 内容一致' : '❌ 内容不一致'}
        </div>`;
}

// 重新开始
function startNewScan() {
    qrData = { first: null, second: null };
    resultDiv.innerHTML = '';
    statusDiv.innerHTML = "点击开始扫描第一个二维码";
    startCamera();
}

// 初始化
startCamera();
</script>
</body>
</html>
