<!DOCTYPE html>  
<html lang="ko">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>반응속도 챔피언십</title>  
    <style>  
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Pretendard', sans-serif; user-select: none; }  
        body { background-color: #0f172a; color: #f8fafc; display: flex; align-items: center; justify-content: center; height: 100vh; overflow: hidden; }  
          
        #screen { width: 100vw; height: 100vh; display: flex; flex-direction: column; align-items: center; justify-content: center; cursor: pointer; transition: background-color 0.1s ease; text-align: center; padding: 20px; }  
        .waiting { background-color: #1e293b; }  
        .ready { background-color: #dc2626; }  
        .now { background-color: #16a34a; }

        /* 입력창 스타일 */  
        .input-container { background: rgba(0,0,0,0.5); padding: 30px; border-radius: 20px; backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.2); }  
        input { padding: 10px; font-size: 1.2rem; border-radius: 8px; border: none; margin-bottom: 15px; text-align: center; }  
        button { padding: 10px 20px; font-size: 1.1rem; cursor: pointer; background: #6366f1; color: white; border: none; border-radius: 8px; }

        /* 랭킹 보드 */  
        #rank-board { position: absolute; top: 20px; right: 20px; background: rgba(0,0,0,0.6); padding: 15px; border-radius: 12px; width: 220px; border: 1px solid rgba(255,255,255,0.1); }  
        .rank-item { display: flex; justify-content: space-between; margin-bottom: 5px; font-size: 0.9rem; }

        h1 { font-size: 2.5rem; margin-bottom: 10px; }  
        p { font-size: 1.2rem; opacity: 0.8; }  
    </style>  
</head>  
<body>

    <div id="rank-board">  
        <div style="text-align: center; font-weight: bold; margin-bottom: 10px;">🏆 실시간 순위 (평균)</div>  
        <div id="rank-list"></div>  
    </div>

    <div id="screen" class="waiting">  
        <!-- 이름 입력 영역 -->  
        <div id="setup" class="input-container">  
            <h1>반응속도 테스트</h1>  
            <p style="margin-bottom: 20px;">이름을 입력하고 도전을 시작하세요!</p>  
            <input type="text" id="userName" placeholder="이름 입력" maxlength="10"><br>  
            <button onclick="startGame()">게임 시작</button>  
        </div>  
          
        <!-- 게임 텍스트 (초기 숨김) -->  
        <div id="game-ui" style="display: none;">  
            <h1 id="main-text">준비...</h1>  
            <p id="sub-text">화면을 클릭하세요!</p>  
        </div>  
    </div>

    <script>  
        const screen = document.getElementById('screen');  
        const setup = document.getElementById('setup');  
        const gameUI = document.getElementById('game-ui');  
        const mainText = document.getElementById('main-text');  
        const subText = document.getElementById('sub-text');  
        const rankList = document.getElementById('rank-list');

        let state = 'SETUP';   
        let currentUser = '';  
        let userRecords = [];  
        let startTime, timerId;  
        let allPlayers = []; // {name: '노지환', avg: 250}

        function startGame() {  
            const nameInput = document.getElementById('userName');  
            if (!nameInput.value.trim()) { alert('이름을 입력해주세요!'); return; }  
              
            currentUser = nameInput.value.trim();  
            userRecords = [];  
            setup.style.display = 'none';  
            gameUI.style.display = 'block';  
            prepareRound();  
        }

        function prepareRound() {  
            state = 'READY';  
            screen.className = 'ready';  
            mainText.innerText = '기다리세요...';  
            subText.innerText = `현재 ${userRecords.length + 1} / 3회차 시도 중`;

            const randomDelay = Math.floor(Math.random() * 3000) + 2000;  
            timerId = setTimeout(() => {  
                state = 'NOW';  
                screen.className = 'now';  
                mainText.innerText = '지금 클릭!!!';  
                startTime = performance.now();  
            }, randomDelay);  
        }

        screen.addEventListener('mousedown', () => {  
            if (state === 'READY') {  
                clearTimeout(timerId);  
                state = 'WAITING';  
                screen.className = 'waiting';  
                mainText.innerText = '성급했습니다! 😜';  
                subText.innerText = '다시 시작하려면 클릭하세요.';  
            } else if (state === 'NOW') {  
                const reactionTime = Math.round(performance.now() - startTime);  
                userRecords.push(reactionTime);  
                state = 'WAITING';  
                screen.className = 'waiting';  
                mainText.innerText = `${reactionTime} ms`;  
                  
                if (userRecords.length < 3) {  
                    subText.innerText = '잘하셨습니다! 다음 라운드를 위해 클릭하세요.';  
                } else {  
                    finishGame();  
                }  
            } else if (state === 'WAITING' && gameUI.style.display === 'block') {  
                prepareRound();  
            }  
        });

        function finishGame() {  
            const avg = Math.round(userRecords.reduce((a, b) => a + b, 0) / 3);  
            mainText.innerText = `평균: ${avg} ms`;  
            subText.innerText = '기록이 저장되었습니다. 다음 사람을 위해 클릭하세요!';  
              
            // 랭킹 데이터 업데이트  
            const playerIndex = allPlayers.findIndex(p => p.name === currentUser);  
            if (playerIndex > -1) allPlayers[playerIndex].avg = avg;  
            else allPlayers.push({name: currentUser, avg: avg});  
              
            updateRankBoard();  
            state = 'FINISHED';  
        }

        function updateRankBoard() {  
            allPlayers.sort((a, b) => a.avg - b.avg); // 낮은 시간순 정렬  
            rankList.innerHTML = allPlayers.map((p, i) =>   
                `<div class="rank-item"><span>${i+1}위 ${p.name}</span><b>${p.avg}ms</b></div>`  
            ).join('');  
        }

        // 게임 종료 후 다시 이름 입력창으로  
        screen.addEventListener('dblclick', () => {  
            if (state === 'FINISHED') {  
                state = 'SETUP';  
                setup.style.display = 'block';  
                gameUI.style.display = 'none';  
                screen.className = 'waiting';  
                document.getElementById('userName').value = '';  
            }  
        });  
    </script>  
</body>  
</html>  
