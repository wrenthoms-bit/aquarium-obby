<!DOCTYPE html>

<html lang="en">

<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Aquatic Fire Obby - Prototype</title>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

    <style>

        body { margin: 0; overflow: hidden; background-color: #001133; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        #ui-layer {

            position: absolute;

            top: 20px;

            left: 20px;

            color: white;

            pointer-events: none;

            text-shadow: 2px 2px 4px #000000;

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

        }

    </style>

</head>

<body>


    <div id="ui-layer">

        <h1>Aquatic Fire Obby</h1>

        <div class="stat-box">

            <div>Stage: <span id="stage-display">1</span></div>

            <div>Coins: <span id="coin-display">0</span></div>

        </div>

    </div>


    <div id="controls-hint">WASD / Arrows to Move • SPACE to Jump • Mouse to Rotate Camera</div>

    <div id="death-screen">OUCH!</div>


    <script>

        // --- Game Constants & State ---

        const GRAVITY = 25.0;

        const JUMP_FORCE = 15.0; // Increased jump height

        const MOVE_SPEED = 18.0; // Increased movement speed

        const FRICTION = 4.0; // Reduced friction for better air momentum

       

        let gameState = {

            coins: 0,

            stage: 1,

            checkpoints: [],

            currentCheckpoint: 0,

            time: 0,

            currentPlatform: null // Track which platform we are on

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


        const playerLight = new THREE.PointLight(0xff00ff, 0.8, 20);

        scene.add(playerLight);


        // --- Player Setup (Jellyfish Avatar) ---

        const player = new THREE.Group();


        // 1. The Bell (Head)

        const bellGeo = new THREE.SphereGeometry(0.6, 32, 16, 0, Math.PI * 2, 0, Math.PI / 2);

        const bellMat = new THREE.MeshStandardMaterial({

            color: 0xff66cc,

            roughness: 0.1,

            metalness: 0.1,

            transparent: true,

            opacity: 0.7,

            emissive: 0x440044,

            emissiveIntensity: 0.2

        });

        const bell = new THREE.Mesh(bellGeo, bellMat);

        bell.position.y = 0.0;

        player.add(bell);


        // 2. Tentacles

        const tentacleGeo = new THREE.CylinderGeometry(0.04, 0.02, 0.8, 8);

        const tentacleMat = new THREE.MeshStandardMaterial({

            color: 0xffccff,

            transparent: true,

            opacity: 0.8

        });


        for(let i = 0; i < 5; i++) {

            const t = new THREE.Mesh(tentacleGeo, tentacleMat);

            const angle = (i / 5) * Math.PI * 2;

            t.position.set(Math.sin(angle) * 0.3, -0.4, Math.cos(angle) * 0.3);

            player.add(t);

        }


        // 3. Eyes

        const eyeGeo = new THREE.SphereGeometry(0.12, 16, 16);

        const eyeMat = new THREE.MeshBasicMaterial({ color: 0xffffff });

        const pupilMat = new THREE.MeshBasicMaterial({ color: 0x000000 });


        const leftEye = new THREE.Mesh(eyeGeo, eyeMat);

        leftEye.position.set(0.2, 0.2, 0.45);

        const leftPupil = new THREE.Mesh(new THREE.SphereGeometry(0.05), pupilMat);

        leftPupil.position.set(0.22, 0.2, 0.54);

       

        const rightEye = new THREE.Mesh(eyeGeo, eyeMat);

        rightEye.position.set(-0.2, 0.2, 0.45);

        const rightPupil = new THREE.Mesh(new THREE.SphereGeometry(0.05), pupilMat);

        rightPupil.position.set(-0.22, 0.2, 0.54);


        player.add(leftEye, rightEye, leftPupil, rightPupil);


        player.traverse(obj => { if (obj.isMesh) obj.castShadow = true; });

        scene.add(player);


        // Physics State

        const playerVelocity = new THREE.Vector3(0, 0, 0);

        const playerPosition = new THREE.Vector3(0, 5, 0);

        let isGrounded = false;

       

        // --- Input Handling ---

        const keys = { w: false, a: false, s: false, d: false, space: false };

        let cameraAngle = 0;

        let isDragging = false;

        let previousMouseX = 0;


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


        document.addEventListener('mousedown', (e) => { isDragging = true; previousMouseX = e.clientX; });

        document.addEventListener('mouseup', () => { isDragging = false; });

        document.addEventListener('mousemove', (e) => {

            if (isDragging) {

                const delta = e.clientX - previousMouseX;

                cameraAngle -= delta * 0.005;

                previousMouseX = e.clientX;

            }

        });


        // --- Level Building Functions ---

        const platforms = [];

        const hazards = [];

        const coins = [];

        const timedHazards = [];

        const movingPlatforms = [];


        function createPlatform(x, y, z, w, h, d, color, type = 'static') {

            const geo = new THREE.BoxGeometry(w, h, d);

            const mat = new THREE.MeshStandardMaterial({

                color: color,

                roughness: 0.8,

                transparent: true,

                opacity: 0.9

            });

            const mesh = new THREE.Mesh(geo, mat);

            mesh.position.set(x, y, z);

            mesh.receiveShadow = true;

            mesh.castShadow = true;

            scene.add(mesh);

           

            const collider = { mesh: mesh, width: w, height: h, depth: d, type: type };

            platforms.push(collider);

           

            if(type === 'moving') {

                movingPlatforms.push({ ...collider, originalY: y, offset: Math.random() * Math.PI * 2 });

            }

            return mesh;

        }


        function createHazard(x, y, z, w, h, d, type = 'static') {

            const geo = new THREE.BoxGeometry(w, h, d);

            const mat = new THREE.MeshStandardMaterial({

                color: 0xff3300,

                emissive: 0xff0000,

                emissiveIntensity: 0.5

            });

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

            // Increased depth from 4 to 6 to make landing easier

            createPlatform(0, i * 2, i * -8, 4, 0.5, 6, 0x228b22);

            if (i % 2 !== 0) createCoin(0, (i * 2) + 1.5, i * -8);

        }


        createPlatform(0, 10, -48, 6, 0.5, 6, 0x32cd32, 'moving');


        createPlatform(0, 10, -60, 8, 1, 8, 0x888888);

        createCheckpoint(0, 10.5, -60, 2);


        createPlatform(-6, 12, -70, 4, 1, 4, 0x666666);

        createPlatform(6, 14, -80, 4, 1, 4, 0x666666);

       

        createPlatform(0, 13, -75, 15, 0.5, 2, 0x555555);

        createHazard(0, 13.5, -75, 2, 0.5, 2);


        createPlatform(0, 15, -90, 4, 1, 4, 0x666666);

        createPlatform(0, 15, -100, 4, 1, 4, 0x666666);

        createPlatform(0, 15, -110, 4, 1, 4, 0x666666);


        createHazard(0, 15, -95, 2, 4, 2, 'timed');

        createHazard(0, 15, -105, 2, 4, 2, 'timed');

        createCoin(0, 17, -100);


        createPlatform(0, 15, -125, 10, 1, 10, 0x333333);

        createCheckpoint(0, 15.5, -125, 3);


        let angle = 0;

        let height = 15;

        for(let i = 0; i < 10; i++) {

            angle += 0.8;

            height += 2.5;

            const x = Math.sin(angle) * 10;

            const z = -125 + Math.cos(angle) * 10;

           

            createPlatform(x, height, z, 4, 0.5, 4, 0x552200);

           

            if (i % 2 === 0) {

                 createHazard(x, height + 0.3, z, 1, 0.5, 1, 'timed');

            } else {

                createCoin(x, height + 1.5, z);

            }

        }


        createPlatform(0, height + 5, -125, 15, 1, 15, 0xffd700);


        // --- Game Logic ---


        function updatePhysics(dt) {

            // 1. Check if we are grounded on a known platform (Sticky Logic)

            if (gameState.currentPlatform) {

                const p = gameState.currentPlatform;

               

                // Bounds check with slight forgiveness

                const pMinX = p.mesh.position.x - p.width/2 - 0.6;

                const pMaxX = p.mesh.position.x + p.width/2 + 0.6;

                const pMinZ = p.mesh.position.z - p.depth/2 - 0.6;

                const pMaxZ = p.mesh.position.z + p.depth/2 + 0.6;


                // Check if player is still roughly over the platform

                if (playerPosition.x >= pMinX && playerPosition.x <= pMaxX &&

                    playerPosition.z >= pMinZ && playerPosition.z <= pMaxZ) {

                   

                    if (keys.space) {

                        // JUMP: Break sticky bond

                        playerVelocity.y = JUMP_FORCE;

                        isGrounded = false;

                        gameState.currentPlatform = null;

                    } else {

                        // STICK: Lock Y to platform top

                        isGrounded = true;

                        playerVelocity.y = 0;

                        // Smoothly follow platform Y (handles moving platforms)

                        playerPosition.y = p.mesh.position.y + p.height/2 + 0.5;

                    }

                } else {

                    // WALK OFF: Apply gravity again

                    isGrounded = false;

                    gameState.currentPlatform = null;

                }

            } else {

                // Air control

                if (keys.space && isGrounded) {

                     playerVelocity.y = JUMP_FORCE;

                     isGrounded = false;

                }

            }


            // 2. Apply Gravity (Only if not grounded)

            if (!isGrounded) {

                playerVelocity.y -= GRAVITY * dt;

            }


            // 3. Movement Inputs (Horizontal)

            let moveDir = new THREE.Vector3(0, 0, 0);

            if (keys.w) moveDir.z -= 1;

            if (keys.s) moveDir.z += 1;

            if (keys.a) moveDir.x -= 1;

            if (keys.d) moveDir.x += 1;


            if (moveDir.length() > 0) {

                moveDir.normalize();

                moveDir.applyAxisAngle(new THREE.Vector3(0, 1, 0), cameraAngle);

               

                playerVelocity.x += moveDir.x * MOVE_SPEED * dt;

                playerVelocity.z += moveDir.z * MOVE_SPEED * dt;

            }


            // Friction

            const dampFactor = Math.exp(-FRICTION * dt);

            playerVelocity.x *= dampFactor;

            playerVelocity.z *= dampFactor;

           

            if (playerVelocity.y < -20) playerVelocity.y = -20;


            // Rotation

            if (Math.abs(playerVelocity.x) > 0.1 || Math.abs(playerVelocity.z) > 0.1) {

                const angle = Math.atan2(playerVelocity.x, playerVelocity.z);

                player.rotation.y = angle;

            }


            // 4. Propose new position (X/Z always, Y only if airborne)

            let nextPos = playerPosition.clone();

            nextPos.x += playerVelocity.x * dt;

            nextPos.z += playerVelocity.z * dt;

           

            if (!isGrounded) {

                nextPos.y += playerVelocity.y * dt;

            } else {

                // If grounded, Y is already set by the Sticky Logic above

            }


            // 5. Collision Detection (Landing on new platforms)

            if (!isGrounded && playerVelocity.y <= 0) { // Only check if falling

                for (let p of platforms) {

                    let pMinX = p.mesh.position.x - p.width/2 - 0.5;

                    let pMaxX = p.mesh.position.x + p.width/2 + 0.5;

                    let pMinY = p.mesh.position.y - p.height/2 - 0.5;

                    let pMaxY = p.mesh.position.y + p.height/2 + 0.5;

                    let pMinZ = p.mesh.position.z - p.depth/2 - 0.5;

                    let pMaxZ = p.mesh.position.z + p.depth/2 + 0.5;


                    // Check if entering from top

                    if (

                        nextPos.x > pMinX && nextPos.x < pMaxX &&

                        nextPos.z > pMinZ && nextPos.z < pMaxZ &&

                        playerPosition.y >= pMaxY - 0.5 && nextPos.y <= pMaxY + 0.2

                    ) {

                        // LANDED!

                        nextPos.y = pMaxY + 0.5;

                        playerVelocity.y = 0;

                        isGrounded = true;

                        gameState.currentPlatform = p; // Lock to this platform

                        break; // Stop checking

                    }

                }

            }


            // Hazards

            hazards.forEach(h => {

                if (!h.active) return;

                if (Math.abs(h.mesh.position.x - nextPos.x) < (h.width/2 + 0.4) &&

                    Math.abs(h.mesh.position.y - nextPos.y) < (h.height/2 + 0.4) &&

                    Math.abs(h.mesh.position.z - nextPos.z) < (h.depth/2 + 0.4)) {

                    die();

                }

            });


            // Coins

            for (let i = coins.length - 1; i >= 0; i--) {

                let c = coins[i];

                if (c.position.distanceTo(nextPos) < 1.5) {

                    scene.remove(c);

                    coins.splice(i, 1);

                    gameState.coins++;

                    document.getElementById('coin-display').innerText = gameState.coins;

                }

            }


            // Checkpoints

            gameState.checkpoints.forEach(cp => {

                if (cp.pos.distanceTo(nextPos) < 3) {

                    if (gameState.stage < cp.stage) {

                        gameState.stage = cp.stage;

                        gameState.currentCheckpoint = gameState.stage - 1;

                        document.getElementById('stage-display').innerText = gameState.stage;

                    }

                }

            });


            if (nextPos.y < -20) {

                die();

                return;

            }


            playerPosition.copy(nextPos);

            player.position.copy(playerPosition);

        }


        function die() {

            const deathScreen = document.getElementById('death-screen');

            deathScreen.style.display = 'flex';

           

            const cp = gameState.checkpoints[gameState.currentCheckpoint].pos;

            playerPosition.set(cp.x, cp.y + 2, cp.z);

            playerVelocity.set(0, 0, 0);

            gameState.currentPlatform = null; // Reset platform lock

           

            setTimeout(() => {

                deathScreen.style.display = 'none';

            }, 1000);

        }


        function updateEnvironment(time) {

            coins.forEach(c => { c.rotation.y += 0.05; });


            movingPlatforms.forEach(p => {

                p.mesh.position.y = p.originalY + Math.sin(time * 2 + p.offset) * 2;

            });


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
