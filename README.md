# GoemtryDash-SweetChildEdition1
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Geometry Dash: Sweet Child Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Tone.js for the rhythmic music engine -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Chivo:wght@900&display=swap" rel="stylesheet">
    <style>
        /* Global Styles for GD Aesthetic */
        body {
            font-family: 'Chivo', sans-serif;
            background-color: #000000;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 0;
            margin: 0;
            user-select: none;
            overflow: hidden;
        }

        .game-container {
            width: 100%;
            height: 100vh;
            max-width: 600px;
            background-color: #110022; /* Darker than black background */
            box-shadow: 0 0 50px rgba(255, 107, 184, 0.4); /* Neon Pink/Rose Glow */
            display: flex;
            flex-direction: column;
        }

        #gameCanvas {
            flex-grow: 1;
            background-color: #000000;
            cursor: pointer;
            touch-action: manipulation;
        }

        .ui-panel {
            background-color: #000000;
            padding: 0.5rem 1rem;
            color: #ffffff;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 0.9rem;
            border-bottom: 2px solid #ec4899; /* Pink border */
        }
        
        .ui-panel div:nth-child(2) {
            text-align: center;
        }

        .modal {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.95);
            backdrop-filter: blur(5px);
            z-index: 10;
            display: none;
            justify-content: center;
            align-items: center;
            max-width: 600px;
        }

        .modal-content {
            background-color: #1e1e3f;
            color: #ffffff;
            padding: 1.5rem;
            border-radius: 16px;
            text-align: center;
            max-width: 90%;
            border: 3px solid #f43f5e; /* Red/Pink border */
            box-shadow: 0 0 40px rgba(244, 63, 94, 0.8);
            animation: modalPop 0.3s ease-out;
            max-height: 90vh;
            overflow-y: auto;
        }

        .btn-action {
            background-color: #ec4899;
            color: #ffffff;
            font-weight: 900;
            padding: 0.75rem 1.5rem;
            border-radius: 8px;
            transition: all 0.2s ease;
            box-shadow: 0 4px 0 #db2777;
            margin: 0.3rem;
            position: relative;
            top: 0;
            display: inline-block;
            letter-spacing: 1px;
            font-size: 0.8rem;
        }

        .btn-action:active {
            box-shadow: 0 1px 0 #db2777;
            top: 3px;
        }

        .cube-select-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 0.5rem;
            margin-bottom: 1.5rem;
            max-width: 400px;
            margin-left: auto;
            margin-right: auto;
        }

        .cube-option {
            width: 40px;
            height: 40px;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.1s;
            border: 3px solid transparent;
            box-sizing: border-box;
            background-color: #f43f5e;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
        }

        .cube-option.selected {
            border-color: #facc15; /* Neon Yellow selection highlight */
            box-shadow: 0 0 10px #facc15;
            transform: scale(1.1);
        }
        
        #levelSelection {
            border-top: 1px solid #f43f5e;
            padding-top: 1rem;
        }

        @keyframes modalPop {
            from { opacity: 0; transform: scale(0.8); }
            to { opacity: 1; transform: scale(1); }
        }
    </style>
</head>
<body>

<div class="game-container">
    <!-- UI Panel -->
    <div class="ui-panel">
        <div>LEVEL: <span id="levelNameDisplay" class="text-pink-400 font-bold"></span></div>
        <h1 class="text-xl font-extrabold text-red-400 tracking-wider">SWEET CHILD DASH</h1>
        <div>PROGRESS: <span id="progressDisplay" class="text-yellow-400 font-bold">0%</span></div>
    </div>

    <!-- Game Canvas -->
    <canvas id="gameCanvas"></canvas>

    <!-- Start/Game Over Modal -->
    <div id="gameModal" class="modal">
        <div class="modal-content">
            <h2 id="modalTitle" class="text-3xl font-bold mb-4 text-red-400">EASY FLOW MODE</h2>
            <p id="modalMessage" class="text-gray-300 mb-6">
                Jump to the slower, melodic rhythm of *Sweet Child o' Mine* (125 BPM).
            </p>

            <h3 class="text-xl text-pink-400 mb-2">Icon Customization (20 Faces)</h3>
            <div id="cubeSelection" class="cube-select-container">
                <!-- Cube options generated by JS -->
            </div>

            <h3 class="text-xl text-pink-400 mb-2">Level Selection (10 Easy Levels)</h3>
            <div id="levelSelection" class="flex flex-wrap justify-center mb-6">
                <!-- Level buttons generated by JS -->
            </div>

            <button id="startButton" class="btn-action bg-green-500 hover:bg-green-600 shadow-green-700/50" disabled>START</button>
        </div>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const progressDisplay = document.getElementById('progressDisplay');
    const levelNameDisplay = document.getElementById('levelNameDisplay');
    const gameModal = document.getElementById('gameModal');
    const modalTitle = document.getElementById('modalTitle');
    const modalMessage = document.getElementById('modalMessage');
    const startButton = document.getElementById('startButton');
    const cubeSelection = document.getElementById('cubeSelection');
    const levelSelection = document.getElementById('levelSelection');

    let gameLoopId;
    let isGameRunning = false;
    let selectedLevelName = null;
    let selectedLevelData = [];
    let currentLevelIndex = 0;
    let progress = 0;

    // Game Constants
    const BPM = 125; // Adjusted to a slower, more melodic tempo
    const PLAYER_SIZE = 30;
    const GRAVITY = 1.2;
    const JUMP_STRENGTH = -20;
    const BASE_GROUND_OFFSET = 5;

    // Dynamic Variables
    const GAME_SPEED = BPM / 24; // Speed synchronized to BPM
    let groundY = 0;
    let playerGroundHeight = 0;

    // Player Object
    const player = {
        x: 50,
        y: 0,
        width: PLAYER_SIZE,
        height: PLAYER_SIZE,
        velocityY: 0,
        isJumping: true,
        color: '#4ade80', // Default Neon Green
        patternIndex: 0
    };

    let obstacles = [];

    // --- Audio/Rhythm Engine (Tone.js) ---
    // Simulating the melodic, arpeggiated feel of Sweet Child o' Mine's main riff
    const masterFx = new Tone.Reverb(2).toDestination();
    const kick = new Tone.MembraneSynth({ pitchDecay: 0.05, octave: 1, envelope: { attack: 0.001, decay: 0.4, sustain: 0.01, release: 1.4, } }).connect(masterFx);
    const lead = new Tone.Synth({
        oscillator: { type: "sawtooth" },
        envelope: { attack: 0.05, decay: 0.2, sustain: 0.1, release: 0.4 }
    }).connect(masterFx);
    let musicLoop;

    function startMusic() {
        Tone.context.resume();
        if (Tone.Transport.state !== 'started') {
            Tone.Transport.start();
        }
        Tone.Transport.bpm.value = BPM;

        // Simplified melodic pattern (Guns N' Roses inspired)
        const riffNotes = ["D4", "G4", "A4", "D4", "G4", "A4", "D4", "C4"]; 
        let index = 0;
        musicLoop = new Tone.Loop(time => {
            // Kick on the main beat (every 4th 16th note, or every beat)
            if (index % 4 === 0) {
                kick.triggerAttackRelease("C1", "8n", time);
            }
            
            // Arpeggiated Lead line (8th notes rhythm)
            if (index % 2 === 0) {
                const note = riffNotes[(index / 2) % riffNotes.length];
                lead.triggerAttackRelease(note, "8n", time, 0.5);
            }

            index++;
        }, "16n").start(0); 
    }

    function stopMusic() {
        if (musicLoop) {
            musicLoop.stop();
        }
    }

    function jumpSound() {
        // Higher pitched sound for jump
        lead.triggerAttackRelease("G5", "32n", Tone.now(), 0.8); 
    }

    // --- Cube Styles (Same as before) ---
    const CUBE_STYLES = [
        { color: '#4ade80', name: 'Neon Green', pattern: 'outline' }, { color: '#facc15', name: 'Voltage Yellow', pattern: 'target' },
        { color: '#f43f5e', name: 'Crimson Surge', pattern: 'crosshair' }, { color: '#3b82f6', name: 'Electric Blue', pattern: 'x-mark' },
        { color: '#8b5cf6', name: 'Synth Purple', pattern: 'grid-4' }, { color: '#10b981', name: 'Aqua Glow', pattern: 'stripes-v' },
        { color: '#f97316', name: 'Lava Orange', pattern: 'stripes-h' }, { color: '#ec4899', name: 'Pink Glitch', pattern: 'chevron' },
        { color: '#06b6d4', name: 'Cyan Streak', pattern: 'diamond' }, { color: '#fbbf24', name: 'Gold Prime', pattern: 'dot-matrix' },
        { color: '#57534e', name: 'Shadow Grey', pattern: 'split' }, { color: '#d1d5db', name: 'Silver Light', pattern: 'triangle-top' },
        { color: '#a78bfa', name: 'Violet Pulse', pattern: 'star-4' }, { color: '#fbcfe8', name: 'Rose Quartz', pattern: 'eye' },
        { color: '#4c0519', name: 'Dark Blood', pattern: 'maze' }, { color: '#2dd4bf', name: 'Teal Turbo', pattern: 'portal' },
        { color: '#eab308', name: 'Bronze Flash', pattern: 'bolt' }, { color: '#94a3b8', pattern: 'Steel Mist', name: 'half-moon' },
        { color: '#1e293b', name: 'Deep Black', pattern: 'checker-1/4' }, { color: '#4338ca', name: 'Indigo Warp', pattern: 'bars-3' },
    ];


    // --- Level Data Structure (EASIER LEVELS) ---
    // [type, height_level, beat_gap (16th notes), width_multiplier]
    // type: 1=Spike, 2=Block/Platform, 3=Triple Spike (3 adjacent spikes)
    // height_level: 0=base ground (GND), 1=low platform (50px), 2=high platform (100px)
    function beatsToFrames(beats) {
        // Calculates frame wait time based on BPM and 16th notes (4 per beat)
        return Math.round((beats * 60) * (60 / BPM / 4));
    }

    // Rookie levels are very simple and spaced out (16-24 beats)
    const LEVEL_ROOKIE = {
        '01: Walk in the Park (Rookie)': [[1, 0, 24, 1], [1, 0, 20, 1], [2, 1, 24, 2], [1, 1, 16, 1], [1, 0, 20, 1]].flatMap(o => [o, [1, 0, 16, 1]]).slice(0, 20),
        '02: Gentle Steps (Rookie)': [[2, 1, 20, 1], [1, 0, 18, 1], [2, 0, 22, 1], [1, 0, 18, 1], [2, 1, 20, 1]].flatMap(o => [o, [1, 0, 16, 1]]).slice(0, 20),
        '03: Single Skip (Rookie)': [[1, 0, 18, 1], [1, 0, 14, 1], [1, 0, 18, 1], [1, 0, 14, 1], [1, 0, 18, 1]].flatMap(o => [o, [1, 0, 14, 1]]).slice(0, 20),
        '04: High Hopes (Rookie)': [[2, 2, 20, 1], [1, 0, 18, 1], [2, 0, 16, 1], [1, 0, 14, 1], [2, 2, 20, 1]].flatMap(o => [o, [1, 0, 16, 1]]).slice(0, 20),
        '05: Block Jump (Rookie)': [[1, 0, 16, 1], [2, 1, 12, 1], [1, 1, 12, 1], [1, 0, 16, 1], [2, 2, 12, 1]].flatMap(o => [o, [1, 0, 12, 1]]).slice(0, 20),
    };

    // Expert levels are moderate (12-16 beats), introducing some double jumps but no triples
    const LEVEL_EXPERT = {
        '06: Melodic Momentum (Expert)': [[1, 0, 16, 1], [1, 0, 12, 1], [1, 0, 16, 1], [2, 0, 12, 1], [1, 0, 14, 1]].flatMap(o => [o, [1, 0, 10, 1]]).slice(0, 18),
        '07: Light Sync (Expert)': [[2, 2, 14, 1], [1, 0, 10, 1], [2, 1, 12, 1], [1, 1, 8, 1], [2, 2, 14, 1]].flatMap(o => [o, [1, 0, 10, 1]]).slice(0, 18),
        '08: Double Spacing (Expert)': [[1, 0, 12, 1], [1, 0, 8, 1], [1, 0, 12, 1], [1, 0, 8, 1], [1, 0, 12, 1]].flatMap(o => [o, [1, 0, 8, 1]]).slice(0, 18),
        '09: Mid-Tempo Flow (Expert)': [[2, 2, 10, 1], [1, 0, 8, 1], [2, 1, 10, 1], [1, 0, 8, 1], [2, 2, 10, 1]].flatMap(o => [o, [1, 0, 8, 1]]).slice(0, 18),
        '10: Final Harmony (Expert)': [[1, 0, 12, 1], [2, 2, 8, 1], [1, 0, 8, 1], [1, 0, 12, 1], [2, 1, 8, 1], [1, 0, 8, 1]].flatMap(o => [o, [1, 0, 8, 1]]).slice(0, 18)
    };
    
    const LEVELS_MAP = {...LEVEL_ROOKIE, ...LEVEL_EXPERT};

    // --- Utility Functions ---
    function resizeCanvas() {
        const container = canvas.parentElement;
        canvas.width = container.clientWidth;
        canvas.height = container.clientHeight - document.querySelector('.ui-panel').offsetHeight;

        groundY = canvas.height - PLAYER_SIZE - BASE_GROUND_OFFSET;
        playerGroundHeight = groundY;

        if (!isGameRunning) {
            player.y = playerGroundHeight;
            drawStartScreen();
        }
    }

    function resetGame() {
        stopMusic();
        obstacles = [];
        currentLevelIndex = 0;
        progress = 0;

        // Reset player state
        playerGroundHeight = groundY;
        player.y = playerGroundHeight;
        player.velocityY = 0;
        player.isJumping = false;

        levelNameDisplay.textContent = selectedLevelName || "N/A";
        progressDisplay.textContent = "0%";
        
        // Start music only if a level is selected
        if (selectedLevelName) {
            startMusic();
        }
    }

    // --- Drawing Functions ---

    function drawBackground() {
        ctx.fillStyle = '#110022';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Neon Grid Lines (mimicking GD background)
        ctx.strokeStyle = 'rgba(244, 63, 94, 0.1)'; /* Pink lines */
        ctx.lineWidth = 1;
        for (let i = 0; i < canvas.height; i += 50) {
            ctx.beginPath();
            ctx.moveTo(0, i);
            ctx.lineTo(canvas.width, i);
            ctx.stroke();
        }
    }

    function drawGround() {
        // Draw the main ground
        ctx.fillStyle = '#1c1c3a';
        ctx.fillRect(0, groundY + PLAYER_SIZE - BASE_GROUND_OFFSET, canvas.width, BASE_GROUND_OFFSET * 2);

        // Draw the neon ground line
        ctx.strokeStyle = '#ec4899'; /* Pink neon */
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(0, groundY + PLAYER_SIZE);
        ctx.lineTo(canvas.width, groundY + PLAYER_SIZE);
        ctx.stroke();
    }
    
    // Function to draw player icon pattern (kept simple for readability)
    function drawPattern(pCtx, pattern, s, x, y) {
        pCtx.strokeStyle = '#1e1e3f'; 
        pCtx.lineWidth = 2;
        const c = s / 2; 
        const offset = 4; 
        pCtx.beginPath();
        
        // Simplified patterns for a single draw function
        if (pattern === 'outline') { pCtx.rect(x + offset, y + offset, s - 2 * offset, s - 2 * offset); } 
        else if (pattern === 'target') { pCtx.arc(x + c, y + c, c - offset, 0, Math.PI * 2); } 
        else if (pattern === 'crosshair') { pCtx.moveTo(x + c, y + offset); pCtx.lineTo(x + c, y + s - offset); pCtx.moveTo(x + offset, y + c); pCtx.lineTo(x + s - offset, y + c); } 
        else if (pattern === 'x-mark') { pCtx.moveTo(x + offset, y + offset); pCtx.lineTo(x + s - offset, y + s - offset); pCtx.moveTo(x + s - offset, y + offset); pCtx.lineTo(x + offset, y + s - offset); } 
        else if (pattern === 'grid-4') { pCtx.rect(x + offset, y + offset, c - offset, c - offset); pCtx.rect(x + c, y + c, c - offset, c - offset); } 
        else if (pattern === 'stripes-v') { pCtx.moveTo(x + 10, y); pCtx.lineTo(x + 10, y + s); pCtx.moveTo(x + 30, y); pCtx.lineTo(x + 30, y + s); } 
        else if (pattern === 'diamond') { pCtx.moveTo(x + c, y + offset); pCtx.lineTo(x + s - offset, y + c); pCtx.lineTo(x + c, y + s - offset); pCtx.lineTo(x + offset, y + c); pCtx.closePath(); } 
        else if (pattern === 'dot-matrix') { 
            pCtx.arc(x + s / 4, y + s / 4, 2, 0, 2 * Math.PI); pCtx.arc(x + 3 * s / 4, y + 3 * s / 4, 2, 0, 2 * Math.PI); 
            pCtx.fill(); pCtx.stroke(); return; 
        }
        else if (pattern === 'bars-3') { pCtx.moveTo(x + s / 4, y + offset); pCtx.lineTo(x + s / 4, y + s - offset); pCtx.moveTo(x + c, y + offset); pCtx.lineTo(x + c, y + s - offset); pCtx.moveTo(x + 3 * s / 4, y + offset); pCtx.lineTo(x + 3 * s / 4, y + s - offset); }
        
        pCtx.stroke();
    }


    function drawPlayer() {
        const P = player;
        const style = CUBE_STYLES[P.patternIndex];

        ctx.fillStyle = P.color;
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 3;

        // Draw the main square
        ctx.beginPath();
        ctx.rect(P.x, P.y, P.width, P.height);
        ctx.fill();
        ctx.stroke();

        drawPattern(ctx, style.pattern, P.width, P.x, P.y);
    }


    function drawObstacle(obstacle) {
        const h = obstacle.height;
        const w = obstacle.width;
        const x = obstacle.x;
        const yBase = groundY + PLAYER_SIZE;

        if (obstacle.type === 1 || obstacle.type === 3) { // Spike or Triple Spike
            ctx.fillStyle = obstacle.color;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 1;

            const y = yBase - h;
            const singleSpikeWidth = 30;
            // Type 3 is fixed 3 spikes, Type 1 uses widthMultiplier (which is 1 here)
            const numSpikes = obstacle.type === 3 ? 3 : (obstacle.width / singleSpikeWidth); 

            for(let i = 0; i < numSpikes; i++) {
                const spikeX = x + (i * singleSpikeWidth); 
                ctx.beginPath();
                ctx.moveTo(spikeX, y + h); // Bottom left
                ctx.lineTo(spikeX + 15, y); // Top point
                ctx.lineTo(spikeX + 30, y + h); // Bottom right
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
            }

        } else if (obstacle.type === 2) { // Platform/Block
            ctx.fillStyle = obstacle.color;
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 3;
            const platformY = yBase - h;

            ctx.fillRect(x, platformY, w, h);
            ctx.strokeRect(x, platformY, w, h);
        }
    }

    function drawStartScreen() {
        drawBackground();
        drawGround();
        drawPlayer();
    }

    // --- Game Logic Functions ---

    function jump() {
        if (!player.isJumping) {
            player.isJumping = true;
            player.velocityY = JUMP_STRENGTH;
            jumpSound();
        }
    }

    function updatePlayer() {
        // Apply gravity
        player.velocityY += GRAVITY;
        player.y += player.velocityY;

        // Check ground collision against the current ground level
        if (player.y > playerGroundHeight) {
            player.y = playerGroundHeight;
            player.velocityY = 0;
            player.isJumping = false;
        }
    }

    function updateObstacles() {
        obstacles.forEach(obstacle => {
            obstacle.x -= GAME_SPEED;
        });

        // Spawn next obstacle if needed
        if (obstacles.length === 0 || obstacles[obstacles.length - 1].x < canvas.width - obstacles[obstacles.length - 1].frameWait) {
            spawnNextObstacle();
        }

        obstacles = obstacles.filter(obstacle => obstacle.x + obstacle.width > 0);

        // Update progress
        progress = Math.floor((currentLevelIndex / selectedLevelData.length) * 100);
        progressDisplay.textContent = ${progress}%;
    }

    function spawnNextObstacle() {
        if (currentLevelIndex >= selectedLevelData.length) {
            return;
        }

        const [type, heightLevel, beatGap, widthMultiplier] = selectedLevelData[currentLevelIndex];
        const framesGap = beatsToFrames(beatGap);
        const obstacleX = canvas.width;

        const PLATFORM_HEIGHTS = [0, 50, 100]; // Base, Low, High
        const BLOCK_HEIGHT = PLATFORM_HEIGHTS[heightLevel];

        if (type === 1 || type === 3) { // Spike or Triple Spike
            const singleSpikeWidth = 30;
            const numSpikes = (type === 3 ? 3 : widthMultiplier); 
            const totalWidth = numSpikes * singleSpikeWidth;

            // Push a single composite obstacle (multiple spikes are drawn in drawObstacle)
            obstacles.push({
                type: type,
                x: obstacleX,
                width: totalWidth,
                height: singleSpikeWidth,
                color: '#f87171', // Spikes are red
                frameWait: framesGap
            });

        } else if (type === 2) { // Platform/Block
            const platformWidth = widthMultiplier * 60;

            obstacles.push({
                type: 2,
                x: obstacleX,
                width: platformWidth,
                height: BLOCK_HEIGHT,
                color: '#3b82f6', // Blocks are blue
                frameWait: framesGap
            });
        }

        currentLevelIndex++;
    }

    function checkCollisionAndPlatforms() {
        let newPlayerGroundHeight = groundY;
        let isGroundedOnPlatform = false;

        for (const obs of obstacles) {
            // Player vs Obstacle Bounding Box Check
            if (player.x + player.width > obs.x + 5 && player.x < obs.x + obs.width - 5) { 
                const obsTopY = groundY + PLAYER_SIZE - obs.height;

                // 1. Platform Landing/Grounding Check (Type 2)
                if (obs.type === 2 && obs.height > 0) {
                    if (player.y + player.height <= obsTopY + player.velocityY && player.y + player.height + player.velocityY > obsTopY) {
                         // Player is landing on the platform
                         newPlayerGroundHeight = obsTopY - PLAYER_SIZE;
                         isGroundedOnPlatform = true;
                    }
                }

                // 2. Collision Check (Spikes and Wall Sides)
                if (player.y + player.height > obsTopY) { // Player bottom is below obstacle top
                    if (obs.type === 1 || obs.type === 3) {
                        return true; // Hit a spike
                    }
                    if (obs.type === 2 && !isGroundedOnPlatform) {
                        // Collision with the side or bottom of a platform
                        if (player.x + player.width > obs.x + 5 && player.x < obs.x + obs.width - 5 && player.y < obsTopY + player.height/2) {
                            return true;
                        }
                    }
                }
            }
        }
        playerGroundHeight = newPlayerGroundHeight;

        // Check for level completion
        if (currentLevelIndex >= selectedLevelData.length && obstacles.length === 0) {
            gameOver(true); 
            return true;
        }

        return false;
    }

    // --- Game Flow Control ---

    function gameLoop(timestamp) {
        if (!isGameRunning) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawBackground();
        drawGround();

        updatePlayer();
        drawPlayer();

        updateObstacles();
        obstacles.forEach(drawObstacle);

        if (checkCollisionAndPlatforms()) {
            // Collision or Win handled in checkCollisionAndPlatforms
            if (isGameRunning) gameOver(false);
            return;
        }

        gameLoopId = requestAnimationFrame(gameLoop);
    }

    function startGame() {
        if (!selectedLevelName) return;
        Tone.context.resume();
        resetGame();
        gameModal.style.display = 'none';
        isGameRunning = true;
        gameLoopId = requestAnimationFrame(gameLoop);
    }

    function gameOver(isWin) {
        isGameRunning = false;
        cancelAnimationFrame(gameLoopId);
        stopMusic();

        modalTitle.textContent = isWin ? "LEVEL CLEARED!" : "GAME OVER";
        modalMessage.innerHTML = `
            ${isWin ? <span class="text-green-400">SUCCESS! You mastered ${selectedLevelName}.</span> : <span class="text-red-400">CRASH!</span>}
            <br>
            Progress: <span class="text-yellow-400 font-extrabold">${progress}%</span>.
            <br><br>
            Ready for another sweet run?
        `;
        startButton.textContent = "RESTART LEVEL";
        gameModal.style.display = 'flex';
    }

    // --- Input Handlers & UI Setup ---

    function handleInputStart(event) {
        event.preventDefault();
        if (isGameRunning) {
            jump();
        } else if (gameModal.style.display === 'flex' && selectedLevelName) {
            // Allows tapping screen to quickly start if level is selected
            startGame();
        }
    }

    function handleLevelSelect(event) {
        const target = event.target.closest('.btn-action');
        if (target && target.dataset.level) {
            document.querySelectorAll('#levelSelection .btn-action').forEach(btn => {
                btn.classList.remove('bg-green-500', 'shadow-green-700/50');
                if (btn.dataset.level.includes('Rookie')) {
                    btn.classList.add('bg-indigo-600', 'shadow-indigo-800');
                } else {
                    btn.classList.add('bg-red-600', 'shadow-red-800');
                }
            });
            target.classList.add('bg-green-500', 'shadow-green-700/50');
            target.classList.remove('bg-indigo-600', 'shadow-indigo-800', 'bg-red-600', 'shadow-red-800');

            selectedLevelName = target.dataset.level;
            selectedLevelData = LEVELS_MAP[selectedLevelName];
            startButton.disabled = false;
            startButton.textContent = START ${selectedLevelName.split(':')[0]};
        }
    }

    function drawPreviewCube(pCtx, index) {
        const style = CUBE_STYLES[index];
        const s = 40;
        const x = 0;
        const y = 0;
        
        pCtx.fillStyle = style.color;
        pCtx.strokeStyle = '#fff';
        pCtx.lineWidth = 2;

        pCtx.beginPath();
        pCtx.rect(x, y, s, s);
        pCtx.fill();
        pCtx.stroke();

        drawPattern(pCtx, style.pattern, s, x, y);
    }


    function handleCubeSelect(event) {
        const target = event.target.closest('.cube-option');
        if (target && target.dataset.index) {
            document.querySelectorAll('.cube-option').forEach(opt => opt.classList.remove('selected'));
            target.classList.add('selected');

            const index = parseInt(target.dataset.index);
            player.color = CUBE_STYLES[index].color;
            player.patternIndex = index;
        }
    }

    function setupUI() {
        // Setup Cube Options
        cubeSelection.innerHTML = CUBE_STYLES.map((style, index) => `
            <div class="cube-option ${index === 0 ? 'selected' : ''}" data-index="${index}" title="${style.name}">
                <canvas width="40" height="40"></canvas>
            </div>
        `).join('');
        
        // Draw the initial patterns on the canvases
        document.querySelectorAll('.cube-option canvas').forEach((canvas, index) => {
            drawPreviewCube(canvas.getContext('2d'), index);
        });

        // Setup Level Buttons
        levelSelection.innerHTML = Object.keys(LEVELS_MAP).map(levelName => `
            <button class="btn-action ${levelName.includes('Rookie') ? 'bg-indigo-600 shadow-indigo-800' : 'bg-red-600 shadow-red-800'}" 
                    data-level="${levelName}">
                ${levelName}
            </button>
        `).join('');

        // Set default level to the first one
        selectedLevelName = Object.keys(LEVELS_MAP)[0];
        selectedLevelData = LEVELS_MAP[selectedLevelName];
        
        // Automatically select the first button visually
        const firstButton = document.querySelector('#levelSelection .btn-action');
        if (firstButton) {
            firstButton.classList.add('bg-green-500', 'shadow-green-700/50');
            firstButton.classList.remove('bg-indigo-600', 'shadow-indigo-800');
        }
        
        startButton.disabled = false;
        startButton.textContent = START ${selectedLevelName.split(':')[0]};
        levelNameDisplay.textContent = selectedLevelName;
    }

    // --- Event Listeners ---
    window.addEventListener('resize', resizeCanvas);
    startButton.addEventListener('click', startGame);
    levelSelection.addEventListener('click', handleLevelSelect);
    cubeSelection.addEventListener('click', handleCubeSelect);

    // Mobile/Desktop Jump Input
    canvas.addEventListener('touchstart', handleInputStart);
    canvas.addEventListener('mousedown', handleInputStart);

    // Initial Setup
    window.onload = () => {
        setupUI();
        resizeCanvas();
        gameModal.style.display = 'flex';
    };

</script>

</body>
</html>

