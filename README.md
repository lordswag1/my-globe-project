<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Globe with Transparent Background & Red Pins</title>
  <style>
    body {
      margin: 0;
      background: transparent;
    }
    #globeViz {
      width: 100vw;
      height: 100vh;
      background: transparent;
    }
  </style>
</head>
<body>
  <div id="globeViz"></div>

  <script type="module">
    import * as THREE from 'https://esm.sh/three';
    import { TrackballControls } from 'https://esm.sh/three/examples/jsm/controls/TrackballControls.js';
    import createGlobe from 'https://cdn.jsdelivr.net/npm/three-globe@2.16.2/+esm';

    // Scene, camera, and renderer
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setClearColor(0x000000, 0); // Transparent background
    document.getElementById('globeViz').appendChild(renderer.domElement);

    // Create a globe
    const globe = createGlobe()
      .globeImageUrl('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg')
      .bumpImageUrl('https://unpkg.com/three-globe/example/img/earth-topology.png');

    scene.add(globe);

    // Country coordinates (latitude, longitude)
    const countries = [
      { name: 'Malaysia', lat: 4.2105, lon: 101.9758 },
      { name: 'Turkey', lat: 38.9637, lon: 35.2433 },
      { name: 'Thailand', lat: 15.8700, lon: 100.9925 },
      { name: 'Vietnam', lat: 14.0583, lon: 108.2772 },
      { name: 'Cambodia', lat: 12.5657, lon: 104.9910 },
      { name: 'Italy', lat: 41.8719, lon: 12.5674 },
      { name: 'Indonesia', lat: -0.7893, lon: 113.9213 }
    ];

    // Convert lat/lon to 3D sphere coordinates
    function latLonToXYZ(lat, lon, radius = 5) {
      const phi = (90 - lat) * (Math.PI / 180);
      const theta = (lon + 180) * (Math.PI / 180);
      return {
        x: radius * Math.sin(phi) * Math.cos(theta),
        y: radius * Math.cos(phi),
        z: radius * Math.sin(phi) * Math.sin(theta)
      };
    }

    // Create red pins for each country
    const pinGeometry = new THREE.BufferGeometry();
    const pinMaterial = new THREE.PointsMaterial({ color: 'red', size: 0.2 });

    const vertices = [];
    countries.forEach(({ lat, lon }) => {
      const { x, y, z } = latLonToXYZ(lat, lon, 5.1); // Slightly above the globe
      vertices.push(x, y, z);
    });

    pinGeometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
    const pins = new THREE.Points(pinGeometry, pinMaterial);
    scene.add(pins);

    // Lighting
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(5, 3, 5);
    scene.add(light);

    // Camera positioning
    camera.position.z = 10;

    // Controls
    const controls = new TrackballControls(camera, renderer.domElement);
    controls.rotateSpeed = 2;
    controls.zoomSpeed = 1.2;

    // Animation loop
    function animate() {
      requestAnimationFrame(animate);
      globe.rotation.y += 0.002; // Slow rotation
      controls.update();
      renderer.render(scene, camera);
    }
    animate();

    // Handle window resize
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

  </script>
</body>
</html>
