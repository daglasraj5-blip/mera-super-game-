<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced AI Ludo Game</title>
    <style>
        :root {
            --red: #ff3e3e;
            --green: #2ecc71;
            --yellow: #f1c40f;
            --blue: #3498db;
            --board-bg: #fff;
            --border-color: #333;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #2c3e50;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            color: white;
        }

        h1 { margin-bottom: 10px; }

        /* Game Container */
        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
        }

        /* Ludo Board Styling */
        .ludo-board {
            width: 90vw;
            max-width: 500px;
            height: 90vw;
            max-height: 500px;
            background: var(--board-bg);
            border: 5px solid var(--border-color);
            display: grid;
            grid-template-columns: repeat(15, 1fr);
            grid-template-rows: repeat(15, 1fr);
            position: relative;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
        }

        /* Cells */
        .cell {
            border: 1px solid #ccc;
            position: relative;
        }

        .cell.red-bg { background-color: var(--red); }
        .cell.green-bg { background-color: var(--green); }
        .cell.yellow-bg { background-color: var(--yellow); }
        .cell.blue-bg { background-color: var(--blue); }
        
        /* Safe Zones / Homes */
        .home-area {
            grid-column: 2 / span 4;
            grid-row: 2 / span 4;
            border: 2px solid #333;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            align-items: center;
            gap: 10px;
        }

        .red-home { background-color: var(--red); }
        .green-home { background-color: var(--green); }
        .yellow-home { background-color: var(--yellow); }
        .blue-home { background-color: var(--blue); }

        .center-home {
            grid-column: 7 / span 3;
            grid-row: 7 / span 3;
            background: conic-gradient(
                var(--red) 0deg 90deg, 
                var(--blue) 90deg 180deg, 
                var(--yellow) 180deg 270deg, 
                var(--green) 270deg 360deg
            );
            border: 2px solid #333;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            color: white;
            text-shadow: 1px 1px 2px black;
        }

        /* Path Colors */
        .path-red { background-color: var(--red); opacity: 0.3; }
        .path-green { background-color: var(--green); opacity: 0.3; }
        .path-yellow { background-color: var(--yellow); opacity: 0.3; }
        .path-blue { background-color: var(--blue); opacity: 0.3; }

        /* Star and Triangles (Visual only, simple CSS) */
        .safe-star::after {
            content: "★";
            position: absolute;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            color: #333;
            font-size: 12px;
        }

        /* Pieces */
        .piece {
            width: 60%;
            height: 60%;
            border-radius: 50%;
            border: 2px solid white;
            box-shadow: 0 2px 5px rgba(0,0,0,0.5);
            position: absolute;
            top: 20%;
            left: 20%;
            transition: all 0.3s ease;
            z-index: 10;
            cursor: pointer;
        }

        .piece.red { background-color: var(--red); }
        .piece.green { background-color: var(--green); }
        .piece.yellow { background-color: var(--yellow); }
        .piece.blue { background-color: var(--blue); }

        .piece.active {
            border: 3px solid white;
            box-shadow: 0 0 10px white;
            animation: pulse 1s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }

        /* Controls */
        .controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: rgba(0,0,0,0.3);
            padding: 20px;
            border-radius: 10px;
            width: 90%;
            max-width: 500px;
        }

        .dice-area {
            display: flex;
            align-items: center;
            gap: 20px;
        }

        #dice-btn {
            padding: 10px 30px;
            font-size: 18px;
            background: #e74c3c;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
        }

        #dice-btn:disabled {
            background: #7f8c8d;
            cursor: not-allowed;
        }

        #dice-value {
            font-size: 24px;
            font-weight: bold;
            width: 40px;
            height: 40px;
            background: white;
            color: black;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 5px;
        }

        .status {
            margin-top: 10px;
            font-size: 18px;
        }
        
        .turn-indicator {
            display: inline-block;
            width: 15px;
            height: 15px;
            border-radius: 50%;
            margin-right: 5px;
        }

    </style>
</head>
<body>

    <h1>AI Ludo Game</h1>

    <div class="game-container">
        <div class="ludo-board" id="board">
            <!-- Grid will be generated by JS -->
            <div class="home-area red-home" id="red-home"></div>
            <div class="home-area green-home" id="green-home"></div>
            <div class="home-area yellow-home" id="yellow-home"></div>
            <div class="home-area blue-home" id="blue-home"></div>
            <div class="center-home">LUDO</div>
        </div>

        <div class="controls">
            <div class="dice-area">
                <div id="dice-value">-</div>
                <button id="dice-btn" onclick="rollDice()">Roll Dice</button>
            </div>
            <div class="status" id="status-text">Red's Turn (You)</div>
        </div>
    </div>

    <script>
        // --- Game Configuration ---
        const board = document.getElementById('board');
        const statusText = document.getElementById('status-text');
        const diceBtn = document.getElementById('dice-btn');
        const diceValueEl = document.getElementById('dice-value');

        // Coordinates for the main path (1-52)
        // 0,0 is top left. Grid is 15x15.
        // This is a simplified coordinate mapping for visual pieces
        // In a real grid system, we calculate top/left % based on 15x15
        const pathCoordinates = [
            // Red Start Path (Up to 5) -> Right (5 to 10) -> Up (10 to 15) -> Right
            {r:6, c:1}, {r:6, c:2}, {r:6, c:3}, {r:6, c:4}, {r:6, c:5}, // 1-5
            {r:5, c:6}, {r:4, c:6}, {r:3, c:6}, {r:2, c:6}, {r:1, c:6}, {r:1, c:7}, // 6-11
            {r:1, c:8}, {r:2, c:8}, {r:3, c:8}, {r:4, c:8}, {r:5, c:8}, // 12-16
            {r:6, c:9}, {r:6, c:10}, {r:6, c:11}, {r:6, c:12}, {r:6, c:13}, // 17-21
            {r:7, c:14}, // 22
            {r:8, c:14}, // 23
            {r:8, c:13}, {r:8, c:12}, {r:8, c:11}, {r:8, c:10}, {r:8, c:9}, // 24-28
            {r:9, c:8}, {r:10, c:8}, {r:11, c:8}, {r:12, c:8}, {r:13, c:8}, // 29-33
            {r:14, c:8}, {r:14, c:7}, {r:14, c:6}, // 34-36
            {r:13, c:6}, {r:12, c:6}, {r:11, c:6}, {r:10, c:6}, {r:9, c:6}, // 37-41
            {r:8, c:5}, {r:8, c:4}, {r:8, c:3}, {r:8, c:2}, {r:8, c:1}, // 42-46
            {r:7, c:1} // 47 (End/Start loop point, next is 48)
        ];

        // Home Stretch Paths
        const homePaths = {
            red: [{r:7, c:2}, {r:7, c:3}, {r:7, c:4}, {r:7, c:5}, {r:7, c:6}, {r:6, c:7}], // Ends at center
            green: [{r:2, c:7}, {r:3, c:7}, {r:4, c:7}, {r:5, c:7}, {r:6, c:7}, {r:7, c:8}], // Ends at center
            yellow: [{r:7, c:12}, {r:7, c:11}, {r:7, c:10}, {r:7, c:9}, {r:7, c:8}, {r:8, c:7}], // Ends at center
            blue: [{r:12, c:7}, {r:11, c:7}, {r:10, c:7}, {r:9, c:7}, {r:8, c:7}, {r:7, c:6}] // Ends at center
        };

        // Starting positions in grid (0-14)
        const startPositions = {
            red: {r: 6, c: 1}, // Visual start on path
            green: {r: 1, c: 8},
            yellow: {r: 8, c: 13},
            blue: {r: 13, c: 6}
        };

        // House positions (where pieces sit when not on board)
        const houseOffsets = [
            {x: 0, y: 0}, {x: 50, y: 0}, {x: 0, y: 50}, {x: 50, y: 50} // TopLeft, TopRight, BotLeft, BotRight relative to home div
        ];

        // --- Game State ---
        const players = ['red', 'green', 'yellow', 'blue'];
        let currentPlayerIndex = 0; // 0=Red (Human), 1,2,3 = AI
        let currentDice = 0;
        let gameState = 'WAITING_FOR_ROLL'; // WAITING_FOR_ROLL, WAITING_FOR_MOVE, ANIMATING, GAME_OVER

        class Player {
            constructor(color, index) {
                this.color = color;
                this.index = index; // 0-3
                this.pieces = [0, 0, 0, 0]; // 0 = House, 1-51 = Path steps, 100 = Finished
                this.pieceElements = [];
            }

            initPieceElements() {
                const homeDiv = document.getElementById(`${this.color}-home`);
                for(let i=0; i<4; i++) {
                    const piece = document.createElement('div');
                    piece.className = `piece ${this.color}`;
                    piece.id = `${this.color}-piece-${i}`;
                    piece.onclick = () => handlePieceClick(this.color, i);
                    
                    // Initial placement in house
                    this.updatePieceVisual(i);
                    homeDiv.appendChild(piece);
                    this.pieceElements.push(piece);
                }
            }

            updatePieceVisual(pieceIndex) {
                const piece = this.pieceElements[pieceIndex];
                const pos = this.pieces[pieceIndex];
                const parent = piece.parentElement;

                if (pos === 0) {
                    // In House
                    parent.appendChild(piece);
                    piece.style.position = 'relative';
                    piece.style.top = '0';
                    piece.style.left = '0';
                    piece.style.transform = 'translate(0,0)';
                } else if (pos === 100) {
                    // Finished (Center)
                    parent.appendChild(piece);
                    piece.style.position = 'relative';
                } else {
                    // On Board
                    board.appendChild(piece);
                    piece.style.position = 'absolute';
                    
                    let r, c;
                    if (pos <= 51) {
                        const coord = pathCoordinates[pos - 1];
                        r = coord.r;
                        c = coord.c;
                    } else {
                        // Home stretch
                        const homeCoord = homePaths[this.color][pos - 52];
                        r = homeCoord.r;
                        c = homeCoord.c;
                    }

                    // Convert grid (0-14) to percentage
                    // Add slight offset for multiple pieces on same cell
                    const percentR = (r * 100) / 15;
                    const percentC = (c * 100) / 15;
                    
                    // Simple offset logic based on piece index to prevent total overlap
                    // (In a full game, you'd sort by ID)
                    let offsetX = (pieceIndex % 2) * 10; 
                    let offsetY = Math.floor(pieceIndex / 2) * 10;

                    piece.style.top = `calc(${percentR}% + ${offsetY}px)`;
                    piece.style.left = `calc(${percentC}% + ${offsetX}px)`;
                    piece.style.transform = 'translate(-50%, -50%)';
                }
            }
        }

        // --- Initialization ---
        const playerObjects = {};
        players.forEach((color, idx) => {
            playerObjects[color] = new Player(color, idx);
        });

        // Generate Grid Visuals (Static board parts)
        function createBoardVisuals() {
            // Helper to create cells
            const createCell = (r, c, className = '') => {
                const div = document.createElement('div');
                div.className = `cell ${className}`;
                div.style.gridRow = r + 1;
                div.style.gridColumn = c + 1;
                return div;
            };

            // Red Path
            for(let i=1; i<=5; i++) board.appendChild(createCell(6, i, 'path-red'));
            // Green Path
            for(let i=1; i<=5; i++) board.appendChild(createCell(i, 8, 'path-green'));
            // Yellow Path
            for(let i=9; i<=13; i++) board.appendChild(createCell(8, i, 'path-yellow'));
            // Blue Path
            for(let i=9; i<=13; i++) board.appendChild(createCell(i, 6, 'path-blue'));
            
            // Start Squares (Safe)
            const startSquares = [
                {r:6, c:6, color:'red'}, {r:2, c:6, color:'green'}, 
                {r:8, c:8, color:'yellow'}, {r:12, c:8, color:'blue'},
                {r:6, c:2, color:'red'}, {r:2, c:12, color:'green'},
                {r:12, c:2, color:'blue'}, {r:12, c:12, color:'yellow'}
            ];
            
            startSquares.forEach(s => {
                const div = createCell(s.r, s.c);
                div.classList.add('safe-star');
                board.appendChild(div);
            });
        }

        createBoardVisuals();
        players.forEach(c => playerObjects[c].initPieceElements());

        // --- Game Logic ---

        function rollDice() {
            if (gameState !== 'WAITING_FOR_ROLL') return;

            diceBtn.disabled = true;
            
            // Animation
            let rolls = 0;
            const interval = setInterval(() => {
                diceValueEl.innerText = Math.floor(Math.random() * 6) + 1;
                rolls++;
                if (rolls > 10) {
                    clearInterval(interval);
                    finalizeDice();
                }
            }, 50);
        }

        function finalizeDice() {
            currentDice = Math.floor(Math.random() * 6) + 1;
            diceValueEl.innerText = currentDice;
            
            checkPossibleMoves();
        }

        function checkPossibleMoves() {
            const player = playerObjects[players[currentPlayerIndex]];
            const color = player.color;
            let movablePieces = [];

            player.pieces.forEach((pos, idx) => {
                if (pos === 100) return; // Already won

                // Rule: Need 6 to start
                if (pos === 0 && currentDice !== 6) return;
                
                // Rule: Can't move if sum > 57 (51 path + 6 home stretch = 57)
                if (pos > 0 && pos + currentDice > 57) return;

                movablePieces.push(idx);
            });

            if (movablePieces.length === 0) {
                statusText.innerText = `No moves for ${color.toUpperCase()}`;
                setTimeout(nextTurn, 1000);
            } else {
                gameState = 'WAITING_FOR_MOVE';
                
                if (currentPlayerIndex === 0) {
                    // Human Turn: Highlight pieces
                    statusText.innerText = `Your Turn! Roll: ${currentDice}`;
                    movablePieces.forEach(idx => {
                        player.pieceElements[idx].classList.add('active');
                    });
                } else {
                    // AI Turn
                    statusText.innerText = `${color.toUpperCase()} is thinking...`;
                    setTimeout(() => makeAIMove(movablePieces), 1000);
                }
            }
        }

        function handlePieceClick(color, idx) {
            if (gameState !== 'WAITING_FOR_MOVE' || currentPlayerIndex !== 0) return;
            if (color !== players[currentPlayerIndex]) return;

            // Check if valid
            const player = playerObjects[color];
            const pos = player.pieces[idx];
            
            if (pos === 0 && currentDice !== 6) return;
            if (pos > 0 && pos + currentDice > 57) return;

            movePiece(color, idx);
        }

        function makeAIMove(movablePieces) {
            const player = playerObjects[players[currentPlayerIndex]];
            const color = player.color;
            
            // Simple AI Logic
            let chosenIdx = movablePieces[0];

            // Priority 1: Capture opponent (if move lands on opponent)
            // Priority 2: Move piece out of house (if 6)
            // Priority 3: Move closest piece to home
            // Priority 4: Random

            // Check for captures
            for (let idx of movablePieces) {
                let currentPos = player.pieces[idx];
                let newPos = (currentPos === 0) ? 1 : currentPos + currentDice;
                
                // Simplified capture check logic would go here
                // For this basic version, we just random move or move closest
            }

            // Let's just pick the piece closest to winning or random
            // Improved: Sort by position descending (highest wins)
            movablePieces.sort((a, b) => player.pieces[b] - player.pieces[a]);
            
            chosenIdx = movablePieces[0]; // Move the one furthest ahead

            movePiece(color, chosenIdx);
        }

        function movePiece(color, pieceIdx) {
            const player = playerObjects[color];
            
            // Remove active highlights
            player.pieceElements.forEach(el => el.classList.remove('active'));

            const oldPos = player.pieces[pieceIdx];
            let newPos;

            if (oldPos === 0) {
                newPos = 1; // Enters board at step 1
            } else {
                newPos = oldPos + currentDice;
            }

            // Check Win
            if (newPos === 57) {
                player.pieces[pieceIdx] = 100; // Finished
            } else {
                player.pieces[pieceIdx] = newPos;
            }

            // Check Collision (Simple implementation: if lands on opponent, send them home)
            if (newPos !== 100 && newPos <= 51) {
                players.forEach(oppColor => {
                    if (oppColor !== color) {
                        const oppPlayer = playerObjects[oppColor];
                        oppPlayer.pieces.forEach((oppPos, oppIdx) => {
                            if (oppPos === newPos) {
                                // Check if safe spot (start points are usually safe)
                                // Start points: Red(1), Green(14), Yellow(27), Blue(40) approx in path logic
                                // For simplicity in this code, we just kill them unless it's a start point
                                if (newPos !== 1 && newPos !== 14 && newPos !== 27 && newPos !== 40) {
                                    oppPlayer.pieces[oppIdx] = 0; // Send back to house
                                    oppPlayer.updatePieceVisual(oppIdx);
                                }
                            }
                        });
                    }
                });
            }

            player.updatePieceVisual(pieceIdx);

            // Check if player rolled 6 to get another turn
            if (currentDice === 6) {
                gameState = 'WAITING_FOR_ROLL';
                statusText.innerText = `${color.toUpperCase()} rolled 6! Roll again.`;
                diceBtn.disabled = (currentPlayerIndex !== 0); // Enable only if human
            } else {
                nextTurn();
            }
        }

        function nextTurn() {
            // Check win condition
            const currentPlayer = playerObjects[players[currentPlayerIndex]];
            if (currentPlayer.pieces.every(p => p === 100)) {
                alert(`${currentPlayer.color.toUpperCase()} WINS!`);
                gameState = 'GAME_OVER';
                return;
            }

            currentPlayerIndex = (currentPlayerIndex + 1) % 4;
            gameState = 'WAITING_FOR_ROLL';
            
            const nextColor = players[currentPlayerIndex];
            statusText.innerText = `${nextColor.toUpperCase()}'s Turn`;
            
            if (currentPlayerIndex === 0) {
                diceBtn.disabled = false;
                statusText.innerText += " (Your Turn)";
            } else {
                diceBtn.disabled = true;
                // Auto roll for AI
                setTimeout(rollDice, 500);
            }
        }

    </script>
</body>
</html>
