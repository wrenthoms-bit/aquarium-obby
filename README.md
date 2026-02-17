<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Steve The Fish Obby - Prototype</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background-color: #001133; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; touch-action: none; user-select: none; -webkit-user-select: none; }
        #ui-layer {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            pointer-events: none;
            text-shadow: 2px 2px 4px #000000;
            z-index: 10;
        }
        h1 { margin: 0; font-size: 24px; color: #00eeff; }
        .stat-box {
            background: rgba(0, 0, 0, 0.5);
            padding: 10px;
            border-radius: 8px;
            margin-top: 10px;
            border: 1px solid #005577;
        }
        #controls-hint {
            position: absolute;
            bottom: 20px;
            width: 100%;
            text-align: center;
            color: rgba(255, 255, 255, 0.7);
            font-size: 14px;
            pointer-events: none;
        }
        #death-screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(100, 0, 0, 0.5);
            display: none;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 40px;
            font-weight: bold;
            pointer-events: none;
            z-index: 20;
        }

        /* --- Touch Controls --- */
        #joystick-zone {
            position: absolute;
            bottom: 40px;
            left: 40px;
            width: 120px;
            height: 120px;
            background: rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            touch-action: none;
            display: none; /* Hidden on desktop by default */
            z-index: 15;
        }
        #joystick-knob {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 50px;
            height: 50px;
            background: rgba(0, 238, 255, 0.6);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            box-shadow: 0 0 10px rgba(0, 238, 255, 0.5);
        }
        #jump-button {
            position: absolute;
            bottom: 50px;
            right: 50px;
            width: 90px;
            height: 90px;
            background: rgba(255, 68, 68, 0.5);
            border: 2px solid rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            color: white;
            display: none; /* Hidden on desktop by default */
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 18px;
            user-select: none;
            touch-action: none;
            z-index: 15;
            box-shadow: 0 0 10px rgba(255, 68, 68, 0.5);
        }
        #jump-button:active {
            background: rgba(255, 68, 68, 0.8);
            transform: scale(0.95);
        }

        /* Show controls on touch devices */
        @media (pointer: coarse) {
            #joystick-zone, #jump-button { display: flex; }
            #controls-hint { display: none; }
        }
    </style>
</head>
<body>

    <div id="ui-layer">
        <h1>Steve The Fish</h1>
        <div class="stat-box">
            <div>Stage: <span id="stage-display">1</span></div>
            <div>Coins: <span id="coin-display">0</span></div>
        </div>
    </div>

    <div id="controls-hint">WASD / Arrows to Move • SPACE to Jump • Mouse to Rotate Camera</div>
    <div id="death-screen">OUCH!</div>

    <!-- Touch Controls -->
    <div id="joystick-zone">
        <div id="joystick-knob"></div>
    </div>
    <div id="jump-button">JUMP</div>

    <script>
        // --- Game Constants & State ---
        const GRAVITY = 25.0; 
        const JUMP_FORCE = 15.0; 
        const MOVE_SPEED = 18.0; 
        const FRICTION = 4.0; 
        
        let gameState = {
            coins: 0,
            stage: 1,
            checkpoints: [], 
            currentCheckpoint: 0,
            time: 0,
            currentPlatform: null
        };

        // --- Three.js Setup ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x001e36);
        scene.fog = new THREE.Fog(0x001e36, 10, 80);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // --- Lighting ---
        const ambientLight = new THREE.AmbientLight(0x404040, 1.5);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0x00aaff, 0.8);
        dirLight.position.set(50, 100, 50);
        dirLight.castShadow = true;
        dirLight.shadow.camera.left = -50;
        dirLight.shadow.camera.right = 50;
        dirLight.shadow.camera.top = 50;
        dirLight.shadow.camera.bottom = -50;
        dirLight.shadow.mapSize.width = 2048;
        dirLight.shadow.mapSize.height = 2048;
        scene.add(dirLight);

        const playerLight = new THREE.PointLight(0xffaa00, 0.8, 20); // Orange glow for Steve
        scene.add(playerLight);

        // --- Player Setup (Steve Fish Avatar) ---
        const player = new THREE.Group();

        // 1. Body (Orange Round Shape)
        const bodyGeo = new THREE.SphereGeometry(0.5, 32, 32);
        const bodyMat = new THREE.MeshStandardMaterial({ 
            color: 0xff8c00, // Dark Orange
            roughness: 0.4,
            metalness: 0.1
        });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        // Scale to make it slightly fish-shaped (longer)
        body.scale.set(1, 0.9, 1.2); 
        player.add(body);

        // 2. Tail
        const tailGeo = new THREE.ConeGeometry(0.3, 0.6, 16);
        const tailMat = new THREE.MeshStandardMaterial({ color: 0xff8c00 });
        const tail = new THREE.Mesh(tailGeo, tailMat);
        tail.rotation.x = -Math.PI / 2; // Point backwards
        tail.position.z = -0.6;
        player.add(tail);

        // 3. Eyes (Big and Goofy)
        const eyeGeo = new THREE.SphereGeometry(0.18, 16, 16);
        const eyeMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
        const pupilGeo = new THREE.SphereGeometry(0.08, 16, 16);
        const pupilMat = new THREE.MeshBasicMaterial({ color: 0x000000 });

        // Left Eye
        const leftEye = new THREE.Mesh(eyeGeo, eyeMat);
        leftEye.position.set(0.25, 0.1, 0.35);
        const leftPupil = new THREE.Mesh(pupilGeo, pupilMat);
        leftPupil.position.set(0.28, 0.1, 0.48);

        // Right Eye
        const rightEye = new THREE.Mesh(eyeGeo, eyeMat);
        rightEye.position.set(-0.25, 0.1, 0.35);
        const rightPupil = new THREE.Mesh(pupilGeo, pupilMat);
        rightPupil.position.set(-0.28, 0.1, 0.48);

        player.add(leftEye, rightEye, leftPupil, rightPupil);

        // 4. Mouth (Frown)
        const mouthGeo = new THREE.TorusGeometry(0.1, 0.03, 8, 16, Math.PI); // Half circle
        const mouthMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
        const mouth = new THREE.Mesh(mouthGeo, mouthMat);
        mouth.rotation.z = Math.PI ; // Frown
        mouth.position.set(0, -0.15, 0.45);
        player.add(mouth);

        // 5. Legs (It's a walking fish!)
        const legGeo = new THREE.CylinderGeometry(0.06, 0.06, 0.5, 8);
        const legMat = new THREE.MeshStandardMaterial({ color: 0xffaa00 }); 
        
        const leftLeg = new THREE.Mesh(legGeo, legMat);
        leftLeg.position.set(0.15, -0.6, 0);
        
        const rightLeg = new THREE.Mesh(legGeo, legMat);
        rightLeg.position.set(-0.15, -0.6, 0);

        // Feet
        const footGeo = new THREE.BoxGeometry(0.15, 0.05, 0.25);
        const leftFoot = new THREE.Mesh(footGeo, legMat);
        leftFoot.position.set(0.15, -0.85, 0.05);

        const rightFoot = new THREE.Mesh(footGeo, legMat);
        rightFoot.position.set(-0.15, -0.85, 0.05);

        player.add(leftLeg, rightLeg, leftFoot, rightFoot);

        // 6. Arms
        const armGeo = new THREE.CylinderGeometry(0.05, 0.05, 0.4, 8);
        const leftArm = new THREE.Mesh(armGeo, legMat);
        leftArm.position.set(0.5, -0.1, 0);
        leftArm.rotation.z = -Math.PI / 4; // Angle out

        const rightArm = new THREE.Mesh(armGeo, legMat);
        rightArm.position.set(-0.5, -0.1, 0);
        rightArm.rotation.z = Math.PI / 4;

        player.add(leftArm, rightArm);

        player.traverse(obj => { if (obj.isMesh) obj.castShadow = true; });
        scene.add(player);

        // --- Input State ---
        const playerVelocity = new THREE.Vector3(0, 0, 0);
        const playerPosition = new THREE.Vector3(0, 5, 0);
        let isGrounded = false;
        
        const keys = { w: false, a: false, s: false, d: false, space: false };
        let joystickVector = { x: 0, y: 0 }; // x is left/right, y is up/down (forward/back)
        let cameraAngle = 0;
        let isDragging = false;
        let previousMouseX = 0;
        let touchCameraId = null;
        let previousTouchX = 0;

        // --- Keyboard Listeners ---
        document.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key)) keys[key] = true;
            if (e.code === 'Space') keys.space = true;
            if (key === 'arrowup') keys.w = true;
            if (key === 'arrowdown') keys.s = true;
            if (key === 'arrowleft') keys.a = true;
            if (key === 'arrowright') keys.d = true;
        });

        document.addEventListener('keyup', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key)) keys[key] = false;
            if (e.code === 'Space') keys.space = false;
            if (key === 'arrowup') keys.w = false;
            if (key === 'arrowdown') keys.s = false;
            if (key === 'arrowleft') keys.a = false;
            if (key === 'arrowright') keys.d = false;
        });

        // --- Mouse Camera Control ---
        document.addEventListener('mousedown', (e) => { isDragging = true; previousMouseX = e.clientX; });
        document.addEventListener('mouseup', () => { isDragging = false; });
        document.addEventListener('mousemove', (e) => {
            if (isDragging) {
                const delta = e.clientX - previousMouseX;
                cameraAngle -= delta * 0.005;
                previousMouseX = e.clientX;
            }
        });

        // --- Touch Control Logic ---
        const joystickZone = document.getElementById('joystick-zone');
        const joystickKnob = document.getElementById('joystick-knob');
        const jumpBtn = document.getElementById('jump-button');

        // Joystick
        joystickZone.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handleJoystickMove(e.changedTouches[0]);
        }, {passive: false});

        joystickZone.addEventListener('touchmove', (e) => {
            e.preventDefault();
            handleJoystickMove(e.changedTouches[0]);
        }, {passive: false});

        joystickZone.addEventListener('touchend', (e) => {
            e.preventDefault();
            joystickVector = { x: 0, y: 0 };
            joystickKnob.style.transform = `translate(-50%, -50%)`; // Reset knob
        }, {passive: false});

        function handleJoystickMove(touch) {
            const rect = joystickZone.getBoundingClientRect();
            const centerX = rect.left + rect.width / 2;
            const centerY = rect.top + rect.height / 2;
            
            // Calculate delta
            let dx = touch.clientX - centerX;
            let dy = touch.clientY - centerY;
            
            // Limit knob distance
            const distance = Math.sqrt(dx*dx + dy*dy);
            const maxRadius = rect.width / 2 - 25; // Zone radius minus knob radius
            
            if (distance > maxRadius) {
                const ratio = maxRadius / distance;
                dx *= ratio;
                dy *= ratio;
            }
            
            // Move knob
            joystickKnob.style.transform = `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`;
            
            // Normalize vector (-1 to 1)
            joystickVector.x = dx / maxRadius;
            joystickVector.y = dy / maxRadius;
        }

        // Jump Button
        jumpBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            keys.space = true;
        }, {passive: false});

        jumpBtn.addEventListener('touchend', (e) => {
            e.preventDefault();
            keys.space = false;
        }, {passive: false});

        // Touch Camera (Drag anywhere else)
        document.addEventListener('touchstart', (e) => {
            // Find a touch that isn't on the controls
            for (let i = 0; i < e.changedTouches.length; i++) {
                const t = e.changedTouches[i];
                if (t.target !== joystickZone && t.target !== joystickKnob && t.target !== jumpBtn) {
                    touchCameraId = t.identifier;
                    previousTouchX = t.clientX;
                    break;
                }
            }
        }, {passive: false});

        document.addEventListener('touchmove', (e) => {
            for (let i = 0; i < e.changedTouches.length; i++) {
                const t = e.changedTouches[i];
                if (t.identifier === touchCameraId) {
                    const delta = t.clientX - previousTouchX;
                    cameraAngle -= delta * 0.008; // Sensitivity
                    previousTouchX = t.clientX;
                    break;
                }
            }
        }, {passive: false});

        document.addEventListener('touchend', (e) => {
            for (let i = 0; i < e.changedTouches.length; i++) {
                if (e.changedTouches[i].identifier === touchCameraId) {
                    touchCameraId = null;
                }
            }
        }, {passive: false});


        // --- Level Building Functions ---
        const platforms = [];
        const hazards = [];
        const coins = [];
        const timedHazards = [];
        const movingPlatforms = [];

        function createPlatform(x, y, z, w, h, d, color, type = 'static') {
            const geo = new THREE.BoxGeometry(w, h, d);
            const mat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.8, transparent: true, opacity: 0.9 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            mesh.receiveShadow = true; mesh.castShadow = true;
            scene.add(mesh);
            const collider = { mesh: mesh, width: w, height: h, depth: d, type: type };
            platforms.push(collider);
            if(type === 'moving') movingPlatforms.push({ ...collider, originalY: y, offset: Math.random() * Math.PI * 2 });
            return mesh;
        }

        function createHazard(x, y, z, w, h, d, type = 'static') {
            const geo = new THREE.BoxGeometry(w, h, d);
            const mat = new THREE.MeshStandardMaterial({ color: 0xff3300, emissive: 0xff0000, emissiveIntensity: 0.5 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            scene.add(mesh);
            const hazard = { mesh: mesh, width: w, height: h, depth: d, active: true };
            hazards.push(hazard);
            if (type === 'timed') timedHazards.push(hazard);
        }

        function createCoin(x, y, z) {
            const geo = new THREE.TorusGeometry(0.3, 0.1, 8, 16);
            const mat = new THREE.MeshStandardMaterial({ color: 0xffd700, metalness: 1, roughness: 0.2 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            scene.add(mesh);
            coins.push(mesh);
        }

        function createCheckpoint(x, y, z, stageNum) {
            const geo = new THREE.CylinderGeometry(1, 1, 0.2, 16);
            const mat = new THREE.MeshStandardMaterial({ color: 0x888888 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            scene.add(mesh);
            gameState.checkpoints.push({ pos: new THREE.Vector3(x, y + 2, z), stage: stageNum });
        }

        // --- Build Level ---
        createPlatform(0, 0, 0, 10, 1, 10, 0xaaaaaa); 
        gameState.checkpoints.push({ pos: new THREE.Vector3(0, 5, 0), stage: 1 });

        for (let i = 1; i <= 5; i++) {
            createPlatform(0, i * 2, i * -6.5, 4, 0.5, 6, 0x228b22); 
            if (i % 2 !== 0) createCoin(0, (i * 2) + 1.5, i * -6.5);
        }

        createPlatform(0, 10, -40, 6, 0.5, 6, 0x32cd32, 'moving');
        createPlatform(0, 10, -50, 8, 1, 8, 0x888888);
        createCheckpoint(0, 10.5, -50, 2);
        createPlatform(-6, 12, -58, 4, 1, 4, 0x666666);
        createPlatform(6, 14, -66, 4, 1, 4, 0x666666);
        createPlatform(0, 13, -62, 15, 0.5, 2, 0x555555); 
        createHazard(0, 13.5, -62, 2, 0.5, 2); 
        createPlatform(0, 15, -72, 4, 1, 4, 0x666666);
        createPlatform(0, 15, -80, 4, 1, 4, 0x666666);
        createPlatform(0, 15, -88, 4, 1, 4, 0x666666);
        createHazard(0, 15, -76, 2, 4, 2, 'timed');
        createHazard(0, 15, -84, 2, 4, 2, 'timed');
        createCoin(0, 17, -80);
        createPlatform(0, 15, -100, 10, 1, 10, 0x333333);
        createCheckpoint(0, 15.5, -100, 3);

        let angle = 0;
        let height = 15;
        for(let i = 0; i < 10; i++) {
            angle += 0.6; height += 2.0;
            const x = Math.sin(angle) * 10;
            const z = -100 + Math.cos(angle) * 10; 
            createPlatform(x, height, z, 4, 0.5, 4, 0x552200);
            if (i % 2 === 0) createHazard(x, height + 0.3, z, 1, 0.5, 1, 'timed');
            else createCoin(x, height + 1.5, z);
        }
        createPlatform(0, height + 3, -100, 15, 1, 15, 0xffd700); 

        // --- Game Logic ---
        function updatePhysics(dt) {
            // Sticky Logic
            if (gameState.currentPlatform) {
                const p = gameState.currentPlatform;
                const pMinX = p.mesh.position.x - p.width/2 - 0.6;
                const pMaxX = p.mesh.position.x + p.width/2 + 0.6;
                const pMinZ = p.mesh.position.z - p.depth/2 - 0.6;
                const pMaxZ = p.mesh.position.z + p.depth/2 + 0.6;

                if (playerPosition.x >= pMinX && playerPosition.x <= pMaxX &&
                    playerPosition.z >= pMinZ && playerPosition.z <= pMaxZ) {
                    if (keys.space) {
                        playerVelocity.y = JUMP_FORCE;
                        isGrounded = false;
                        gameState.currentPlatform = null;
                    } else {
                        isGrounded = true;
                        playerVelocity.y = 0;
                        playerPosition.y = p.mesh.position.y + p.height/2 + 0.5;
                    }
                } else {
                    isGrounded = false;
                    gameState.currentPlatform = null;
                }
            } else {
                if (keys.space && isGrounded) {
                     playerVelocity.y = JUMP_FORCE;
                     isGrounded = false;
                }
            }

            if (!isGrounded) playerVelocity.y -= GRAVITY * dt;

            // Movement Inputs (Mix Keyboard + Joystick)
            let moveDir = new THREE.Vector3(0, 0, 0);
            
            // Keyboard
            if (keys.w) moveDir.z -= 1;
            if (keys.s) moveDir.z += 1;
            if (keys.a) moveDir.x -= 1;
            if (keys.d) moveDir.x += 1;

            // Joystick (Add to keyboard input)
            moveDir.x += joystickVector.x;
            moveDir.z += joystickVector.y; // Y from 2D input maps to Z in 3D world

            if (moveDir.length() > 0) {
                // Clamp length to 1 to prevent double speed if using both inputs
                if(moveDir.length() > 1) moveDir.normalize();
                
                // Rotate vector based on camera angle
                moveDir.applyAxisAngle(new THREE.Vector3(0, 1, 0), cameraAngle);
                
                playerVelocity.x += moveDir.x * MOVE_SPEED * dt;
                playerVelocity.z += moveDir.z * MOVE_SPEED * dt;
            }

            // Friction
            const dampFactor = Math.exp(-FRICTION * dt);
            playerVelocity.x *= dampFactor;
            playerVelocity.z *= dampFactor;
            
            if (playerVelocity.y < -20) playerVelocity.y = -20;

            if (Math.abs(playerVelocity.x) > 0.1 || Math.abs(playerVelocity.z) > 0.1) {
                const angle = Math.atan2(playerVelocity.x, playerVelocity.z);
                player.rotation.y = angle;
            }

            let nextPos = playerPosition.clone();
            nextPos.x += playerVelocity.x * dt;
            nextPos.z += playerVelocity.z * dt;
            if (!isGrounded) nextPos.y += playerVelocity.y * dt;

            // Collision
            if (!isGrounded && playerVelocity.y <= 0) {
                for (let p of platforms) {
                    let pMinX = p.mesh.position.x - p.width/2 - 0.5;
                    let pMaxX = p.mesh.position.x + p.width/2 + 0.5;
                    let pMinY = p.mesh.position.y - p.height/2 - 0.5;
                    let pMaxY = p.mesh.position.y + p.height/2 + 0.5;
                    let pMinZ = p.mesh.position.z - p.depth/2 - 0.5;
                    let pMaxZ = p.mesh.position.z + p.depth/2 + 0.5;
                    if (
                        nextPos.x > pMinX && nextPos.x < pMaxX &&
                        nextPos.z > pMinZ && nextPos.z < pMaxZ &&
                        playerPosition.y >= pMaxY - 0.5 && nextPos.y <= pMaxY + 0.2
                    ) {
                        nextPos.y = pMaxY + 0.5; 
                        playerVelocity.y = 0;
                        isGrounded = true;
                        gameState.currentPlatform = p; 
                        break; 
                    }
                }
            }

            hazards.forEach(h => {
                if (!h.active) return;
                if (Math.abs(h.mesh.position.x - nextPos.x) < (h.width/2 + 0.4) &&
                    Math.abs(h.mesh.position.y - nextPos.y) < (h.height/2 + 0.4) &&
                    Math.abs(h.mesh.position.z - nextPos.z) < (h.depth/2 + 0.4)) die();
            });

            for (let i = coins.length - 1; i >= 0; i--) {
                let c = coins[i];
                if (c.position.distanceTo(nextPos) < 1.5) {
                    scene.remove(c);
                    coins.splice(i, 1);
                    gameState.coins++;
                    document.getElementById('coin-display').innerText = gameState.coins;
                }
            }

            gameState.checkpoints.forEach(cp => {
                if (cp.pos.distanceTo(nextPos) < 3) {
                    if (gameState.stage < cp.stage) {
                        gameState.stage = cp.stage;
                        gameState.currentCheckpoint = gameState.stage - 1;
                        document.getElementById('stage-display').innerText = gameState.stage;
                    }
                }
            });

            if (nextPos.y < -20) { die(); return; }

            playerPosition.copy(nextPos);
            player.position.copy(playerPosition);
        }

        function die() {
            const deathScreen = document.getElementById('death-screen');
            deathScreen.style.display = 'flex';
            const cp = gameState.checkpoints[gameState.currentCheckpoint].pos;
            playerPosition.set(cp.x, cp.y + 2, cp.z);
            playerVelocity.set(0, 0, 0);
            gameState.currentPlatform = null; 
            setTimeout(() => { deathScreen.style.display = 'none'; }, 1000);
        }

        function updateEnvironment(time) {
            coins.forEach(c => { c.rotation.y += 0.05; });
            movingPlatforms.forEach(p => { p.mesh.position.y = p.originalY + Math.sin(time * 2 + p.offset) * 2; });
            const cycle = Math.floor(time / 2);
            timedHazards.forEach((h, index) => {
                const isOn = (cycle + index) % 2 === 0;
                h.active = isOn;
                h.mesh.material.opacity = isOn ? 1 : 0.2;
                h.mesh.material.transparent = true;
                h.mesh.material.emissiveIntensity = isOn ? 1 : 0.1;
            });
        }

        function updateCamera() {
            const dist = 10;
            const camX = playerPosition.x + Math.sin(cameraAngle) * dist;
            const camZ = playerPosition.z + Math.cos(cameraAngle) * dist;
            const camY = playerPosition.y + 5;
            camera.position.set(camX, camY, camZ);
            camera.lookAt(playerPosition);
            playerLight.position.copy(playerPosition);
        }

        let lastTime = performance.now();
        function animate() {
            requestAnimationFrame(animate);
            const now = performance.now();
            const dt = Math.min((now - lastTime) / 1000, 0.1); 
            lastTime = now;
            gameState.time = now / 1000;

            updatePhysics(dt);
            updateEnvironment(gameState.time);
            updateCamera();

            renderer.render(scene, camera);
        }

        gameState.checkpoints[0].pos.copy(new THREE.Vector3(0, 5, 0));
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

    </script>
</body>
</html>
