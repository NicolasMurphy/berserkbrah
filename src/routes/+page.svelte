<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import * as THREE from 'three';

	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera;
	let renderer: THREE.WebGLRenderer;
	let tesseract: THREE.Group;
	let edgesMesh: THREE.LineSegments;

	onMount(() => {
		// Text animations
		gsap.from('.title', { opacity: 0, y: -50, duration: 1.2, ease: 'bounce.out' });
		gsap.from('.subtitle', { opacity: 0, y: 50, duration: 1.2, delay: 0.3, ease: 'bounce.out' });

		// Title glow effect
		gsap.to('.title', {
			textShadow: '0 0 10px #000000, 0 0 20px #ff5577',
			duration: 2.5,
			repeat: -1,
			yoyo: true,
			ease: 'sine.inOut'
		});

		// Subtitle floating effect
		gsap.to('.subtitle', {
			y: 10,
			opacity: 1,
			duration: 2.2,
			yoyo: true,
			ease: 'power1.inOut'
		});

		// Initialize Three.js Scene
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
		renderer = new THREE.WebGLRenderer({ alpha: true });
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.querySelector('.spinning-bg')?.appendChild(renderer.domElement);

		// Define the vertices of a 4D hypercube (tesseract)
		const hypercubeVertices: number[][] = [];
		for (let i = 0; i < 16; i++) {
			hypercubeVertices.push([
				(i & 1 ? 1 : -1), // x
				(i & 2 ? 1 : -1), // y
				(i & 4 ? 1 : -1), // z
				(i & 8 ? 1 : -1)  // w (4th dimension)
			]);
		}

		// Function to project 4D points into 3D space
		function project4Dto3D(v: number[]): THREE.Vector3 {
			const scale = 3.5; // Scale it up to avoid small rendering
			const w = 2 + v[3] * 0.5; // Perspective effect using W coordinate
			return new THREE.Vector3(v[0] / w * scale, v[1] / w * scale, v[2] / w * scale);
		}

		// Create edges for the tesseract (connecting its vertices)
		const edges: number[][] = [];
		for (let i = 0; i < 16; i++) {
			for (let j = 0; j < 4; j++) {
				const neighbor = i ^ (1 << j);
				if (neighbor > i) edges.push([i, neighbor]);
			}
		}

		// Function to create a wireframe for the tesseract
		function createHyperCubeEdges(): THREE.LineSegments {
			const geometry = new THREE.BufferGeometry();
			const positions = new Float32Array(edges.length * 6); // Each edge has 2 points (x, y, z)
			edges.forEach((edge, index) => {
				const p1 = project4Dto3D(hypercubeVertices[edge[0]]);
				const p2 = project4Dto3D(hypercubeVertices[edge[1]]);
				positions.set([...p1.toArray(), ...p2.toArray()], index * 6);
			});

			geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
			const material = new THREE.LineBasicMaterial({ color: 0xff5577, transparent: true, opacity: 0.8 });
			return new THREE.LineSegments(geometry, material);
		}

		// Create 4D projected tesseract
		tesseract = new THREE.Group();
		edgesMesh = createHyperCubeEdges();
		tesseract.add(edgesMesh);
		scene.add(tesseract);
		camera.position.z = 8; // Adjusted camera to fit the larger projection

		// Function to rotate the tesseract in 4D space
		function rotate4D(theta: number, dimA: number, dimB: number) {
			for (let i = 0; i < hypercubeVertices.length; i++) {
				const a = hypercubeVertices[i][dimA];
				const b = hypercubeVertices[i][dimB];
				hypercubeVertices[i][dimA] = a * Math.cos(theta) - b * Math.sin(theta);
				hypercubeVertices[i][dimB] = a * Math.sin(theta) + b * Math.cos(theta);
			}
		}

		// Animation loop with proper updates
		function animate() {
			requestAnimationFrame(animate);

			// Rotate the hypercube in 4D space
			rotate4D(0.001, 0, 3); // Rotate X-W
			rotate4D(0.002, 1, 3); // Rotate Y-W
			rotate4D(0.003, 2, 3); // Rotate Z-W

			// Update the 3D projected positions
			const positions = edgesMesh.geometry.attributes.position.array as Float32Array;
			edges.forEach((edge, index) => {
				const p1 = project4Dto3D(hypercubeVertices[edge[0]]);
				const p2 = project4Dto3D(hypercubeVertices[edge[1]]);
				positions.set([...p1.toArray(), ...p2.toArray()], index * 6);
			});
			edgesMesh.geometry.attributes.position.needsUpdate = true;

			// Render scene
			renderer.render(scene, camera);
		}

		animate();
	});
</script>


<main>
	<div class="spinning-bg"></div>
	<h1 class="title">Berserk Brah</h1>
	<p class="subtitle">Coming soon lol...</p>
</main>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;700&display=swap');

	:global(body) {
		margin: 0;
		padding: 0;
		font-family: 'Poppins', sans-serif;
		background: #0d0d0d;
		color: white;
		text-align: center;
		overflow: hidden;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
	}

	.spinning-bg {
		position: absolute;
		width: 100vw;
		height: 100vh;
		top: 0;
		left: 0;
		z-index: -1;
	}

	.title {
		font-size: 3rem;
		font-weight: bold;
		letter-spacing: 2px;
		text-shadow:
			0 0 10px #000000,
			0 0 20px #ff5577;
	}

	.subtitle {
		font-size: 1.2rem;
		opacity: 0.8;
		text-shadow: 0 0 5px #000000;
	}
</style>
