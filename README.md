<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>大模型对话</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f4f5fa; /* 浅灰色背景 */
            margin: 0;
            padding: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .chat-container {
            width: 70%;
            height: 100%;
            display: flex;
            flex-direction: column;
            position: relative;
            background-image: url('https://www.helloimg.com/i/2025/02/01/679db51a7dbc3.png&#39;);  /* 设置背景图片 */
            background-size: cover;
            background-position: center;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            box-sizing: border-box;
            padding: 20px;
        }
        .news-feed {
            width: 98%;
            height: 40%; /* 将高度增加到40% */
            overflow-y: auto;
            margin-bottom: 20px;
            border: 1px solid #ddd;
            border-radius: 10px;
            padding: 10px;
            box-sizing: border-box;
            background-color: rgba(255, 255, 255, 0.9); /* 白色背景，透明度50% */
            margin-left: 1%;
            position: relative;
            top: 10%; /* 沿Y轴下移15% */
        }
        .refresh-button-container {
            display: flex;
            justify-content: flex-end;
            margin-bottom: 10px;
        }
        .refresh-button {
            background: none;
            border: none;
            color: #007bff;
            cursor: pointer;
            font-size: 1.2em;
            position: relative;
            overflow: hidden;
        }
        .refresh-button.loading::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.2);
            z-index: 1;
        }
        .refresh-button.loading span {
            opacity: 0;
        }
        .refresh-button.loading::before {
            content: '🔄';
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            animation: spin 1s linear infinite;
            z-index: 2;
        }
        @keyframes spin {
            to {
                transform: rotate(360deg) translate(-50%, -50%);
            }
        }
        .news-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }
        .news-item {
            display: flex;
            align-items: center;
            width: calc(33% - 10px);
        }
        .news-image {
            width: 100px;
            height: 100px;
            object-fit: cover;
            margin-right: 10px;
            border-radius: 5px;
        }
        .news-details {
            flex-grow: 1;
        }
        .news-title {
            font-size: 1.2em;
            color: #333;
            margin-bottom: 5px;
        }
        .news-date {
            font-size: 0.9em;
            color: rgba(0, 0, 0, 0.5);
        }
        .button-area {
            display: flex;
            justify-content: flex-start; /* 向左靠拢 */
            padding: 10px;
            margin-bottom: 10px;
            gap: 10px; /* 缩小间距 */
            position: absolute;
            bottom: 80px; /* 往上移动一点，减少50%间隔 */
            width: 100%;
        }
        .button-area button {
            padding: 10px 20px;
            background-color: white; /* 按钮颜色改为白色 */
            color: black; /* 字体颜色保持不变 */
            border: 1px solid #738bf8;
            cursor: pointer;
            border-radius: 30px;
            font-size: 1em;
            transition: background-color 0.3s, border-color 0.3s;
        }
        .button-area button:hover {
            background-color: #e9ecef;
            border-color: #bbb;
        }
        .chat-messages {
            flex: 1;
            padding: 50px;
            overflow-y: auto;
            background-color: transparent;
            display: block; /* 初始状态下显示 */
        }
        .message.user {
            display: flex;
            justify-content: flex-end;
            margin-bottom: 15px;
        }
        .message.bot {
            display: flex;
            justify-content: flex-start;
            margin-bottom: 15px;
        }
        .message-content {
            background-color: rgba(255, 255, 255, 0.9);
            padding: 15px;
            border-radius: 15px;
            max-width: 70%;
            word-wrap: break-word;
        }
        .bot .message-content {
            background-color: rgba(255, 255, 255, 0.9);
        }
        .input-area {
            display: flex;
            width: 96%;
            padding: 10px;
            box-sizing: border-box;
            position: absolute;
            bottom: 20px;
        }
        .input-field {
            flex: 1;
            padding: 20px 20px; /* 增加高度一倍 */
            border: 2px solid #738bf8;
            outline: none;
            font-size: 1em;
            border-radius: 10px 0 0 10px; /* 左侧圆角 */
            background-color: rgba(255, 255, 255, 0.9);
            margin-right: -2px; /* 负边距以覆盖边界 */
            width: calc(100% - 100px); /* 计算宽度 */
        }
        .send-button {
            padding: 20px 20px; /* 增加高度一倍 */
            background: linear-gradient(to right, #738bf8, #0047ab);
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 0 10px 10px 0; /* 右侧圆角 */
            font-size: 1em;
            margin-left: -2px; /* 负边距以覆盖边界 */
            width: 80px; /* 固定宽度 */
        }
        .send-button:hover {
            background: linear-gradient(to right, #b9d7ea, #8cbdd9);
        }
        .send-icon {
            width: 20px;
            height: 20px;
            fill: white;
        }
        .company-selector {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 600px;
            min-width: 300px;
            max-height: 160vh;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            z-index: 1000;
            overflow-y: auto;
            padding: 20px;
            box-sizing: border-box;
        }
        .tool-report-selector {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 600px;
            min-width: 300px;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            z-index: 1000;
            overflow-y: auto;
            padding: 20px;
            box-sizing: border-box;
        }
        .overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 999;
        }
        .company-title {
            font-size: 2em;
            font-weight: bold;
            text-align: center;
            margin-bottom: 20px;
        }
        .company-buttons {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .company-row {
            display: flex;
            justify-content: flex-start;
            gap: 10px;
            flex-wrap: wrap;
        }
        .company-row button {
            padding: 10px 20px;
            background-color: white;
            color: black;
            border: 1px solid #ccc;
            cursor: pointer;
            border-radius: 5px;
            font-size: 1em;
            transition: background-color 0.3s, border-color 0.3s;
            white-space: nowrap;
        }
        .company-row button:hover {
            background-color: #e9ecef;
            border-color: #bbb;
        }
        .sub-button-area {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .sub-button-area button {
            margin-right: 10px;
            margin-bottom: 10px;
        }
        .sub-button-area button:last-child {
            margin-right: 0;
        }
        .ai-intro-container {
            display: none;
            position: absolute;
            top: 20%;
            left: 50%;
            transform: translateX(-50%);
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            z-index: 1000;
            overflow-y: auto;
            padding: 20px;
            box-sizing: border-box;
            width: 60%;
            max-height: 60vh;
        }
        .ai-intro-container button {
            font-size: 1.1em;
            width: 100%;
            text-align: left;
            background-color: #e5effa;
            color: black;
            border: 1px solid #738bf8;
            cursor: pointer;
            border-radius: 30px;
            transition: background-color 0.3s, border-color 0.3s;
        }
        .ai-intro-container button:hover {
            background-color: #b9d7ea;
            border-color: #bbb;
        }
        .close-button {
            position: absolute;
            top: 10px;
            right: 10px;
            background: none;
            border: none;
            color: red;
            font-size: 1.5em;
            cursor: pointer;
        }
        .stop-generating-button {
            display: none;
            padding: 10px 20px;
            background-color: #ffcccc;
            color: #cc0000;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }
        .stop-generating-button:hover {
            background-color: #ff9999;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="news-feed" id="newsFeed">
            <div class="refresh-button-container">
                <button class="refresh-button" onclick="refreshNews()">
                    <span>🔄</span>
                </button>
            </div>
            <div class="news-row" id="newsRow1">
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 1" class="news-image">
                    <div class="news-details">
                        <div class="news-title">人工智能技术引领未来</div>
                        <div class="news-date">2025年1月30日</div>
                    </div>
                </div>
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 2" class="news-image">
                    <div class="news-details">
                        <div class="news-title">AI改变生活的力量</div>
                        <div class="news-date">2025年1月29日</div>
                    </div>
                </div>
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 3" class="news-image">
                    <div class="news-details">
                        <div class="news-title">探索机器学习的奥秘</div>
                        <div class="news-date">2025年1月28日</div>
                    </div>
                </div>
            </div>
            <div class="news-row" id="newsRow2">
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 4" class="news-image">
                    <div class="news-details">
                        <div class="news-title">深度学习的发展趋势</div>
                        <div class="news-date">2025年1月27日</div>
                    </div>
                </div>
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 5" class="news-image">
                    <div class="news-details">
                        <div class="news-title">自然语言处理的新进展</div>
                        <div class="news-date">2025年1月26日</div>
                    </div>
                </div>
                <div class="news-item">
                    <img src="https://img.alicdn.com/tfs/TB1QVgGqYLYBeNjy0FiXXcXgpXa-300-300.png&#34;  alt="News Image 6" class="news-image">
                    <div class="news-details">
                        <div class="news-title">计算机视觉的应用案例</div>
                        <div class="news-date">2025年1月25日</div>
                    </div>
                </div>
            </div>
        </div>
        <div class="button-area">
    <button onclick="sendMessage('行业报告')">行业报告</button>
    <button onclick="window.open('https://www.aigc.cn/#term-1907-700&#39;,  '_blank')">AI工具大全</button>
    <button onclick="showToolReportSelector()">AI工具测评报告</button>
    <button onclick="toggleAIDialog()">AI入门</button>
    <button onclick="showCompanySelector()">大公司资讯</button>
    <button onclick="sendMessage('实时行业资讯')">实时行业资讯</button>
    <button onclick="goToHome()">主页</button> <!-- 新增主页按钮 -->
    </div>
        <div class="chat-messages" id="chatMessages"></div>
        <button class="stop-generating-button" id="stopGeneratingButton" onclick="stopGenerating()">停止生成</button>
        <div class="input-area">
            <input type="text" class="input-field" id="userInput" placeholder="请输入您的问题..." onkeypress="handleKeyPress(event)">
            <button class="send-button" onclick="sendMessage()">
                <svg class="send-icon" xmlns="http://www.w3.org/2000/svg&#34;  viewBox="0 0 24 24">
                    <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
                </svg>
            </button>
        </div>
        <div class="ai-intro-container" id="aiIntroContainer">
            <span class="close-button" onclick="hideAIDialog()">&times;</span>
            <div class="sub-button-area" id="subButtonArea">
                <button onclick="displaySubButtons('需要认识的人工智能')">需要认识的人工智能</button>
                <button onclick="displaySubButtons('机器学习')">机器学习</button>
                <button onclick="displaySubButtons('多模态感知及理解')">多模态感知及理解</button>
                <button onclick="displaySubButtons('AI产品从定义到落地')">AI产品从定义到落地</button>
            </div>
        </div>
        <div class="tool-report-selector" id="toolReportSelector">
            <div class="company-buttons">
                <div class="company-row">
                    <button onclick="sendMessageWithParams('大语言模型')">大语言模型</button>
                    <button onclick="sendMessageWithParams('AI音频')">AI音频</button>
                    <button onclick="sendMessageWithParams('AI图像')">AI图像</button>
                    <button onclick="sendMessageWithParams('AI视频')">AI视频</button>
                </div>
            </div>
        </div>
    </div>
    <div class="overlay" id="overlay" onclick="hideSelectors()"></div>
    <div class="company-selector" id="companySelector">
        <div class="company-title">国内</div>
        <div class="company-buttons">
            <div class="company-row">
                <button onclick="selectCompany('百度')">百度</button>
                <button onclick="selectCompany('阿里巴巴')">阿里巴巴</button>
                <button onclick="selectCompany('腾讯')">腾讯</button>
                <button onclick="selectCompany('字节跳动')">字节跳动</button>
                <button onclick="selectCompany('科大讯飞')">科大讯飞</button>
            </div>
            <div class="company-row">
                <button onclick="selectCompany('快手')">快手</button>
                <button onclick="selectCompany('智谱')">智谱</button>
                <button onclick="selectCompany('商汤科技')">商汤科技</button>
                <button onclick="selectCompany('昆仑万维')">昆仑万维</button>
                <button onclick="selectCompany('生数科技')">生数科技</button>
            </div>
            <div class="company-row">
                <button onclick="selectCompany('秘塔')">秘塔</button>
                <button onclick="selectCompany('月之暗面')">月之暗面</button>
                <button onclick="selectCompany('Minimax')">Minimax</button>
            </div>
        </div>
        <div class="company-title">国外</div>
        <div class="company-buttons">
            <div class="company-row">
                <button onclick="selectCompany('OpenAI')">OpenAI</button>
                <button onclick="selectCompany('微软')">微软</button>
                <button onclick="selectCompany('NVIDIA')">NVIDIA</button>
                <button onclick="selectCompany('谷歌')">谷歌</button>
                <button onclick="selectCompany('Meta')">Meta</button>
            </div>
            <div class="company-row">
                <button onclick="selectCompany('亚马逊')">亚马逊</button>
                <button onclick="selectCompany('X平台')">X平台</button>
            </div>
        </div>
    </div>

    <!-- 引入 marked.js 库 -->
    <script src="<url id="" type="url" status="" title="" wc="">https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script></url> 

    <script>
        // 修改marked.js的渲染器，为超链接添加target="_blank"
if (typeof marked === 'function') {
    marked.use({
        renderer: {
            link(href, title, text) {
                return `<a href="${href}" title="${title}" target="_blank">${text}</a>`;
            }
        }
    });
} else {
    console.error('marked.js is not loaded correctly.');
}

const apiURL = "https://api.coze.cn/v1/workflow/stream_run&#34;;  // 替换为实际的API URL
const apiKey = "pat_HErM452z7KnDSYlrpQBhc1vzos2RNoZJtYmZkhxF62pQi6uf9NyfOnsRLVQLXA8a"; // 替换为实际的token
const workflowId = "7463422440091598857"; // 替换为实际的workflow ID
const refreshWorkflowId = "7465745755198717971"; // 新的工作流ID用于刷新新闻
const botId = ""; // 如果有具体的bot_id，请在这里填写
const extParams = {}; // 如果有额外的参数，请在这里填写

let currentBotMessageBuffer = '';
let currentMessageElement = null;
let controller = null;
let isGenerating = false;

        function sendMessage(message, type) {
            if (isGenerating) {
                stopGenerating();
            }

            if (!message) message = document.getElementById('userInput').value.trim();
            if (!message) return;

            appendMessage(message, 'user');

            currentBotMessageBuffer = '';

            currentMessageElement = createMessageElement('', 'bot');

            const parameters = { input: message };
            if (type) {
                parameters.type = type;
            }

            isGenerating = true;
            document.getElementById('stopGeneratingButton').style.display = 'block';

            controller = new AbortController();
            const signal = controller.signal;

            fetch(apiURL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${apiKey}`
                },
                body: JSON.stringify({
                    parameters: parameters,
                    workflow_id: workflowId,
                    bot_id: botId,
                    ext: extParams
                }),
                signal: signal
            }).then(response => {
                if (!response.ok) {
                    throw new Error('Network response was not ok ' + response.statusText);
                }
                return response.body.getReader();
            }).then(reader => {
                readStream(reader);
            }).catch(error => {
                console.error('There has been a problem with your fetch operation:', error);
                appendMessage('请求出错，请稍后再试。', 'bot');
            });

            document.getElementById('userInput').value = '';
            
            hideAIDialog();
            hideCompanySelector();
            hideToolReportSelector();
            showChatMessages(); // 显示对话框消息内容
        }

        function handleKeyPress(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        }

        function readStream(reader) {
            reader.read().then(({ done, value }) => {
                if (done || !isGenerating) {
                    console.log('Stream complete or stopped');
                    if (currentBotMessageBuffer) {
                        updateCurrentMessage(currentBotMessageBuffer);
                    }
                    document.getElementById('stopGeneratingButton').style.display = 'none';
                    return;
                }
                processChunk(value);
                return readStream(reader);
            });
        }

        function processChunk(chunk) {
            const decoder = new TextDecoder('utf-8');
            const chunkStr = decoder.decode(chunk, { stream: true });
            const lines = chunkStr.split('\n').filter(line => line.startsWith('data:'));

            lines.forEach(line => {
                try {
                    const data = JSON.parse(line.substring(5));
                    console.log('Received event:', data);
                    handleEvent(data);
                } catch (error) {
                    console.warn('Failed to parse JSON, treating as plain text:', error);
                    updateCurrentMessage(line.substring(5));
                }
            });
        }

        function handleEvent(eventData) {
            if (eventData.content && eventData.content_type === 'text') {
                currentBotMessageBuffer += eventData.content;
                if (eventData.event === 'Done') {
                    updateCurrentMessage(currentBotMessageBuffer);
                    currentBotMessageBuffer = '';
                } else {
                    updateCurrentMessage(currentBotMessageBuffer);
                }
            } else if (eventData.debug_url) {
                console.log('Debug URL:', eventData.debug_url);
            } else {
                console.log('Unhandled event data:', eventData);
            }
        }
function updateCurrentMessage(content) {
    if (!currentMessageElement) {
        currentMessageElement = createMessageElement(content, 'bot');
    } else {
        const messageContent = currentMessageElement.querySelector('.message-content');
        if (messageContent) {
            // 使用marked.js解析Markdown内容
            let parsedContent = marked.parse(content);
            // 替换所有<a>标签，添加target="_blank"
            parsedContent = parsedContent.replace(/<a\s+href="([^"]+)"\s*>([^<]+)<\/a>/g, '<a href="$1" target="_blank">$2</a>');
            messageContent.innerHTML = parsedContent;
        }
    }
    scrollChatToBottom();
}

        function createMessageElement(content, sender) {
            const chatMessages = document.getElementById('chatMessages');
            const messageElement = document.createElement('div');
            messageElement.classList.add('message', sender);
            const messageContent = document.createElement('div');
            messageContent.classList.add('message-content');

            if (sender === 'bot') {
                messageContent.innerHTML = marked.parse(content);
            } else {
                messageContent.textContent = content;
            }

            messageElement.appendChild(messageContent);
            chatMessages.appendChild(messageElement);
            return messageElement;
        }

        function appendMessage(content, sender) {
            const chatMessages = document.getElementById('chatMessages');
            const messageElement = document.createElement('div');
            messageElement.classList.add('message', sender);
            const messageContent = document.createElement('div');
            messageContent.classList.add('message-content');

            if (sender === 'bot') {
                messageContent.innerHTML = marked.parse(content);
            } else {
                messageContent.textContent = content;
            }

            messageElement.appendChild(messageContent);
            chatMessages.appendChild(messageElement);
            scrollChatToBottom();
        }

        function scrollChatToBottom() {
            const chatMessages = document.getElementById('chatMessages');
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        function showCompanySelector() {
            document.getElementById('overlay').style.display = 'block';
            document.getElementById('companySelector').style.display = 'flex';
        }

        function hideCompanySelector() {
            document.getElementById('overlay').style.display = 'none';
            document.getElementById('companySelector').style.display = 'none';
        }

        function selectCompany(companyName) {
            sendMessage(companyName, '大公司资讯');
            hideCompanySelector();
        }

        function toggleAIDialog() {
            const aiIntroContainer = document.getElementById('aiIntroContainer');
            if (aiIntroContainer.style.display === 'none' || !aiIntroContainer.style.display) {
                aiIntroContainer.style.display = 'block';
            } else {
                aiIntroContainer.style.display = 'none';
            }
        }

        function displaySubButtons(mainButton) {
            const subButtons = getSubButtonsForMainButton(mainButton);
            const subButtonArea = document.getElementById('subButtonArea');
            subButtonArea.innerHTML = `
                <div class="sub-button-area">
                    ${subButtons.map(button => `<button onclick="sendMessage('${button}')">${button}</button>`).join('')}
                    <button onclick="resetToMainMenu()">返回</button>
                </div>
            `;
        }

        function resetToMainMenu() {
            const subButtonArea = document.getElementById('subButtonArea');
            subButtonArea.innerHTML = `
                <div class="sub-button-area">
                    <button onclick="displaySubButtons('需要认识的人工智能')">需要认识的人工智能</button>
                    <button onclick="displaySubButtons('机器学习')">机器学习</button>
                    <button onclick="displaySubButtons('多模态感知及理解')">多模态感知及理解</button>
                    <button onclick="displaySubButtons('AI产品从定义到落地')">AI产品从定义到落地</button>
                </div>
            `;
        }

        function getSubButtonsForMainButton(mainButton) {
            switch (mainButton) {
                case '需要认识的人工智能':
                    return [
                        '与“互联网思维”不同的“人工智能思维”',
                        '机器模拟人脑学习的五种方式',
                        'AI大模型的底层原理是什么？',
                        '人工智能系统的结构',
                        '模拟人脑的神经网络模型'
                    ];
                case '机器学习':
                    return [
                        '机器学习的基本概念',
                        '监督学习与非监督学习的区别',
                        '深度学习的发展历程',
                        '常见的机器学习算法',
                        '机器学习的应用场景'
                    ];
                case '多模态感知及理解':
                    return [
                        '多模态数据的概念',
                        '跨模态学习的方法',
                        '自然语言处理与计算机视觉的结合',
                        '语音识别技术的发展',
                        '情感分析的应用'
                    ];
                case 'AI产品从定义到落地':
                    return [
                        '产品的市场调研',
                        '需求分析的重要性',
                        '原型设计与迭代',
                        '技术选型与架构设计',
                        '项目管理与团队协作'
                    ];
                default:
                    return [];
            }
        }

        document.addEventListener('click', function(event) {
            const aiIntroContainer = document.getElementById('aiIntroContainer');
            if (aiIntroContainer.style.display === 'block' && !aiIntroContainer.contains(event.target)) {
                aiIntroContainer.style.display = 'none';
            }
        });

        document.querySelector('.button-area button:nth-child(4)').addEventListener('click', function(event) {
            event.stopPropagation();
        });

        document.getElementById('aiIntroContainer').innerHTML = `
            <div class="sub-button-area" id="subButtonArea">
                <button onclick="displaySubButtons('需要认识的人工智能')">需要认识的人工智能</button>
                <button onclick="displaySubButtons('机器学习')">机器学习</button>
                <button onclick="displaySubButtons('多模态感知及理解')">多模态感知及理解</button>
                <button onclick="displaySubButtons('AI产品从定义到落地')">AI产品从定义到落地</button>
            </div>
        `;

        document.getElementById('aiIntroContainer').addEventListener('click', function(event) {
            event.stopPropagation();
        });

        if (typeof marked !== 'function') {
            console.error('marked.js is not loaded correctly.');
        }

        function hideAIDialog() {
            document.getElementById('aiIntroContainer').style.display = 'none';
        }

        function showToolReportSelector() {
            document.getElementById('overlay').style.display = 'block';
            document.getElementById('toolReportSelector').style.display = 'flex';
        }

        function hideSelectors() {
            hideCompanySelector();
            hideAIDialog();
            hideToolReportSelector();
        }

        function hideToolReportSelector() {
            document.getElementById('overlay').style.display = 'none';
            document.getElementById('toolReportSelector').style.display = 'none';
        }

        function sendMessageWithParams(inputValue) {
            sendMessage(inputValue, '产品评测');
        }

        function refreshNews() {
            const refreshButton = document.querySelector('.refresh-button');
            refreshButton.classList.add('loading');

            fetch(apiURL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${apiKey}`
                },
                body: JSON.stringify({
                    workflow_id: refreshWorkflowId,
                    bot_id: botId,
                    ext: extParams
                })
            }).then(response => {
                if (!response.ok) {
                    throw new Error('Network response was not ok ' + response.statusText);
                }
                return response.text();
            }).then(textResponse => {
                console.log('Text Response:', textResponse);
                try {
                    const events = textResponse.split('\n').filter(line => line.startsWith('data:'));
                    let newsData = {};
                    events.forEach(event => {
                        try {
                            const jsonData = JSON.parse(event.substring(5));
                            if (jsonData.content && jsonData.content_type === 'text') {
                                newsData = JSON.parse(jsonData.content);
                            }
                        } catch (parseError) {
                            console.warn('Failed to parse JSON within event:', parseError);
                        }
                    });
                    updateNewsFeed(newsData);
                } catch (parseError) {
                    console.error('Failed to parse JSON:', parseError);
                    alert('刷新新闻失败，请稍后再试。');
                }
            }).catch(error => {
                console.error('There has been a problem with your fetch operation:', error);
                alert('刷新新闻失败，请稍后再试。');
            }).finally(() => {
                refreshButton.classList.remove('loading');
            });
        }

        function updateNewsFeed(newsData) {
            for (let i = 1; i <= 6; i++) {
                const newsItem = {
                    title: newsData[`new${i}`] || "无标题",
                    url: newsData[`newurl${i}`] || "#",
                    date: newsData[`newtime${i}`] || "未知日期",
                    image: newsData[`newimage${i}`] || "https://via.placeholder.com/100&#34; 
                };

                const rowNumber = Math.ceil(i / 3);
                const itemIndex = (i - 1) % 3;
                const newsRow = document.getElementById(`newsRow${rowNumber}`);
                const newsItems = newsRow.getElementsByClassName('news-item');

                if (newsItems[itemIndex]) {
                    const img = newsItems[itemIndex].querySelector('.news-image');
                    const title = newsItems[itemIndex].querySelector('.news-title');
                    const date = newsItems[itemIndex].querySelector('.news-date');

                    img.src = newsItem.image;
                    title.innerHTML = `<a href="${newsItem.url}" target="_blank">${newsItem.title}</a>`;
                    date.textContent = newsItem.date;
                }
            }
        }

        function goToHome() {
            document.getElementById('newsFeed').style.display = 'block'; // 显示新闻资讯展示框
            document.getElementById('chatMessages').style.display = 'none'; // 隐藏对话框消息内容
        }

        function showChatMessages() {
            document.getElementById('newsFeed').style.display = 'none'; // 隐藏新闻资讯展示框
            document.getElementById('chatMessages').style.display = 'block'; // 显示对话框消息内容
        }

        function stopGenerating() {
            if (controller) {
                controller.abort();
                isGenerating = false;
                document.getElementById('stopGeneratingButton').style.display = 'none';
            }
        }

        window.onload = function() {
            adjustChatMessagesHeight();
            window.addEventListener('resize', adjustChatMessagesHeight);
            refreshNews(); // 自动刷新新闻
        };

        function adjustChatMessagesHeight() {
            const chatContainer = document.querySelector('.chat-container');
            const buttonArea = document.querySelector('.button-area');
            const inputArea = document.querySelector('.input-area');
            const newsFeed = document.querySelector('.news-feed');

            const containerHeight = chatContainer.clientHeight;
            const buttonAreaHeight = buttonArea.offsetHeight;
            const inputAreaHeight = inputArea.offsetHeight;
            const newsFeedHeight = newsFeed.offsetHeight;

            const availableHeight = containerHeight - buttonAreaHeight - inputAreaHeight - newsFeedHeight - 40; // 40px for some extra spacing

            document.querySelector('.chat-messages').style.maxHeight = `${availableHeight}px`;
        }
    </script>
</body>
</html>
