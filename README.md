# QRcodematching
QRcode matching 
<!DOCTYPE html>
<html>
<head>
    <title>两步验证二维码对比</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    <style>
        body { font-family: Arial; text-align: center; max-width: 400px; margin: 0 auto; }
        video { width: 90%; border: 2px solid #007bff; border-radius: 8px; }
        button { 
            padding: 12px 25px; 
            margin: 10px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 25px;
            font-size: 16px;
            cursor: pointer;
            display: none; /* 默认隐藏 */
        }
        .status { color: #666; margin: 15px; font-size: 18px; }
        .result { 
            padding: 20px;
            margin: 20px;
            border-radius: 10px;
            font-size: 16px;
        }
        .success { background: #e8f5e9; color: #2e7d32; }
        .error { background: #ffebee; color: #c62828; }
    </style>
</head>
<body>
    <h2>二维码两步验证</h2>
    <video id="video" playsinline></video>
    <div class="status" id="status">请扫描第一个二维码</div>
    <button id="confirmBtn" onclick="confirmFirstCode()">确认第一个二维码</button>
    <div id="result"></div>

<script>
const video = document.getElementById('video');
const statusDiv = document.getElementById('status');
const confirmBtn = document.getElementById('confirmBtn');
const resultDiv = document.getElementById('result');
let qrData = { first: null, second: null };
let scannerActive = true;

// 摄像头启动
async function initCamera() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: { facingMode: "environment" }
        });
        video.srcObject = stream;
        video.play();
        scanFrame();
    } catch (error) {
        statusDiv.innerHTML = `摄像头错误：${error.message}`;
    }
}

// 视频帧扫描
function scanFrame() {
    if (!scannerActive) return;
    
    const canvas = document.createElement('canvas');
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const code = jsQR(imageData.data, imageData.width, imageData.height);

    if (code) {
        handleQRScan(code.data);
    }
    requestAnimationFrame(scanFrame);
}

// 处理扫描结果
function handleQRScan(data) {
    if (!qrData.first) {
        qrData.first = data;
        scannerActive = false; // 暂停扫描
        statusDiv.innerHTML = "第一个二维码已扫描！";
        confirmBtn.style.display = 'inline-block'; // 显示确认按钮
        resultDiv.innerHTML = `<div class="result">第一个内容：<br>${data}</div>`;
    } else if (!qrData.second) {
        qrData.second = data;
        scannerActive = false;
        compareResults();
    }
}

// 确认第一个二维码
function confirmFirstCode() {
    confirmBtn.style.display = 'none';
    scannerActive = true; // 恢复扫描
    statusDiv.innerHTML = "请扫描第二个二维码";
    resultDiv.innerHTML += '<div class="result">等待第二个二维码...</div>';
}

// 结果对比
function compareResults() {
    const isMatch = qrData.first === qrData.second;
    video.srcObject.getTracks().forEach(track => track.stop());
    
    resultDiv.innerHTML = `
        <div class="result ${isMatch ? 'success' : 'error'}">
            第一个内容：${qrData.first}<br><br>
            第二个内容：${qrData.second}<br><br>
            <strong>对比结果：${isMatch ? '✅ 完全一致' : '❌ 不一致'}</strong>
        </div>`;
    statusDiv.innerHTML = "验证完成";
}

// 初始化
initCamera();
</script>
</body>
</html>
