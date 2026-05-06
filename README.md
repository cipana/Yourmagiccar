# Yourmagiccar
YourMagicCar este un simulator auto open-source pe GitHub ce îmbină condusul realist cu elemente fantastice. Modifică parametrii mașinii în timp real, de la fizică la estetică magică, și explorează trasee diverse. Proiectul este modular, ideal pentru developeri și jucători care vor să transforme codul în viteză pură.
<!DOCTYPE html>
<html lang="ro">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>YourMagicCar - Racing Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #1a202c;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        canvas {
            display: block;
            margin: 0 auto;
            background-color: #333;
            box-shadow: 0 0 50px rgba(0,0,0,0.5);
        }
        .ui-overlay {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
            pointer-events: none;
        }
        .menu-screen {
            position: absolute;
            inset: 0;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10;
            color: white;
        }
        .card-select {
            transition: all 0.3s ease;
            cursor: pointer;
            border: 2px solid transparent;
        }
        .card-select:hover {
            transform: scale(1.05);
            border-color: #4a90e2;
        }
        .card-select.active {
            border-color: #f6ad55;
            background: rgba(246, 173, 85, 0.2);
        }
    </style>
</head>
<body>

    <div id="start-menu" class="menu-screen">
        <h1 class="text-5xl font-bold mb-8 text-orange-500">YourMagicCar</h1>
        
        <div class="mb-6 w-full max-w-4xl px-4">
            <h2 class="text-2xl mb-4 text-center">Alege Mașina Magică</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div onclick="selectCar(0)" id="car-0" class="card-select p-4 bg-gray-800 rounded-lg active">
                    <div class="text-xl font-bold mb-2">⚡ Viteza</div>
                    <p class="text-sm text-gray-400">Accelerație mare, dar manevrabilitate redusă.</p>
                </div>
                <div onclick="selectCar(1)" id="car-1" class="card-select p-4 bg-gray-800 rounded-lg">
                    <div class="text-xl font-bold mb-2">🚜 Off-Road</div>
                    <p class="text-sm text-gray-400">Control excelent, viteză maximă moderată.</p>
                </div>
                <div onclick="selectCar(2)" id="car-2" class="card-select p-4 bg-gray-800 rounded-lg">
                    <div class="text-xl font-bold mb-2">🛡️ Echilibrat</div>
                    <p class="text-sm text-gray-400">Perfect pentru orice tip de circuit.</p>
                </div>
            </div>
        </div>

        <div class="mb-8 w-full max-w-4xl px-4">
            <h2 class="text-2xl mb-4 text-center">Alege Mapa</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div onclick="selectMap(0)" id="map-0" class="card-select p-4 bg-gray-800 rounded-lg active">
                    <div class="text-xl font-bold mb-2">🏙️ Circuit Urban</div>
                    <p class="text-sm text-gray-400">Asfalt neted, curbe strânse.</p>
                </div>
                <div onclick="selectMap(1)" id="map-1" class="card-select p-4 bg-gray-800 rounded-lg">
                    <div class="text-xl font-bold mb-2">🏜️ Valea Moartă</div>
                    <p class="text-sm text-gray-400">Nisip și praf, vizibilitate redusă.</p>
                </div>
                <div onclick="selectMap(2)" id="map-2" class="card-select p-4 bg-gray-800 rounded-lg">
                    <div class="text-xl font-bold mb-2">🌌 Noapte în Oraș</div>
                    <p class="text-sm text-gray-400">Lumini neon și viteză extremă.</p>
                </div>
            </div>
        </div>

        <button onclick="startGame()" class="bg-orange-500 hover:bg-orange-600 text-white px-12 py-4 rounded-full text-2xl font-bold transition-transform active:scale-95">
            START CURSĂ
        </button>
    </div>

    <div class="ui-overlay">
        <div class="text-3xl font-mono" id="timer">Timp: 00:00</div>
        <div class="text-xl font-mono mt-2" id="speed">Viteză: 0 km/h</div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const timerElement = document.getElementById('timer');
        const speedElement = document.getElementById('speed');

        // Configurații Joc
        let gameState = 'menu';
        let startTime = 0;
        let selectedCarIdx = 0;
        let selectedMapIdx = 0;

        const cars = [
            { name: "Viteza", color: "#e53e3e", accel: 0.15, friction: 0.98, turnSpeed: 0.04, maxSpeed: 10 },
            { name: "Off-Road", color: "#38a169", accel: 0.1, friction: 0.96, turnSpeed: 0.07, maxSpeed: 7 },
            { name: "Echilibrat", color: "#3182ce", accel: 0.12, friction: 0.97, turnSpeed: 0.05, maxSpeed: 8.5 }
        ];

        const maps = [
            { name: "Urban", bg: "#4a5568", track: "#2d3748", decoration: "#cbd5e0" },
            { name: "Desert", bg: "#ed8936", track: "#dd6b20", decoration: "#f6ad55" },
            { name: "Night", bg: "#1a202c", track: "#2d3748", decoration: "#4a5568" }
        ];

        // Obiectul Jucător
        const player = {
            x: 0,
            y: 0,
            angle: 0,
            speed: 0,
            width: 40,
            height: 20,
            controls: {
                up: false,
                down: false,
                left: false,
                right: false
            }
        };

        // Inițializare Canvas
        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            player.x = canvas.width / 2;
            player.y = canvas.height / 2;
        }

        window.addEventListener('resize', resize);
        resize();

        // Control input
        window.addEventListener('keydown', e => handleKey(e, true));
        window.addEventListener('keyup', e => handleKey(e, false));

        function handleKey(e, isDown) {
            if (e.key === 'ArrowUp' || e.key === 'w') player.controls.up = isDown;
            if (e.key === 'ArrowDown' || e.key === 's') player.controls.down = isDown;
            if (e.key === 'ArrowLeft' || e.key === 'a') player.controls.left = isDown;
            if (e.key === 'ArrowRight' || e.key === 'd') player.controls.right = isDown;
        }

        function selectCar(idx) {
            document.querySelectorAll('#car-0, #car-1, #car-2').forEach(el => el.classList.remove('active'));
            document.getElementById(`car-${idx}`).classList.add('active');
            selectedCarIdx = idx;
        }

        function selectMap(idx) {
            document.querySelectorAll('#map-0, #map-1, #map-2').forEach(el => el.classList.remove('active'));
            document.getElementById(`map-${idx}`).classList.add('active');
            selectedMapIdx = idx;
        }

        function startGame() {
            document.getElementById('start-menu').style.display = 'none';
            gameState = 'playing';
            startTime = Date.now();
            player.speed = 0;
            player.x = canvas.width / 2;
            player.y = canvas.height / 2;
            requestAnimationFrame(gameLoop);
        }

        function update() {
            if (gameState !== 'playing') return;

            const carConfig = cars[selectedCarIdx];

            // Accelerație
            if (player.controls.up) {
                player.speed += carConfig.accel;
            } else if (player.controls.down) {
                player.speed -= carConfig.accel;
            }

            // Fricțiune naturală
            player.speed *= carConfig.friction;

            // Limită viteză
            if (player.speed > carConfig.maxSpeed) player.speed = carConfig.maxSpeed;
            if (player.speed < -carConfig.maxSpeed / 2) player.speed = -carConfig.maxSpeed / 2;

            // Direcție (doar dacă se mișcă)
            if (Math.abs(player.speed) > 0.1) {
                const direction = player.speed > 0 ? 1 : -1;
                if (player.controls.left) player.angle -= carConfig.turnSpeed * direction;
                if (player.controls.right) player.angle += carConfig.turnSpeed * direction;
            }

            // Poziție
            player.x += Math.cos(player.angle) * player.speed;
            player.y += Math.sin(player.angle) * player.speed;

            // Coliziune cu marginile (simplificată)
            if (player.x < 0) player.x = canvas.width;
            if (player.x > canvas.width) player.x = 0;
            if (player.y < 0) player.y = canvas.height;
            if (player.y > canvas.height) player.y = 0;

            // Update UI
            const elapsed = ((Date.now() - startTime) / 1000).toFixed(2);
            timerElement.innerText = `Timp: ${elapsed}s`;
            speedElement.innerText = `Viteză: ${Math.abs(Math.round(player.speed * 20))} km/h`;
        }

        function draw() {
            const mapConfig = maps[selectedMapIdx];
            
            // Fundal Mapă
            ctx.fillStyle = mapConfig.bg;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Desenare Drum (Decorativ)
            ctx.strokeStyle = mapConfig.track;
            ctx.lineWidth = 100;
            ctx.beginPath();
            ctx.arc(canvas.width/2, canvas.height/2, 300, 0, Math.PI * 2);
            ctx.stroke();

            // Marcaje drum
            ctx.setLineDash([20, 20]);
            ctx.strokeStyle = "white";
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.setLineDash([]);

            // Desenare Mașină
            ctx.save();
            ctx.translate(player.x, player.y);
            ctx.rotate(player.angle);

            // Corpul mașinii
            ctx.fillStyle = cars[selectedCarIdx].color;
            ctx.fillRect(-player.width / 2, -player.height / 2, player.width, player.height);
            
            // Geam
            ctx.fillStyle = "rgba(255,255,255,0.5)";
            ctx.fillRect(5, -player.height / 2 + 2, 10, player.height - 4);

            // Stopuri/Faruri
            ctx.fillStyle = "yellow";
            ctx.fillRect(player.width/2 - 5, -player.height/2, 5, 5);
            ctx.fillRect(player.width/2 - 5, player.height/2 - 5, 5, 5);

            ctx.restore();

            // Particule praf (dacă are viteză)
            if (Math.abs(player.speed) > 2) {
                ctx.fillStyle = mapConfig.decoration;
                ctx.globalAlpha = 0.3;
                ctx.beginPath();
                ctx.arc(player.x - Math.cos(player.angle) * 30, player.y - Math.sin(player.angle) * 30, Math.random() * 10, 0, Math.PI*2);
                ctx.fill();
                ctx.globalAlpha = 1.0;
            }
        }

        function gameLoop() {
            update();
            draw();
            if (gameState === 'playing') {
                requestAnimationFrame(gameLoop);
            }
        }

        // Inițializare menu decor
        function drawMenuBackground() {
            if (gameState !== 'menu') return;
            // Un mic efect de animație pe fundalul meniului ar putea fi adăugat aici
        }
    </script>
</body>
</html>
        
