<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Shared Drawing Board</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            height: 100vh;
            overflow: hidden;
        }

        .container {
            display: flex;
            height: 100vh;
            transition: all 0.3s ease;
        }

        .drawing-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            background: white;
            margin: 10px;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }

        .drawing-area.fullscreen {
            margin: 0;
            border-radius: 0;
        }

        .toolbar {
            background: #2c3e50;
            padding: 15px 20px;
            display: flex;
            align-items: center;
            gap: 15px;
            flex-wrap: wrap;
            border-radius: 15px 15px 0 0;
        }

        .drawing-area.fullscreen .toolbar {
            border-radius: 0;
        }

        .tool-group {
            display: flex;
            align-items: center;
            gap: 10px;
            background: rgba(255,255,255,0.1);
            padding: 8px 12px;
            border-radius: 8px;
        }

        .tool-group label {
            color: white;
            font-size: 14px;
            font-weight: 500;
        }

        #colorPicker {
            width: 50px;
            height: 35px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        #brushSize {
            width: 120px;
            height: 8px;
            background: rgba(255,255,255,0.3);
            border-radius: 4px;
            outline: none;
            cursor: pointer;
        }

        #brushSize::-webkit-slider-thumb {
            appearance: none;
            width: 20px;
            height: 20px;
            background: #3498db;
            border-radius: 50%;
            cursor: pointer;
        }

        .action-btn {
            background: #e74c3c;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.3s ease;
        }

        .action-btn:hover {
            background: #c0392b;
            transform: translateY(-2px);
        }

        .action-btn.disabled {
            background: #95a5a6;
            cursor: not-allowed;
            transform: none;
        }

        .action-btn.undo {
            background: #f39c12;
        }

        .action-btn.undo:hover {
            background: #e67e22;
        }

        .action-btn.toggle {
            background: #3498db;
        }

        .action-btn.toggle:hover {
            background: #2980b9;
        }

        .action-btn.fullscreen {
            background: #9b59b6;
        }

        .action-btn.fullscreen:hover {
            background: #8e44ad;
        }

        .canvas-container {
            flex: 1;
            overflow: auto;
            background: #f8f9fa;
            position: relative;
        }

        #drawingCanvas {
            display: block;
            cursor: crosshair;
            background: white;
        }

        .chat-panel {
            width: 300px;
            background: white;
            margin: 10px 10px 10px 0;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            transition: all 0.3s ease;
        }

        .chat-panel.hidden {
            transform: translateX(100%);
            opacity: 0;
            pointer-events: none;
        }

        .chat-header {
            background: #34495e;
            color: white;
            padding: 20px;
            text-align: center;
            border-radius: 15px 15px 0 0;
        }

        .chat-header h3 {
            font-size: 18px;
            margin-bottom: 5px;
        }

        .connection-status {
            font-size: 12px;
            opacity: 0.8;
            margin-bottom: 5px;
        }

        .connection-status.connected {
            color: #2ecc71;
        }

        .connection-status.connecting {
            color: #f1c40f;
        }

        .connection-status.disconnected {
            color: #e74c3c;
        }

        .online-count {
            font-size: 12px;
            opacity: 0.8;
        }

        .reset-timer {
            font-size: 11px;
            opacity: 0.7;
            margin-top: 5px;
        }

        .room-info {
            background: rgba(255,255,255,0.1);
            padding: 8px;
            margin: 10px;
            border-radius: 5px;
            font-size: 11px;
        }

        .chat-messages {
            flex: 1;
            padding: 15px;
            overflow-y: auto;
            max-height: calc(100vh - 250px);
        }

        .message {
            margin-bottom: 12px;
            padding: 8px 12px;
            border-radius: 8px;
            background: #f1f2f6;
            animation: slideIn 0.3s ease;
        }

        .message.system {
            background: #e8f5e8;
            border-left: 3px solid #27ae60;
        }

        .message.alert {
            background: #fff3cd;
            border-left: 3px solid #ffc107;
        }

        .message.join {
            background: #e3f2fd;
            border-left: 3px solid #2196f3;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .message .username {
            font-weight: bold;
            color: #2c3e50;
            font-size: 12px;
        }

        .message .text {
            margin-top: 2px;
            color: #333;
            font-size: 14px;
        }

        .message .timestamp {
            font-size: 10px;
            color: #7f8c8d;
            float: right;
        }

        .chat-input {
            padding: 15px;
            border-top: 1px solid #ecf0f1;
            border-radius: 0 0 15px 15px;
        }

        .input-group {
            display: flex;
            gap: 10px;
        }

        #usernameInput, #messageInput {
            flex: 1;
            padding: 10px;
            border: 2px solid #ecf0f1;
            border-radius: 8px;
            font-size: 14px;
            outline: none;
            transition: border-color 0.3s ease;
        }

        #usernameInput {
            max-width: 80px;
        }

        #usernameInput:focus, #messageInput:focus {
            border-color: #3498db;
        }

        #sendBtn {
            background: #27ae60;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.3s ease;
        }

        #sendBtn:hover {
            background: #219a52;
            transform: scale(1.05);
        }

        .brush-preview {
            width: 30px;
            height: 30px;
            border: 2px solid white;
            border-radius: 50%;
            background: white;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .brush-dot {
            border-radius: 50%;
            background: #333;
            transition: all 0.3s ease;
        }

        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            background: #e74c3c;
            color: white;
            padding: 15px 20px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
            z-index: 1000;
            animation: notificationSlide 0.3s ease;
        }

        .notification.success {
            background: #27ae60;
        }

        .notification.warning {
            background: #f39c12;
        }

        @keyframes notificationSlide {
            from {
                transform: translateX(100%);
                opacity: 0;
            }
            to {
                transform: translateX(0);
                opacity: 1;
            }
        }

        .setup-panel {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.8);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 2000;
        }

        .setup-modal {
            background: white;
            padding: 30px;
            border-radius: 15px;
            text-align: center;
            max-width: 500px;
            width: 90%;
        }

        .setup-modal h2 {
            margin-bottom: 20px;
            color: #2c3e50;
        }

        .setup-modal p {
            margin-bottom: 20px;
            color: #7f8c8d;
            line-height: 1.6;
        }

        .room-input {
            width: 100%;
            padding: 12px;
            border: 2px solid #ecf0f1;
            border-radius: 8px;
            font-size: 16px;
            margin-bottom: 20px;
            text-align: center;
        }

        .setup-btn {
            background: #3498db;
            color: white;
            border: none;
            padding: 12px 30px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            margin: 0 10px;
            transition: all 0.3s ease;
        }

        .setup-btn:hover {
            background: #2980b9;
        }

        .setup-btn.create {
            background: #27ae60;
        }

        .setup-btn.create:hover {
            background: #219a52;
        }

        .room-list-container {
            margin-top: 20px;
            max-height: 150px;
            overflow-y: auto;
            border: 1px solid #ecf0f1;
            border-radius: 8px;
            padding: 10px;
            background-color: #fcfcfc;
            text-align: left;
        }

        .room-list-container h4 {
            color: #555;
            margin-bottom: 10px;
            font-size: 14px;
        }

        .room-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }

        .room-list li {
            padding: 8px;
            margin-bottom: 5px;
            background-color: #eef;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.2s ease;
            font-size: 14px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .room-list li:hover {
            background-color: #ddf;
        }

        .room-list li .join-button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 12px;
        }

        .room-list li .join-button:hover {
            background-color: #2980b9;
        }

        @media (max-width: 768px) {
            .container {
                flex-direction: column;
            }
            
            .chat-panel {
                width: 100%;
                height: 250px;
                margin: 0 10px 10px 10px;
            }
            
            .chat-panel.hidden {
                transform: translateY(100%);
            }
            
            .drawing-area {
                margin: 10px 10px 0 10px;
            }
        }
    </style>
</head>
<body>
    <div class="setup-panel" id="setupPanel">
        <div class="setup-modal">
            <h2>공유 그림판 시작하기</h2>
            <p>실시간으로 함께 그림을 그리고 채팅할 수 있는 방을 만들거나 참여하세요.</p>
            
            <input type="text" class="room-input" id="roomInput" placeholder="방 이름을 입력하세요 (예: myroom123)">
            
            <div>
                <button class="setup-btn create" onclick="createRoom()">새 방 만들기</button>
                <button class="setup-btn" onclick="showJoinPanel()">방 참여하기</button>
            </div>
            
            <div class="room-list-container" id="roomListContainer" style="display: none;">
                <h4>현재 활성화된 방 목록</h4>
                <ul class="room-list" id="activeRoomList">
                    </ul>
            </div>
            
            <p style="margin-top: 20px; font-size: 12px;">
                ⚠️ 현재는 데모 버전입니다. 실제 실시간 공유를 위해서는 서버가 필요합니다.
            </p>
        </div>
    </div>

    <div class="container">
        <div class="drawing-area" id="drawingArea">
            <div class="toolbar">
                <div class="tool-group">
                    <label for="colorPicker">색상</label>
                    <input type="color" id="colorPicker" value="#000000">
                </div>
                
                <div class="tool-group">
                    <label for="brushSize">브러시 크기</label>
                    <input type="range" id="brushSize" min="1" max="50" value="5">
                    <div class="brush-preview">
                        <div class="brush-dot" id="brushDot"></div>
                    </div>
                </div>
                
                <button class="action-btn undo" id="undoBtn">내 작업 취소</button>
                <button class="action-btn toggle" id="toggleChatBtn">채팅 토글</button>
                <button class="action-btn fullscreen" id="fullscreenBtn">전체화면</button>
            </div>
            
            <div class="canvas-container">
                <canvas id="drawingCanvas" width="2000" height="1500"></canvas>
            </div>
        </div>
        
        <div class="chat-panel" id="chatPanel">
            <div class="chat-header">
                <h3>실시간 채팅</h3>
                <div class="connection-status disconnected" id="connectionStatus">연결 안됨</div>
                <div class="online-count">접속자: <span id="onlineCount">1</span>명</div>
                <div class="reset-timer" id="resetTimer">다음 리셋까지: <span id="timeLeft"></span></div>
                <div class="room-info" id="roomInfo">방: <span id="currentRoom">-</span></div>
            </div>
            
            <div class="chat-messages" id="chatMessages">
                <div class="message system">
                    <div class="username">시스템</div>
                    <div class="text">공유 그림판에 오신 것을 환영합니다!</div>
                    <div class="timestamp"></div>
                </div>
            </div>
            
            <div class="chat-input">
                <div class="input-group">
                    <input type="text" id="usernameInput" placeholder="닉네임" value="익명">
                    <input type="text" id="messageInput" placeholder="메시지를 입력하세요...">
                    <button id="sendBtn">전송</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 전역 변수들
        let currentRoom = null;
        let userId = 'user_' + Math.random().toString(36).substr(2, 9);
        let broadcastChannel = null;
        let peers = new Map(); // 실제 P2P 구현 시 사용
        let isHost = false; // 현재 탭이 방을 생성했는지 여부

        // 캔버스 설정
        const canvas = document.getElementById('drawingCanvas');
        const ctx = canvas.getContext('2d');
        const colorPicker = document.getElementById('colorPicker');
        const brushSize = document.getElementById('brushSize');
        const brushDot = document.getElementById('brushDot');
        const undoBtn = document.getElementById('undoBtn');
        const toggleChatBtn = document.getElementById('toggleChatBtn');
        const fullscreenBtn = document.getElementById('fullscreenBtn');

        // 패널 요소들
        const drawingArea = document.getElementById('drawingArea');
        const chatPanel = document.getElementById('chatPanel');
        const setupPanel = document.getElementById('setupPanel');
        const roomListContainer = document.getElementById('roomListContainer');
        const activeRoomList = document.getElementById('activeRoomList');

        // 채팅 요소들
        const chatMessages = document.getElementById('chatMessages');
        const usernameInput = document.getElementById('usernameInput');
        const messageInput = document.getElementById('messageInput');
        const sendBtn = document.getElementById('sendBtn');
        const onlineCount = document.getElementById('onlineCount');
        const resetTimer = document.getElementById('resetTimer');
        const timeLeft = document.getElementById('timeLeft');
        const connectionStatus = document.getElementById('connectionStatus');
        const currentRoomSpan = document.getElementById('currentRoom');
        const roomInput = document.getElementById('roomInput');

        // 그리기 상태
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;
        let myPaths = [];
        let allPaths = [];
        let currentPath = [];
        let connectedUsers = new Set([userId]); // 현재 연결된 사용자 ID

        // --- 방 생성 및 참여 로직 (수정된 부분) ---

        // 방 목록을 localStorage에서 불러오거나 초기화
        function getActiveRooms() {
            try {
                return JSON.parse(localStorage.getItem('activeDrawingRooms')) || {};
            } catch (e) {
                console.error("Error parsing active rooms from localStorage:", e);
                return {};
            }
        }

        // 방 목록을 localStorage에 저장
        function saveActiveRooms(rooms) {
            localStorage.setItem('activeDrawingRooms', JSON.stringify(rooms));
        }

        // 새 방 만들기 (실제 서버 없이 로컬에서만 관리)
        function createRoom() {
            const roomName = roomInput.value.trim();
            if (!roomName) {
                showNotification('방 이름을 입력해주세요!', 'warning');
                return;
            }

            let rooms = getActiveRooms();
            if (rooms[roomName]) {
                showNotification(`"${roomName}" 방이 이미 존재합니다.`, 'warning');
                return;
            }

            rooms[roomName] = { creator: userId, createdAt: Date.now() };
            saveActiveRooms(rooms);
            joinRoomWithName(roomName, true);
        }

        // 방 참여하기 버튼 클릭 시 목록 보여주기
        function showJoinPanel() {
            updateRoomList();
            roomListContainer.style.display = 'block';
        }

        // 기존 방 참여
        function joinRoom(roomName) {
            if (!roomName) {
                roomName = roomInput.value.trim();
            }

            if (!roomName) {
                showNotification('방 이름을 입력하거나 목록에서 선택해주세요!', 'warning');
                return;
            }

            let rooms = getActiveRooms();
            if (!rooms[roomName]) {
                showNotification(`"${roomName}" 방이 존재하지 않습니다.`, 'warning');
                return;
            }

            joinRoomWithName(roomName, false);
        }

        // 방 목록 업데이트
        function updateRoomList() {
            const rooms = getActiveRooms();
            activeRoomList.innerHTML = ''; // 기존 목록 초기화

            const roomNames = Object.keys(rooms);
            if (roomNames.length === 0) {
                activeRoomList.innerHTML = '<li>활성화된 방이 없습니다. 새 방을 만들어보세요!</li>';
            } else {
                roomNames.forEach(name => {
                    const listItem = document.createElement('li');
                    listItem.innerHTML = `
                        <span>${name}</span>
                        <button class="join-button" onclick="joinRoom('${name}')">참여</button>
                    `;
                    activeRoomList.appendChild(listItem);
                });
            }
        }

        // 실제 방 참여 처리
        function joinRoomWithName(roomName, isCreator) {
            currentRoom = roomName;
            isHost = isCreator;
            currentRoomSpan.textContent = roomName;
            
            // BroadcastChannel로 같은 브라우저 내 탭들과 통신
            if (broadcastChannel) {
                broadcastChannel.close(); // 기존 채널 닫기
            }
            
            // 새 BroadcastChannel 연결
            broadcastChannel = new BroadcastChannel('drawing_room_' + roomName);
            broadcastChannel.addEventListener('message', handleBroadcastMessage);
            
            // 설정 패널 숨기기
            setupPanel.style.display = 'none';
            
            // 연결 상태 업데이트
            updateConnectionStatus('connecting');
            
            // 참여 알림 전송 (BroadcastChannel을 통해 다른 탭에 알림)
            broadcastMessage({
                type: 'user_joined',
                userId: userId,
                username: usernameInput.value || '익명',
                timestamp: Date.now()
            });
            
            // 잠시 후 연결됨으로 상태 변경
            setTimeout(() => {
                updateConnectionStatus('connected');
                addSystemMessage(`방 "${roomName}"에 ${isCreator ? '생성하여' : ''} 참여했습니다.`);
                
                if (isCreator) {
                    addSystemMessage('다른 사용자들이 같은 방 이름으로 참여하면 실시간으로 공유됩니다.');
                }
            }, 1000);
        }

        // 사용자가 페이지를 닫거나 새로고침할 때 BroadcastChannel 닫기
        window.addEventListener('beforeunload', () => {
            if (broadcastChannel) {
                broadcastMessage({
                    type: 'user_left',
                    userId: userId,
                    username: usernameInput.value || '익명',
                    timestamp: Date.now()
                });
                broadcastChannel.close();
            }
            // 방 생성자가 나갈 경우, 해당 방을 목록에서 제거 (선택 사항)
            if (isHost && currentRoom) {
                let rooms = getActiveRooms();
                delete rooms[currentRoom];
                saveActiveRooms(rooms);
            }
        });


        // --- 기존 BroadcastChannel 메시지 처리 로직 (변동 없음) ---
        function handleBroadcastMessage(event) {
            const data = event.data;
            
            switch (data.type) {
                case 'user_joined':
                    if (data.userId !== userId) {
                        connectedUsers.add(data.userId);
                        updateOnlineCount();
                        addJoinMessage(`${data.username}님이 참여했습니다.`);
                    }
                    break;
                    
                case 'user_left':
                    if (data.userId !== userId) {
                        connectedUsers.delete(data.userId);
                        updateOnlineCount();
                        addJoinMessage(`${data.username}님이 나갔습니다.`);
                    }
                    break;
                    
                case 'chat_message':
                    if (data.userId !== userId) {
                        addMessage(data.username, data.message, false, data.timestamp);
                    }
                    break;
                    
                case 'drawing_data':
                    if (data.userId !== userId) {
                        drawReceivedPath(data.path);
                    }
                    break;
                    
                case 'canvas_clear':
                    if (data.userId !== userId) {
                        ctx.clearRect(0, 0, canvas.width, canvas.height);
                        allPaths = [];
                        addSystemMessage('다른 사용자가 캔버스를 지웠습니다.');
                    }
                    break;
            }
        }

        // 메시지 브로드캐스트
        function broadcastMessage(data) {
            if (broadcastChannel) {
                broadcastChannel.postMessage(data);
            }
        }

        // 연결 상태 업데이트
        function updateConnectionStatus(status) {
            connectionStatus.className = `connection-status ${status}`;
            switch (status) {
                case 'connected':
                    connectionStatus.textContent = '연결됨';
                    break;
                case 'connecting':
                    connectionStatus.textContent = '연결중...';
                    break;
                case 'disconnected':
                    connectionStatus.textContent = '연결 안됨';
                    break;
            }
        }

        // 브러시 미리보기 업데이트
        function updateBrushPreview() {
            const size = Math.max(2, Math.min(brushSize.value / 2, 15));
            brushDot.style.width = size + 'px';
            brushDot.style.height = size + 'px';
            brushDot.style.backgroundColor = colorPicker.value;
        }

        // 캔버스 초기 설정
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        // 마우스 이벤트
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mouseout', stopDrawing);

        // 터치 이벤트
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchmove', handleTouchMove);
        canvas.addEventListener('touchend', stopDrawing);

        function handleTouchStart(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const x = touch.clientX - rect.left;
            const y = touch.clientY - rect.top;
            startDrawing({offsetX: x, offsetY: y});
        }

        function handleTouchMove(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const x = touch.clientX - rect.left;
            const y = touch.clientY - rect.top;
            draw({offsetX: x, offsetY: y});
        }

        function startDrawing(e) {
            isDrawing = true;
            lastX = e.offsetX;
            lastY = e.offsetY;
            
            currentPath = [{
                x: e.offsetX,
                y: e.offsetY,
                color: colorPicker.value,
                size: brushSize.value,
                isMyPath: true
            }];
            
            ctx.beginPath();
            ctx.moveTo(e.offsetX, e.offsetY);
            ctx.lineWidth = brushSize.value;
            ctx.strokeStyle = colorPicker.value;
        }

        function draw(e) {
            if (!isDrawing) return;
            
            ctx.lineWidth = brushSize.value;
            ctx.strokeStyle = colorPicker.value;
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(e.offsetX, e.offsetY);
            ctx.stroke();
            
            currentPath.push({
                x: e.offsetX,
                y: e.offsetY,
                color: colorPicker.value,
                size: brushSize.value,
                isMyPath: true
            });
            
            lastX = e.offsetX;
            lastY = e.offsetY;
        }

        function stopDrawing() {
            if (isDrawing && currentPath.length > 0) {
                isDrawing = false;
                myPaths.push([...currentPath]);
                allPaths.push([...currentPath]);
                
                // 그리기 데이터 브로드캐스트
                broadcastMessage({
                    type: 'drawing_data',
                    userId: userId,
                    path: currentPath,
                    timestamp: Date.now()
                });
                
                currentPath = [];
                updateUndoButton();
            }
        }

        // 받은 그리기 데이터 그리기
        function drawReceivedPath(path) {
            if (path.length === 0) return;
            
            ctx.beginPath();
            ctx.moveTo(path[0].x, path[0].y);
            ctx.strokeStyle = path[0].color;
            ctx.lineWidth = path[0].size;
            
            for (let i = 1; i < path.length; i++) {
                ctx.lineTo(path[i].x, path[i].y);
            }
            ctx.stroke();
            
            // 받은 경로도 전체 경로에 추가
            allPaths.push(path);
        }

        // 실행 취소
        function undoMyLastAction() {
            if (myPaths.length > 0) {
                const removedPath = myPaths.pop();
                const index = allPaths.findIndex(path => 
                    path.length === removedPath.length && 
                    path[0] && removedPath[0] &&
                    path[0].x === removedPath[0].x && 
                    path[0].y === removedPath[0].y &&
                    path[0].isMyPath
                );
                if (index !== -1) {
                    allPaths.splice(index, 1);
                }
                redrawCanvas();
                updateUndoButton();
                showNotification('내 마지막 작업이 취소되었습니다.', 'success');
            }
        }

        function updateUndoButton() {
            if (myPaths.length === 0) {
                undoBtn.classList.add('disabled');
                undoBtn.disabled = true;
            } else {
                undoBtn.classList.remove('disabled');
                undoBtn.disabled = false;
            }
        }

        function redrawCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            allPaths.forEach(path => {
                if (path.length > 0) {
                    ctx.beginPath();
                    ctx.moveTo(path[0].x, path[0].y);
                    ctx.strokeStyle = path[0].color;
                    ctx.lineWidth = path[0].size;
                    
                    for (let i = 1; i < path.length; i++) {
                        ctx.lineTo(path[i].x, path[i].y);
                    }
                    ctx.stroke();
                }
            });
        }

        // 채팅 토글
        function toggleChat() {
            chatPanel.classList.toggle('hidden');
            if (chatPanel.classList.contains('hidden')) {
                toggleChatBtn.textContent = '채팅 보기';
            } else {
                toggleChatBtn.textContent = '채팅 숨기기';
            }
        }

        // 전체화면 토글
        function toggleFullscreen() {
            chatPanel.classList.toggle('hidden');
            drawingArea.classList.toggle('fullscreen');
            
            if (drawingArea.classList.contains('fullscreen')) {
                fullscreenBtn.textContent = '일반 화면';
            } else {
                fullscreenBtn.textContent = '전체 화면';
            }
        }

        // 시간 관련 함수들
        function getTimeUntilMidnight() {
            const now = new Date();
            const midnight = new Date();
            midnight.setHours(24, 0, 0, 0);
            return midnight.getTime() - now.getTime();
        }

        function formatTime(ms) {
            const hours = Math.floor(ms / (1000 * 60 * 60));
            const minutes = Math.floor((ms % (1000 * 60 * 60)) / (1000 * 60));
            const seconds = Math.floor((ms % (1000 * 60)) / 1000);
            return `${hours}시간 ${minutes}분 ${seconds}초`;
        }

        function updateResetTimer() {
            const timeRemaining = getTimeUntilMidnight();
            timeLeft.textContent = formatTime(timeRemaining);
            
            if (timeRemaining <= 60000 && timeRemaining > 55000) {
                showNotification('1분 후 그림판이 리셋됩니다!', 'warning');
            }
            
            if (timeRemaining <= 1000) {
                resetCanvas();
            }
        }

        function resetCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            allPaths = [];
            myPaths = [];
            updateUndoButton();
            addSystemMessage('자정이 되어 그림판이 초기화되었습니다!');
            showNotification('그림판이 리셋되었습니다!', 'success');
            
            // 캔버스 클리어 브로드캐스트
            broadcastMessage({
                type: 'canvas_clear',
                userId: userId,
                timestamp: Date.now()
            });
        }

        // 알림 표시
        function showNotification(message, type = 'info', duration = 3000) {
            const notification = document.createElement('div');
            notification.className = `notification ${type}`;
            notification.textContent = message;
            document.body.appendChild(notification);

            setTimeout(() => {
                notification.style.opacity = '0';
                notification.style.transform = 'translateX(100%)';
                notification.addEventListener('transitionend', () => {
                    notification.remove();
                });
            }, duration);
        }

        // 채팅 메시지 추가
        function addMessage(username, text, isMe = false, timestamp = Date.now(), type = '') {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${type}`;
            if (isMe) messageDiv.classList.add('my-message'); // 필요하면 CSS 추가

            const usernameSpan = document.createElement('div');
            usernameSpan.className = 'username';
            usernameSpan.textContent = username;

            const textDiv = document.createElement('div');
            textDiv.className = 'text';
            textDiv.textContent = text;

            const timestampSpan = document.createElement('div');
            timestampSpan.className = 'timestamp';
            const date = new Date(timestamp);
            timestampSpan.textContent = `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`;

            messageDiv.appendChild(usernameSpan);
            messageDiv.appendChild(textDiv);
            messageDiv.appendChild(timestampSpan);
            chatMessages.appendChild(messageDiv);

            chatMessages.scrollTop = chatMessages.scrollHeight; // 스크롤 하단으로
        }

        function addSystemMessage(text) {
            addMessage('시스템', text, false, Date.now(), 'system');
        }

        function addJoinMessage(text) {
            addMessage('알림', text, false, Date.now(), 'join');
        }

        // 온라인 접속자 수 업데이트 (BroadcastChannel 기반으로 흉내)
        function updateOnlineCount() {
            onlineCount.textContent = connectedUsers.size;
        }

        // 이벤트 리스너 등록
        colorPicker.addEventListener('input', updateBrushPreview);
        brushSize.addEventListener('input', updateBrushPreview);
        undoBtn.addEventListener('click', undoMyLastAction);
        toggleChatBtn.addEventListener('click', toggleChat);
        fullscreenBtn.addEventListener('click', toggleFullscreen);

        sendBtn.addEventListener('click', () => {
            const username = usernameInput.value.trim() || '익명';
            const message = messageInput.value.trim();
            if (message) {
                addMessage(username, message, true);
                broadcastMessage({
                    type: 'chat_message',
                    userId: userId,
                    username: username,
                    message: message,
                    timestamp: Date.now()
                });
                messageInput.value = '';
            }
        });

        messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                sendBtn.click();
            }
        });

        // 초기화 함수 호출
        updateBrushPreview();
        updateUndoButton();
        updateOnlineCount();
        setInterval(updateResetTimer, 1000); // 1초마다 타이머 업데이트

        // 페이지 로드 시 방 목록 업데이트
        document.addEventListener('DOMContentLoaded', updateRoomList);

    </script>
</body>
</html>
