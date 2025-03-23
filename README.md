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
        #hud {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-family: Arial, sans-serif;
            background: rgba(0, 0, 0, 0.5);
            padding: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="hud">Speed: 0 km/h | Altitude: 0 m | Heading: 0°</div>
    <script>
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 10000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
        directionalLight.position.set(10, 50, 10);
        directionalLight.castShadow = true;
        scene.add(directionalLight);

        // Skybox
        const skyboxGeometry = new THREE.SphereGeometry(5000, 32, 32);
        const skyboxMaterial = new THREE.MeshBasicMaterial({ color: 0x87CEEB, side: THREE.BackSide });
        const skybox = new THREE.Mesh(skyboxGeometry, skyboxMaterial);
        scene.add(skybox);

        // Terrain
        const groundGeometry = new THREE.PlaneGeometry(10000, 10000, 256, 256);
        const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x228822, wireframe: false });
        const ground = new THREE.Mesh(groundGeometry, groundMaterial);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Runway
        const runwayGeometry = new THREE.PlaneGeometry(200, 20);
        const runwayMaterial = new THREE.MeshStandardMaterial({ color: 0x333333 });
        const runway = new THREE.Mesh(runwayGeometry, runwayMaterial);
        runway.rotation.x = -Math.PI / 2;
        runway.position.set(0, 0.01, 0);
        scene.add(runway);

        // Aircraft Model
        const fuselageGeometry = new THREE.CylinderGeometry(0.5, 0.5, 5, 32);
        const wingGeometry = new THREE.BoxGeometry(6, 0.2, 1);
        const tailGeometry = new THREE.BoxGeometry(1, 1.5, 0.1);
        const material = new THREE.MeshStandardMaterial({ color: 0xff0000 });
        
        const fuselage = new THREE.Mesh(fuselageGeometry, material);
        fuselage.rotation.z = Math.PI / 2;
        fuselage.castShadow = true;
        scene.add(fuselage);

        const wings = new THREE.Mesh(wingGeometry, material);
        wings.position.set(0, 0, 0);
        wings.castShadow = true;
        scene.add(wings);

        const tail = new THREE.Mesh(tailGeometry, material);
        tail.position.set(-2.5, 0.75, 0);
        scene.add(tail);

        // Physics world
        const world = new CANNON.World();
        world.gravity.set(0, -9.81, 0);

        const planeBody = new CANNON.Body({ mass: 5, shape: new CANNON.Box(new CANNON.Vec3(1, 0.25, 2)) });
        world.addBody(planeBody);

        // Controls
        let keys = {};
        window.addEventListener("keydown", (event) => { keys[event.code] = true; });
        window.addEventListener("keyup", (event) => { keys[event.code] = false; });

        function updateControls() {
            if (keys["ArrowUp"]) planeBody.velocity.z -= 0.5;
            if (keys["ArrowDown"]) planeBody.velocity.z += 0.5;
            if (keys["ArrowLeft"]) planeBody.angularVelocity.y += 0.2;
            if (keys["ArrowRight"]) planeBody.angularVelocity.y -= 0.2;
            if (keys["KeyW"]) planeBody.velocity.y += 0.2;
            if (keys["KeyS"]) planeBody.velocity.y -= 0.2;
        }

        function updateHUD() {
            document.getElementById("hud").innerText = 
                `Speed: ${(planeBody.velocity.z * -10).toFixed(1)} km/h | ` +
                `Altitude: ${planeBody.position.y.toFixed(1)} m | ` +
                `Heading: ${(planeBody.quaternion.y * 180 / Math.PI).toFixed(1)}°`;
        }

        function animate() {
            requestAnimationFrame(animate);
            world.step(1 / 60);
            updateControls();
            updateHUD();
            fuselage.position.copy(planeBody.position);
            fuselage.quaternion.copy(planeBody.quaternion);
            wings.position.copy(planeBody.position);
            wings.quaternion.copy(planeBody.quaternion);
            tail.position.copy(planeBody.position);
            tail.quaternion.copy(planeBody.quaternion);
            renderer.render(scene, camera);
        }

        camera.position.set(0, 5, 15);
        camera.lookAt(fuselage.position);
        animate();
    </script>
</body>
</html>


