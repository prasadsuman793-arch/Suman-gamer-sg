# Suman-gamer-sg
Open world one CHARACTER jiska pura lon family and house is mahal or promt me dek lo
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SUMAN GAMER - 3D Open World</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #111;
            user-select: none;
        }
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            box-sizing: border-box;
        }
        .hud-element {
            pointer-events: auto;
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
        }
        #main-menu {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #1a1a2e, #16213e, #0f3460);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            color: white;
        }
        h1 {
            font-size: 4rem;
            margin-bottom: 10px;
            background: linear-gradient(45deg, #ff416c, #ff4b2b);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
            letter-spacing: 3px;
        }
        .menu-btn {
            background: #ff4b2b;
            border: none;
            color: white;
            padding: 15px 40px;
            font-size: 1.2rem;
            margin: 10px;
            cursor: pointer;
            border-radius: 30px;
            font-weight: bold;
            transition: 0.3s;
            box-shadow: 0 5px 15px rgba(255, 75, 43, 0.4);
        }
        .menu-btn:hover {
            transform: scale(1.05);
            background: #ff416c;
        }
        #game-hud {
            display: none;
            justify-content: space-between;
            align-items: flex-start;
        }
        .stats-box {
            background: rgba(0, 0, 0, 0.5);
            padding: 15px;
            border-radius: 10px;
            border-left: 5px solid #ff4b2b;
            font-size: 1.1rem;
        }
        #controls-hint {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.7);
            padding: 10px 20px;
            border-radius: 8px;
            color: #ddd;
            font-size: 0.9rem;
            text-align: center;
        }
    </style>
    <!-- Three.js लाइब्रेरी लोड करें -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <!-- मेनू स्क्रीन -->
    <div id="main-menu">
        <h1>SUMAN GAMER</h1>
        <p>Indian Open World 3D Action Game</p>
        <button class="menu-btn" onclick="startGame()">PLAY GAME</button>
        <button class="menu-btn" onclick="alert('Settings: Low/Medium/High graphics presets available in full version.')">SETTINGS</button>
        <button class="menu-btn" onclick="alert('Garage: Unlock sports cars and bikes with in-game money!')">GARAGE</button>
    </div>

    <!-- गेम यूआई (HUD) -->
    <div id="ui-layer">
        <div id="game-hud" class="hud-element">
            <div class="stats-box">
                <div>🎮 <b>SUMAN GAMER</b></div>
                <div>💰 Money: $<span id="money">0</span></div>
                <div>⭐ Missions Completed: <span id="missions">0</span></div>
            </div>
            <div class="stats-box" style="border-left-color: #4facfe;">
                <div>🚗 Status: <span id="vehicle-status">Walking (On Foot)</span></div>
                <div id="mission-objective" style="font-size: 0.9rem; color: #ffeb3b; margin-top: 5px;">Objective: Explore the city & collect cash!</div>
            </div>
        </div>
        <div id="controls-hint" class="hud-element">
            Controls: <b>W, A, S, D</b> या Arrow Keys से चलें | <b>F</b> से गाड़ी में बैठें/बाहर निकलें | <b>Space</b> से कूदें
        </div>
    </div>

    <script>
        let scene, camera, renderer, player, car, cityBuildings = [], coins = [];
        let score = 0, missionsCompleted = 0;
        let isDriving = false;
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false;
        let playerVelocity = new THREE.Vector3();
        let gameActive = false;

        function startGame() {
            document.getElementById('main-menu').style.display = 'none';
            document.getElementById('game-hud').style.display = 'flex';
            gameActive = true;
            initThreeJS();
            animate();
        }

        function initThreeJS() {
            // सीन, कैमरा और रेंडरर सेटअप
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb); // खुला आसमान
            scene.fog = new THREE.FogExp2(0x87ceeb, 0.015);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            document.body.appendChild(renderer.domElement);

            // रोशनी (Lighting)
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);

            const sunLight = new THREE.DirectionalLight(0xffffff, 0.8);
            sunLight.position.set(50, 100, 50);
            sunLight.castShadow = true;
            scene.add(sunLight);

            // जमीन (Ground / Road)
            const groundGeo = new THREE.PlaneGeometry(500, 500);
            const groundMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // शहर की इमारतें (Indian City Buildings)
            const buildingMat = new THREE.MeshStandardMaterial({ color: 0x90a4ae });
            for (let i = -200; i <= 200; i += 40) {
                for (let j = -200; j <= 200; j += 40) {
                    if (Math.abs(i) < 20 || Math.abs(j) < 20) continue; // सड़क के लिए जगह छोड़ें
                    const height = Math.random() * 40 + 15;
                    const bGeo = new THREE.BoxGeometry(20, height, 20);
                    const building = new THREE.Mesh(bGeo, buildingMat);
                    building.position.set(i, height / 2, j);
                    building.castShadow = true;
                    building.receiveShadow = true;
                    scene.add(building);
                    cityBuildings.push(building);
                }
            }

            // प्लेयर करैक्टर (Suman Gamer Avatar - 3D Box/Humanoid representation)
            const playerGeo = new THREE.BoxGeometry(1, 2, 1);
            const playerMat = new THREE.MeshStandardMaterial({ color: 0xff4b2b });
            player = new THREE.Mesh(playerGeo, playerMat);
            player.position.set(0, 1, 10);
            player.castShadow = true;
            scene.add(player);

            // स्पोर्ट्स कार (Vehicle)
            const carGeo = new THREE.BoxGeometry(2, 1, 4);
            const carMat = new THREE.MeshStandardMaterial({ color: 0xffeb3b });
            car = new THREE.Mesh(carGeo, carMat);
            car.position.set(10, 0.5, 10);
            car.castShadow = true;
            scene.add(car);

            // सिक्के / मिशन ऑब्जेक्टिव्स (Collectibles)
            const coinGeo = new THREE.CylinderGeometry(1, 1, 0.2, 16);
            const coinMat = new THREE.MeshStandardMaterial({ color: 0xffd700 });
            for (let k = 0; k < 15; k++) {
                const coin = new THREE.Mesh(coinGeo, coinMat);
                coin.rotation.x = Math.PI / 2;
                coin.position.set((Math.random() - 0.5) * 300, 0.5, (Math.random() - 0.5) * 300);
                scene.add(coin);
                coins.push(coin);
            }

            // कीबोर्ड इवेंट्स सुनैं
            window.addEventListener('keydown', onKeyDown);
            window.addEventListener('keyup', onKeyUp);
            window.addEventListener('resize', onWindowResize);
        }

        function onKeyDown(event) {
            if (!gameActive) return;
            switch (event.code) {
                case 'KeyW': case 'ArrowUp': moveForward = true; break;
                case 'KeyS': case 'ArrowDown': moveBackward = true; break;
                case 'KeyA': case 'ArrowLeft': moveLeft = true; break;
                case 'KeyD': case 'ArrowRight': moveRight = true; break;
                case 'KeyF': toggleVehicle(); break;
            }
        }

        function onKeyUp(event) {
            switch (event.code) {
                case 'KeyW': case 'ArrowUp': moveForward = false; break;
                case 'KeyS': case 'ArrowDown': moveBackward = false; break;
                case 'KeyA': case 'ArrowLeft': moveLeft = false; break;
                case 'KeyD': case 'ArrowRight': moveRight = false; break;
            }
        }

        function toggleVehicle() {
            const distance = player.position.distanceTo(car.position);
            if (distance < 5) {
                isDriving = !isDriving;
                document.getElementById('vehicle-status').innerText = isDriving ? "Driving Sports Car 🏎️" : "Walking (On Foot)";
                player.visible = !isDriving;
            }
        }

        function animate() {
            if (!gameActive) return;
            requestAnimationFrame(animate);

            // मूवमेंट लॉजिक
            let speed = 0.2;
            if (isDriving) speed = 0.6;

            if (moveForward) {
                if (isDriving) car.translateZ(-speed);
                else player.translateZ(-speed);
            }
            if (moveBackward) {
                if (isDriving) car.translateZ(speed);
                else player.translateZ(speed);
            }
            if (moveLeft) {
                if (isDriving) car.rotation.y += 0.05;
                else player.rotation.y += 0.05;
            }
            if (moveRight) {
                if (isDriving) car.rotation.y -= 0.05;
                else player.rotation.y -= 0.05;
            }

            // कैमरा फॉलो सिस्टम
            let targetObj = isDriving ? car : player;
            let cameraOffset = new THREE.Vector3(0, 5, isDriving ? 12 : 8);
            let targetCameraPos = targetObj.position.clone().add(cameraOffset);
            camera.position.lerp(targetCameraPos, 0.1);
            camera.lookAt(targetObj.position);

            // सिक्का उठाने (Collectibles) का चेकिंग
            coins.forEach((coin, index) => {
                if (targetObj.position.distanceTo(coin.position) < 2.5) {
                    scene.remove(coin);
                    coins.splice(index, 1);
                    score += 100;
                    missionsCompleted += 1;
                    document.getElementById('money').innerText = score;
                    document.getElementById('missions').innerText = missionsCompleted;
                }
            });

            renderer.render(scene, camera);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
    </script>
</body>
</html>
