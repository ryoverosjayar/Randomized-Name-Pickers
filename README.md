
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Picker</title>
    <style>
        body { font-family: 'Comic Sans MS', 'Comic Sans', cursive; display: flex; align-items: center; justify-content: center; background: #1a1a2e; color: white; min-height: 100vh; margin: 0; overflow: hidden; }
        
        #side-panel { position: absolute; left: 0; top: 0; width: 250px; height: 100vh; background: rgba(22, 33, 62, 0.95); padding: 20px; box-sizing: border-box; display: flex; flex-direction: column; gap: 15px; z-index: 20; transition: transform 0.3s ease; }
        #side-panel.hidden { transform: translateX(-100%); }

        #game-area { text-align: center; z-index: 10; position: absolute; cursor: move; display: flex; flex-direction: column; align-items: center; }
        #spinner { width: 450px; height: 450px; border: 15px solid #e94560; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 3rem; font-weight: bold; background: white; color: #333; overflow: hidden; box-shadow: 0 0 50px rgba(233, 69, 96, 0.6); transition: width 0.1s, height 0.1s; animation: pulse 2s infinite; }
        #spinner img { width: 100%; height: 100%; object-fit: cover; }
        
        @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
        
        textarea { width: 100%; height: 100px; border-radius: 8px; border: none; padding: 10px; box-sizing: border-box; font-family: 'Comic Sans MS', cursive; }
        
        .btn-group { display: flex; flex-direction: column; gap: 10px; }
        button { background: #e94560; color: white; border: none; padding: 15px; font-size: 1.1rem; cursor: pointer; border-radius: 8px; font-weight: bold; transition: 0.3s; font-family: 'Comic Sans MS', cursive; }
        button:hover { background: #ff2e63; }
        button.secondary { background: #4ecca3; }
        button.secondary:hover { background: #45b792; }
        
        #floating-spin { display: none; margin-top: 20px; padding: 20px 50px; font-size: 1.5rem; background: #e94560; border: none; color: white; border-radius: 50px; cursor: pointer; font-weight: bold; font-family: 'Comic Sans MS', cursive; }
        
        .upload-bg-btn { background: #555; color: white; border: none; padding: 10px; font-size: 0.9rem; cursor: pointer; border-radius: 8px; text-align: center; }
        
        .credit { position: fixed; bottom: 15px; left: 50%; transform: translateX(-50%); padding: 10px 20px; background: rgba(0, 102, 255, 0.8); border: 2px solid white; color: white; border-radius: 50px; font-size: 1rem; font-weight: bold; text-shadow: 1px 1px 2px #000; z-index: 10; font-family: 'Comic Sans MS', cursive; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        
        canvas { position: fixed; top: 0; left: 0; pointer-events: none; z-index: 5; }

        #bg-video { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; z-index: -1; object-fit: cover; object-position: center; }
        
        #modal-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0,0,0,0.8); z-index: 100; display: none; align-items: center; justify-content: center; }
        #modal-content { background: #1a1a2e; padding: 30px; border-radius: 15px; max-width: 500px; color: white; border: 2px solid #e94560; text-align: left; font-family: 'Comic Sans MS', cursive; }
    </style>
</head>
<body>

<video id="bg-video" autoplay loop muted playsinline src="https://res.cloudinary.com/dukvdvfro/video/upload/v1781413332/download_8_l8646r.mp4"></video>

<div id="side-panel">
    <textarea id="input" placeholder="Names (One per row or separated by commas)"></textarea>
    <input type="file" id="imageUpload" accept="image/*" multiple>
    <div class="btn-group">
        <button onclick="startSpin()">SPIN!</button>
        <button class="secondary" onclick="toggleFullScreen()">FULLSCREEN</button>
        <button class="info" onclick="toggleInstructions()">HOW TO USE</button>
        <label for="bgUpload" class="upload-bg-btn">Upload Background</label>
        <input type="file" id="bgUpload" accept="image/jpeg, image/png, video/mp4" style="display: none;">
    </div>
</div>

<div id="modal-overlay" onclick="toggleInstructions()">
    <div id="modal-content" onclick="event.stopPropagation()">
        <h2>How to use:</h2>
        <ul style="line-height: 1.8;">
            <li><b>Option 1: Names</b> - Type names, one per row.</li>
            <li><b>Option 2: Student Number</b> - Type numbers, one per row.</li>
            <li><b>Option 3: Images</b> - Upload student photos.</li>
            <li><b>Move:</b> Click and drag the circle.</li>
            <li><b>Resize:</b> Scroll mouse wheel over the circle.</li>
        </ul>
        <button style="width: 100%; margin-top: 10px;" onclick="toggleInstructions()">CLOSE</button>
    </div>
</div>

<div id="game-area">
    <div id="spinner">READY</div>
    <button id="floating-spin" onclick="startSpin()">SPIN!</button>
</div>

<div class="credit">Created by : Teacher Jay Ryu in Thailand</div>

<canvas id="confetti"></canvas>

<script>
    let uploadedImages = [];
    let currentPool = [];
    const spinner = document.getElementById('spinner');
    const gameArea = document.getElementById('game-area');
    const panel = document.getElementById('side-panel');
    const floatBtn = document.getElementById('floating-spin');
    const modal = document.getElementById('modal-overlay');

    function toggleInstructions() {
        modal.style.display = modal.style.display === 'flex' ? 'none' : 'flex';
    }

    // Dragging Logic
    let isDragging = false;
    gameArea.addEventListener('mousedown', (e) => { isDragging = true; });
    document.addEventListener('mousemove', (e) => {
        if(isDragging) {
            gameArea.style.left = (e.clientX - gameArea.offsetWidth/2) + 'px';
            gameArea.style.top = (e.clientY - gameArea.offsetHeight/2) + 'px';
        }
    });
    document.addEventListener('mouseup', () => { isDragging = false; });

    // Resize Logic
    spinner.addEventListener('wheel', (e) => {
        e.preventDefault();
        let size = spinner.offsetWidth;
        size += e.deltaY * -0.5;
        size = Math.min(Math.max(size, 100), 800);
        spinner.style.width = size + 'px';
        spinner.style.height = size + 'px';
    });

    // Background Management
    const bgUpload = document.getElementById('bgUpload');
    const bgVideo = document.getElementById('bg-video');
    bgUpload.addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (!file) return;
        const fileURL = URL.createObjectURL(file);
        bgVideo.src = fileURL;
        bgVideo.play().catch(console.error);
    });

    document.getElementById('imageUpload').addEventListener('change', (e) => {
        uploadedImages = Array.from(e.target.files).map(file => URL.createObjectURL(file));
        currentPool = []; // Reset pool when files update
    });

    function toggleFullScreen() {
        if (!document.fullscreenElement) document.documentElement.requestFullscreen();
        else document.exitFullscreen();
    }

    document.addEventListener('fullscreenchange', () => {
        floatBtn.style.display = document.fullscreenElement ? 'block' : 'none';
    });

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playTick() {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain); gain.connect(audioCtx.destination);
        osc.frequency.value = 600; gain.gain.value = 0.05;
        osc.start(); osc.stop(audioCtx.currentTime + 0.03);
    }

    function playWinSound() {
        const audio = new Audio('https://res.cloudinary.com/dukvdvfro/video/upload/v1781414052/Record_online-voice-recorder.com_1_elrgj5.mp3');
        audio.volume = 1.0;
        audio.play().catch(e => console.error("Audio play failed:", e));
    }

    function startSpin() {
        // Refill pool if empty
        if (currentPool.length === 0) {
            const inputData = document.getElementById('input').value.split(/[\n,]+/).map(s => s.trim()).filter(s => s !== "");
            currentPool = [...inputData, ...uploadedImages];
        }

        if (currentPool.length === 0) return alert("Add names, numbers, or images first!");
        
        panel.classList.add('hidden');
        floatBtn.style.display = 'none';
        
        let count = 0;
        const interval = setInterval(() => {
            const item = currentPool[Math.floor(Math.random() * currentPool.length)];
            if (item.startsWith('blob:')) spinner.innerHTML = `<img src="${item}">`;
            else spinner.innerText = item;
            playTick();
            count++;
            if (count > 25) {
                clearInterval(interval);
                finish(item);
            }
        }, 80);
    }

    function finish(winner) {
        // Remove the winner from the pool
        const index = currentPool.indexOf(winner);
        if (index > -1) currentPool.splice(index, 1);
        
        playWinSound();
        startConfetti();
        setTimeout(() => {
            panel.classList.remove('hidden');
            if(document.fullscreenElement) floatBtn.style.display = 'block';
        }, 2000);
    }

    function startConfetti() {
        const canvas = document.getElementById('confetti');
        canvas.width = window.innerWidth; canvas.height = window.innerHeight;
        const ctx = canvas.getContext('2d');
        let particles = Array.from({length: 400}, () => ({
            x: canvas.width / 2, y: canvas.height / 2,
            vx: (Math.random() - 0.5) * 40, vy: (Math.random() - 0.5) * 40,
            color: `hsl(${Math.random()*360}, 100%, 50%)`,
            size: Math.random() * 8 + 2
        }));
        function animate() {
            ctx.clearRect(0,0, canvas.width, canvas.height);
            particles.forEach(p => { 
                p.x += p.vx; p.y += p.vy; 
                p.vy += 0.5; 
                ctx.fillStyle = p.color; 
                ctx.fillRect(p.x, p.y, p.size, p.size); 
            });
            requestAnimationFrame(animate);
        }
        animate();
    }
</script>
</body>
</html>
