<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Chấm Điểm Phát Âm Tiếng Trung</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 450px;
            width: 100%;
        }
        h1 {
            color: #1a73e8;
            font-size: 24px;
            margin-bottom: 20px;
        }
        .phrase-box {
            background-color: #e8f0fe;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        .chinese-text {
            font-size: 32px;
            font-weight: bold;
            color: #333;
            margin: 0;
            letter-spacing: 2px;
        }
        .pinyin-text {
            font-size: 16px;
            color: #666;
            margin: 8px 0 0 0;
        }
        .btn-mic {
            background-color: #1a73e8;
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 16px;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            font-weight: bold;
        }
        .btn-mic:hover {
            background-color: #1557b0;
        }
        .btn-mic:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        .status {
            margin-top: 15px;
            font-style: italic;
            color: #555;
            height: 20px;
        }
        .result-box {
            margin-top: 25px;
            border-top: 2px solid #eee;
            padding-top: 20px;
        }
        .score {
            font-size: 48px;
            font-weight: bold;
            margin: 10px 0;
        }
        .good { color: #2ea44f; }
        .average { color: #f9ab00; }
        .bad { color: #d93025; }
        .btn-next {
            margin-top: 15px; 
            background: none; 
            border: 1px solid #1a73e8; 
            color: #1a73e8; 
            padding: 8px 15px; 
            border-radius: 4px; 
            cursor: pointer;
            font-weight: bold;
        }
        .btn-next:hover {
            background-color: #e8f0fe;
        }
    </style>
</head>
<body>
<H1>Võ Nhựt Kì</H1>

<div class="container">
    <h1>Phát Âm Tiếng Trung AI</h1>
    
    <div class="phrase-box">
        <p class="chinese-text" id="target-text">你好</p>
        <p class="pinyin-text" id="target-pinyin">nǐ hǎo (Xin chào)</p>
    </div>

    <button class="btn-mic" id="btn-start">🎙️ Bấm để nói</button>
    <p class="status" id="status">Sẵn sàng...</p>

    <div class="result-box">
        <p>Máy nghe được: <strong id="user-spoken" style="color: #1a73e8; font-size: 20px;">...</strong></p>
        <p>Điểm số của bạn:</p>
        <div class="score" id="score-display">--</div>
    </div>
    
    <button class="btn-next" onclick="changePhrase()">Đổi câu khác ➡️</button>
</div>

<script>
    // Danh sách các câu mẫu đã được sửa chuẩn tiếng Trung
    const phrases = [
        { text: "你好", pinyin: "nǐ hǎo (Xin chào)" },
        { text: "谢谢", pinyin: "xièxie (Cảm ơn)" },
        { text: "我爱你", pinyin: "wǒ ài nǐ (Tôi yêu bạn)" },
        { text: "中国", pinyin: "Zhōngguó (Trung Quốc)" },
        { text: "我想学汉语", pinyin: "wǒ xiǎng xué hànyǔ (Tôi muốn học tiếng Trung)" }
    ];

    let currentIdx = 0;

    // Khởi tạo Web Speech API
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        document.getElementById('status').innerText = "❌ Trình duyệt không hỗ trợ! Hãy dùng Google Chrome trên máy tính.";
        document.getElementById('btn-start').disabled = true;
    } else {
        const recognition = new SpeechRecognition();
        recognition.lang = 'zh-CN'; // Nhận diện tiếng Trung Phổ Thông
        recognition.interimResults = false;
        recognition.maxAlternatives = 1;

        const btnStart = document.getElementById('btn-start');
        const statusText = document.getElementById('status');
        const userSpokenText = document.getElementById('user-spoken');
        const scoreDisplay = document.getElementById('score-display');
        const targetTextElement = document.getElementById('target-text');

        btnStart.addEventListener('click', () => {
            try {
                recognition.start();
                statusText.innerText = "🎤 Đang nghe... Hãy đọc to câu phía trên!";
                btnStart.disabled = true;
            } catch (e) {
                statusText.innerText = "❌ Hệ thống bận, thử lại sau 1 giây.";
                btnStart.disabled = false;
            }
        });

        recognition.onspeechend = () => {
            recognition.stop();
            statusText.innerText = "🔄 Đang phân tích giọng nói...";
            btnStart.disabled = false;
        };

        recognition.onerror = (event) => {
            btnStart.disabled = false;
            if (event.error === 'not-allowed') {
                statusText.innerText = "❌ Lỗi: Bạn chưa cho phép Chrome mở Micro.";
                alert("Hãy bấm vào biểu tượng 🔒 hoặc hình cái Mic ở thanh địa chỉ web để CHO PHÉP (Allow) sử dụng Micro nhé!");
            } else if (event.error === 'no-speech') {
                statusText.innerText = "❌ Không nghe thấy tiếng. Hãy thử lại!";
            } else {
                statusText.innerText = "❌ Lỗi kết nối: " + event.error;
            }
        };

        recognition.onresult = (event) => {
            const resultText = event.results[0][0].transcript;
            // Xóa bỏ các dấu câu tiếng Trung để so sánh chính xác hơn
            const cleanResult = resultText.replace(/[.,\/#!$%\^&\*;:{}=\-_`~()？。，！]/g,"").trim();
            userSpokenText.innerText = cleanResult;

            const targetText = targetTextElement.innerText.trim();
            
            // Tính điểm tương đồng dựa trên số ký tự trùng khớp
            let score = calculateAccuracy(cleanResult, targetText);
            
            scoreDisplay.innerText = score + "%";
            scoreDisplay.className = "score"; 
            
            if(score >= 80) {
                scoreDisplay.classList.add('good');
                statusText.innerText = "🎉 Tuyệt vời! Phát âm rất chuẩn.";
            } else if(score >= 40) {
                scoreDisplay.classList.add('average');
                statusText.innerText = "👍 Khá tốt, cố gắng nói rõ chữ hơn.";
            } else {
                scoreDisplay.classList.add('bad');
                statusText.innerText = "😢 Chưa đúng rồi, bấm để thử lại nào!";
            }
        };
    }

    function changePhrase() {
        currentIdx = (currentIdx + 1) % phrases.length;
        document.getElementById('target-text').innerText = phrases[currentIdx].text;
        document.getElementById('target-pinyin').innerText = phrases[currentIdx].pinyin;
        document.getElementById('user-spoken').innerText = "...";
        document.getElementById('score-display').innerText = "--";
        document.getElementById('score-display').className = "score";
        document.getElementById('status').innerText = "Sẵn sàng...";
    }

    // Hàm chấm điểm thông minh hơn dựa trên độ dài chuỗi
    function calculateAccuracy(str1, str2) {
        if (str1 === str2) return 100;
        if (!str1 || !str2) return 0;

        let matchCount = 0;
        let visited = new Array(str2.length).fill(false);

        for (let i = 0; i < str1.length; i++) {
            for (let j = 0; j < str2.length; j++) {
                if (!visited[j] && str1[i] === str2[j]) {
                    matchCount++;
                    visited[j] = true;
                    break;
                }
            }
        }

        let maxLen = Math.max(str1.length, str2.length);
        return Math.round((matchCount / maxLen) * 100);
    }
</script>

</body>
</html>
