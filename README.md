<!DOCTYPE html>
<html>
<head>
    <title>Space Station Shooter</title>
    <style>
        body { margin: 0; overflow: hidden; }
        canvas { display: block; }
        #hud { position: absolute; top: 10px; left: 10px; color: white; font-family: Arial; }
        #crosshair { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: cyan; font-size: 20px; }
    </style>
</head>
<body>
    <div id="hud">Drones Left: <span id="enemies">5</span> | Health: <span id="health">100</span></div>
    <div id="crosshair">+</div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r134/three.min.js"></script>
    <script>
        // Scene Setup
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Player
        const player = { pos: new THREE.Vector3(0, 1, 0), health: 100, speed: 0.1, canShoot: true };
        camera.position.set(player.pos.x, player.pos.y, player.pos.z);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0x404040);
        scene.add(ambientLight);
        const pointLight = new THREE.PointLight(0xffffff, 1, 100);
        pointLight.position.set(0, 10, 0);
        scene.add(pointLight);

        // Space Station Floor
        const floorGeo = new THREE.PlaneGeometry(50, 50);
        const floorMat = new THREE.MeshBasicMaterial({ color: 0x333333, side: THREE.DoubleSide });
        const floor = new THREE.Mesh(floorGeo, floorMat);
        floor.rotation.x = Math.PI / 2;
        scene.add(floor);

        // Walls
        const wallGeo = new THREE.BoxGeometry(50, 5, 1);
        const wallMat = new THREE.MeshBasicMaterial({ color: 0x555555 });
        const walls = [
            new THREE.Mesh(wallGeo, wallMat), // North
            new THREE.Mesh(wallGeo, wallMat), // South
            new THREE.Mesh(new THREE.BoxGeometry(1, 5, 50), wallMat), // West
            new THREE.Mesh(new THREE.BoxGeometry(1, 5, 50), wallMat)  // East
        ];
        walls[0].position.set(0, 2.5, -25);
        walls[1].position.set(0, 2.5, 25);
        walls[2].position.set(-25, 2.5, 0);
        walls[3].position.set(25, 2.5, 0);
        walls.forEach(wall => scene.add(wall));

        // Enemies (Drones)
        const enemyGeo = new THREE.SphereGeometry(0.5, 16, 16);
        const enemyMat = new THREE.MeshBasicMaterial({ color: 0xff0000 });
        const enemies = [];
        for (let i = 0; i < 5; i++) {
            const enemy = new THREE.Mesh(enemyGeo, enemyMat);
            enemy.position.set(
                (Math.random() - 0.5) * 40,
                0.5,
                (Math.random() - 0.5) * 40
            );
            enemy.alive = true;
            enemies.push(enemy);
            scene.add(enemy);
        }

        // Controls
        const keys = {};
        document.addEventListener('keydown', e => keys[e.code] = true);
        document.addEventListener('keyup', e => keys[e.code] = false);
        document.addEventListener('click', shoot);

        function shoot() {
            if (!player.canShoot) return;
            player.canShoot = false;
            setTimeout(() => player.canShoot = true, 500); // Cooldown

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(enemies);
            if (intersects.length > 0) {
                const enemy = intersects[0].object;
                if (enemy.alive) {
                    enemy.alive = false;
                    scene.remove(enemy);
                    document.getElementById('enemies').innerText = enemies.filter(e => e.alive).length;
                    if (enemies.every(e => !e.alive)) alert('Level Cleared!');
                }
            }
        }

        // Enemy AI
        function updateEnemies() {
            enemies.forEach(enemy => {
                if (!enemy.alive) return;
                const dir = player.pos.clone().sub(enemy.position).normalize();
                enemy.position.add(dir.multiplyScalar(0.03));
                if (enemy.position.distanceTo(player.pos) < 1) {
                    player.health -= 1;
                    document.getElementById('health').innerText = player.health;
                    if (player.health <= 0) alert('Game Over!');
                }
            });
        }

        // Movement
        function movePlayer() {
            const dir = new THREE.Vector3();
            camera.getWorldDirection(dir);
            const side = new THREE.Vector3().crossVectors(dir, new THREE.Vector3(0, 1, 0));

            if (keys['KeyW']) player.pos.add(dir.multiplyScalar(player.speed));
            if (keys['KeyS']) player.pos.sub(dir.multiplyScalar(player.speed));
            if (keys['KeyA']) player.pos.sub(side.multiplyScalar(player.speed));
            if (keys['KeyD']) player.pos.add(side.multiplyScalar(player.speed));

            // Boundary check
            player.pos.x = Math.max(-24, Math.min(24, player.pos.x));
            player.pos.z = Math.max(-24, Math.min(24, player.pos.z));
            camera.position.set(player.pos.x, player.pos.y, player.pos.z);
        }

        // Mouse Look
        document.addEventListener('mousemove', e => {
            const sensitivity = 0.002;
            camera.rotation.y -= e.movementX * sensitivity;
            camera.rotation.x -= e.movementY * sensitivity;
            camera.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, camera.rotation.x));
        });
        document.body.requestPointerLock = document.body.requestPointerLock || document.body.mozRequestPointerLock;
        document.addEventListener('click', () => document.body.requestPointerLock());

        // Game Loop
        function animate() {
            requestAnimationFrame(animate);
            movePlayer();
            updateEnemies();
            renderer.render(scene, camera);
        }
        animate();

        // Resize Handler
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
