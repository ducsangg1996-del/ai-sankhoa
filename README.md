<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OB-GYN Assistant - Trợ lý Sản Phụ Khoa</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
            background-color: #f5f5f7;
            margin: 0; padding: 0;
            display: flex; justify-content: center; align-items: center; min-height: 100vh;
        }
        .phone-container {
            width: 100%; max-width: 412px; height: 92vh;
            background-color: #ffffff; box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            border-radius: 30px; display: flex; flex-direction: column; overflow: hidden; border: 6px solid #333;
        }
        .header { padding: 15px; text-align: center; border-bottom: 1px solid #eee; background-color: #fff; }
        .header h1 { font-size: 20px; margin: 5px 0; color: #1d1d1f; }
        .header p { font-size: 13px; color: #86868b; margin: 0; }
        .chat-area { flex: 1; padding: 15px; overflow-y: auto; background-color: #fafafa; }
        .mode-container { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .mode-btn {
            background: #ffffff; border: 2px solid #e5e5ea; border-radius: 12px;
            padding: 12px 8px; text-align: left; cursor: pointer; transition: all 0.2s;
        }
        .mode-btn.active { border-color: #6c5ce7; background-color: #f3f0ff; }
        .mode-btn h3 { margin: 0 0 4px 0; font-size: 13px; color: #1d1d1f; }
        .mode-btn p { margin: 0; font-size: 10px; color: #86868b; line-height: 1.3; }
        .output-box {
            background: #ffffff; border-radius: 12px; padding: 15px;
            border: 1px solid #e5e5ea; min-height: 150px; font-size: 14px; line-height: 1.6; white-space: pre-line;
        }
        .api-box { margin-bottom: 15px; display: flex; gap: 5px; }
        .api-input { flex: 1; padding: 8px; border: 1px solid #ccc; border-radius: 6px; font-size: 12px; }
        .input-area { padding: 15px; border-top: 1px solid #eee; background-color: #fff; display: flex; gap: 8px; align-items: center; }
        .input-box { flex: 1; padding: 10px 12px; border: 1px solid #d2d2d7; border-radius: 20px; font-size: 14px; outline: none; }
        .btn-send { background-color: #6c5ce7; color: white; border: none; border-radius: 50%; width: 38px; height: 38px; cursor: pointer; font-size: 14px; }
        .btn-mic { background-color: #ff453a; color: white; border: none; border-radius: 50%; width: 38px; height: 38px; cursor: pointer; }
    </style>
</head>
<body>

<div class="phone-container">
    <div class="header">
        <h1>🤰 OB-GYN Assistant</h1>
        <p><b>Trợ lý thông dịch sản phụ khoa hai chiều</b></p>
    </div>

    <div class="chat-area">
        <div class="api-box">
            <input type="password" class="api-input" id="api-key" placeholder="Dán OpenAI API Key (sk-...) tại đây để chạy">
        </div>

        <div class="mode-container">
            <div class="mode-btn active" id="btn-doctor" onclick="setMode('doctor')">
                <h3>🇻🇳 → 🌐 Bác sĩ nói</h3>
                <p>Nhập Tiếng Việt → Dịch + Phiên âm sang tiếng khách</p>
            </div>
            <div class="mode-btn" id="btn-patient" onclick="setMode('patient')">
                <h3>🌐 → 🇻🇳 Khách nói</h3>
                <p>Nhập tiếng nước ngoài → Dịch + Gợi ý trả lời</p>
            </div>
        </div>

        <div class="output-box" id="output-content">
            Hệ thống đã sẵn sàng. Vui lòng nhập API Key của bạn để bắt đầu dịch tự động bằng trí tuệ nhân tạo (GPT-4o).
        </div>
    </div>

    <div class="input-area">
        <input type="text" class="input-box" id="user-input" placeholder="Nhập câu thoại tại đây...">
        <button class="btn-mic" id="mic-btn" onclick="toggleListening()">🎤</button>
        <button class="btn-send" onclick="processTranslation()">➔</button>
    </div>
</div>

<script>
    let currentMode = 'doctor';
    let isListening = false;
    let recognition;

    if ('webkitSpeechRecognition' in window) {
        recognition = new webkitSpeechRecognition();
        recognition.continuous = false;
        recognition.interimResults = false;
        recognition.onstart = () => { isListening = true; document.getElementById('mic-btn').style.backgroundColor = '#34c759'; };
        recognition.onresult = (e) => { document.getElementById('user-input').value = e.results[0][0].transcript; };
        recognition.onerror = () => stopListening();
        recognition.onend = () => stopListening();
    }

    function toggleListening() {
        if (!recognition) { alert("Trình duyệt không hỗ trợ giọng nói, hãy gõ phím nhé!"); return; }
        if (isListening) { recognition.stop(); } 
        else { recognition.lang = currentMode === 'doctor' ? 'vi-VN' : 'en-US'; recognition.start(); }
    }
    function stopListening() { isListening = false; document.getElementById('mic-btn').style.backgroundColor = '#ff453a'; }
    function setMode(m) { currentMode = m; document.getElementById('btn-doctor').classList.toggle('active', m==='doctor'); document.getElementById('btn-patient').classList.toggle('active', m==='patient'); }

    // GỌI API OPENAI THẬT ĐỂ ĐỌC VÀ DỊCH Y KHOA
    async function processTranslation() {
        const apiKey = document.getElementById('api-key').value.trim();
        const userInput = document.getElementById('user-input').value.trim();
        const outputBox = document.getElementById('output-content');

        if (!apiKey) { alert("Vui lòng điền OpenAI API Key của bạn vào ô phía trên!"); return; }
        if (!userInput) return;

        outputBox.innerHTML = "⏳ Đang kết nối AI và dịch thuật ngữ y khoa...";
        
        const systemPrompt = `Bạn là OB-GYN Assistant - Trợ lý thông dịch hai chiều tại phòng khám Sản Phụ khoa. 
Nguyên tắc thuật ngữ: Dùng từ y khoa quốc tế chuẩn xác (Gestational age, Due date, Contractions, Amniotic fluid, C-section...). Ngắn gọn, chỉ hiển thị kết quả theo cấu trúc, không giải thích.

Nhiệm vụ:
${currentMode === 'doctor' 
    ? 'Đầu vào là tiếng Việt từ Bác sĩ. Hãy dịch sang tiếng Anh (hoặc ngôn ngữ của khách tương ứng) và kèm phiên âm tiếng Việt dễ đọc. Định dạng: \n**Dịch sang [Ngôn ngữ]:** [Câu dịch]\n**Phiên âm:** [Cách đọc].' 
    : 'Đầu vào là tiếng nước ngoài từ Bệnh nhân. Hãy dịch sang tiếng Việt và gợi ý câu hỏi/câu trả lời lâm sàng tiếp theo bằng tiếng nước ngoài đó cho bác sĩ nói. Định dạng: \n**Dịch nghĩa tiếng Việt:** [Câu dịch]\n**Gợi ý trả lời:** [Câu gợi ý]'}`;

        try {
            const response = await fetch('https://api.openai.com/v1/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${apiKey}`
                },
                body: JSON.stringify({
                    model: "gpt-4o-mini", // Dùng bản mini để tốc độ dịch cực nhanh và tiết kiệm chi phí
                    messages: [
                        { role: "system", content: systemPrompt },
                        { role: "user", content: userInput }
                    ],
                    temperature: 0.2
                })
            });

            const data = await response.json();
            if (data.choices && data.choices[0]) {
                outputBox.innerHTML = data.choices[0].message.content;
                document.getElementById('user-input').value = "";
            } else {
                outputBox.innerHTML = "❌ Lỗi: API Key không đúng hoặc hết hạn hạn mức.";
            }
        } catch (error) {
            outputBox.innerHTML = "❌ Lỗi kết nối mạng hoặc lỗi API.";
            console.error(error);
        }
    }
</script>
</body>
</html>
