# OG-Game
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flight Simulator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cannon-es/0.20.0/cannon-es.js"></script>
    <style>
        body { margin: 0; overflow: hidden; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script>
        // Scene setup
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); // Sky blue

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
        directionalLight.position.set(5, 10, 5);
        directionalLight.castShadow = true;
        scene.add(directionalLight);

        // Plane (Basic Model)
        const planeGeometry = new THREE.BoxGeometry(2, 0.5, 5);
        const planeMaterial = new THREE.MeshStandardMaterial({ color: 0x0077ff, metalness: 0.5, roughness: 0.3 });
        const plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.castShadow = true;
        scene.add(plane);

        // Physics world
        const world = new CANNON.World();
        world.gravity.set(0, -9.81, 0);

        const planeBody = new CANNON.Body({ mass: 5, shape: new CANNON.Box(new CANNON.Vec3(1, 0.25, 2.5)) });
        world.addBody(planeBody);

        // Ground
        const groundGeometry = new THREE.PlaneGeometry(500, 500);
        const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x228822, side: THREE.DoubleSide });
        const ground = new THREE.Mesh(groundGeometry, groundMaterial);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Physics ground
        const groundBody = new CANNON.Body({ mass: 0, shape: new CANNON.Plane() });
        groundBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0);
        world.addBody(groundBody);

        // Controls
        let keys = {};
        window.addEventListener("keydown", (event) => { keys[event.code] = true; });
        window.addEventListener("keyup", (event) => { keys[event.code] = false; });

        function updateControls() {
            if (keys["ArrowUp"]) planeBody.velocity.z -= 0.1; // Forward
            if (keys["ArrowDown"]) planeBody.velocity.z += 0.1; // Backward
            if (keys["ArrowLeft"]) planeBody.angularVelocity.y += 0.1; // Turn left
            if (keys["ArrowRight"]) planeBody.angularVelocity.y -= 0.1; // Turn right
            if (keys["KeyW"]) planeBody.velocity.y += 0.1; // Ascend
            if (keys["KeyS"]) planeBody.velocity.y -= 0.1; // Descend
        }

        // Animation loop
        function animate() {
            requestAnimationFrame(animate);
            world.step(1 / 60);
            updateControls();
            plane.position.copy(planeBody.position);
            plane.quaternion.copy(planeBody.quaternion);
            renderer.render(scene, camera);
        }

        camera.position.set(0, 5, 10);
        camera.lookAt(plane.position);

        animate();
    </script>
</body>
</html>
