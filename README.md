<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>일본 문화 챗봇</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap');
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f0f4f8;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .chat-container {
            width: 100%;
            max-width: 500px;
            height: 80vh;
            display: flex;
            flex-direction: column;
            background-color: #ffffff;
            border-radius: 1.5rem;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }
        .chat-header {
            background-color: #6366f1;
            color: white;
            padding: 1rem;
            text-align: center;
            font-weight: bold;
            font-size: 1.5rem;
            border-top-left-radius: 1.5rem;
            border-top-right-radius: 1.5rem;
        }
        .chat-box {
            flex-grow: 1;
            padding: 1.5rem;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 1rem;
        }
        .message {
            max-width: 80%;
            padding: 0.75rem 1rem;
            border-radius: 1rem;
            word-wrap: break-word;
        }
        .user-message {
            background-color: #e0e7ff;
            align-self: flex-end;
            border-bottom-right-radius: 0.25rem;
        }
        .bot-message {
            background-color: #f1f5f9;
            align-self: flex-start;
            border-bottom-left-radius: 0.25rem;
        }
        .input-area {
            display: flex;
            padding: 1rem;
            gap: 0.5rem;
            border-top: 1px solid #e2e8f0;
        }
        .input-area input {
            flex-grow: 1;
            padding: 0.75rem 1rem;
            border-radius: 9999px;
            border: 1px solid #cbd5e1;
            outline: none;
        }
        .input-area button {
            background-color: #6366f1;
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 9999px;
            font-weight: bold;
            transition: background-color 0.2s;
        }
        .input-area button:hover {
            background-color: #4f46e5;
        }
        .loading-dots span {
            animation: bounce 0.8s infinite;
        }
        .loading-dots span:nth-child(2) { animation-delay: 0.2s; }
        .loading-dots span:nth-child(3) { animation-delay: 0.4s; }
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-5px); }
        }
        .message.bot-loading {
            background-color: #f1f5f9;
            align-self: flex-start;
            border-bottom-left-radius: 0.25rem;
            padding: 0.75rem 1rem;
            display: flex;
            align-items: center;
        }
    </style>
</head>
<body class="bg-gray-100 p-4">

<div class="chat-container">
    <div class="chat-header">일본 문화 친구 챗봇</div>
    <div id="chat-box" class="chat-box">
        <div class="message bot-message">
            안녕하세요! 저는 일본 문화에 대해 알려주는 친구예요. 무엇이든 물어보세요! 😊
        </div>
    </div>
    <div class="input-area">
        <input type="text" id="user-input" placeholder="질문을 입력해 주세요...">
        <button id="send-button">전송</button>
    </div>
</div>

<script type="module">
    // Firebase 설정은 사용하지 않지만, 템플릿 규칙에 따라 포함합니다.
    const __app_id = 'your-app-id';
    const __firebase_config = '{}';
    const __initial_auth_token = 'your-auth-token';
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const firebaseConfig = JSON.parse(__firebase_config);

    const userInput = document.getElementById('user-input');
    const sendButton = document.getElementById('send-button');
    const chatBox = document.getElementById('chat-box');
    
    // API 키는 이 환경에서 자동으로 제공됩니다.
    const apiKey = ""; 

    async function sendMessage() {
        const userMessage = userInput.value.trim();
        if (userMessage === '') return;

        // 사용자 메시지를 화면에 표시
        addMessage(userMessage, 'user');
        userInput.value = '';

        // 로딩 메시지 추가
        const loadingMessageElement = addLoadingMessage();
        
        // Gemini API 호출
        try {
            // 시스템 지시문: 챗봇의 페르소나와 규칙을 정의합니다.
            const systemPrompt = `
                너는 초등학생을 위한 일본 문화 챗봇이야. 너의 이름은 하루야.
                - 초등학생이 이해하기 쉬운 친절한 말투와 이모티콘을 사용해.
                - 일본 문화(음식, 전통 의상, 축제, 학교 생활, 일본어 인사말, 애니메이션, 만화 등)와 관련된 질문에만 답변해.
                - **일본 문화와 관련 없는 질문이 들어오면, "죄송해요, 저는 일본 문화에 대해서만 대답할 수 있어요. 다른 질문은 답해드릴 수 없답니다. 다시 질문해 주실래요?" 라고 답변해줘.**
                - 질문이 일본 문화와 관련이 있는지 없는지 스스로 판단해야 해.
            `;

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: userMessage }] }],
                tools: [{ "google_search": {} }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                },
            };
            
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                console.error('API Error:', response.status, response.statusText);
                throw new Error('API 호출 중 오류가 발생했습니다.');
            }

            const result = await response.json();
            const candidate = result.candidates?.[0];
            const botMessageText = candidate?.content?.parts?.[0]?.text || '답변을 생성하는 데 문제가 발생했습니다. 다시 시도해 주세요.';
            
            removeLoadingMessage(loadingMessageElement);
            addMessage(botMessageText, 'bot');

        } catch (error) {
            console.error('Error:', error);
            removeLoadingMessage(loadingMessageElement);
            addMessage('죄송해요, 지금은 답변을 할 수 없습니다. 잠시 후 다시 시도해 주세요.', 'bot');
        }
    }

    // 메시지를 채팅창에 추가하는 함수
    function addMessage(text, sender) {
        const messageDiv = document.createElement('div');
        messageDiv.classList.add('message');
        if (sender === 'user') {
            messageDiv.classList.add('user-message');
        } else {
            messageDiv.classList.add('bot-message');
        }
        messageDiv.textContent = text;
        chatBox.appendChild(messageDiv);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    // 로딩 메시지 추가 함수
    function addLoadingMessage() {
        const loadingDiv = document.createElement('div');
        loadingDiv.classList.add('message', 'bot-loading');
        loadingDiv.innerHTML = `<div class="loading-dots">
            <span class="text-sm">●</span>
            <span class="text-sm">●</span>
            <span class="text-sm">●</span>
        </div>`;
        chatBox.appendChild(loadingDiv);
        chatBox.scrollTop = chatBox.scrollHeight;
        return loadingDiv;
    }

    // 로딩 메시지 제거 함수
    function removeLoadingMessage(element) {
        if (element && element.parentNode) {
            element.parentNode.removeChild(element);
        }
    }

    // 전송 버튼 클릭 이벤트
    sendButton.addEventListener('click', sendMessage);

    // Enter 키 입력 이벤트
    userInput.addEventListener('keydown', (event) => {
        if (event.key === 'Enter') {
            sendMessage();
        }
    });

</script>
</body>
</html>
