<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import * as THREE from 'three';

	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera;
	let renderer: THREE.WebGLRenderer;
	let tesseract: THREE.Group;

	onMount(() => {
		// Text animations
		gsap.from('.title', { opacity: 0, y: -50, duration: 1.2, ease: 'bounce.out' });
		gsap.from('.subtitle', { opacity: 0, y: 50, duration: 1.2, delay: 0.3, ease: 'bounce.out' });

		// Title dark glow effect
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

		// Function to create a more subtle wireframe tesseract
		function createSubtleWireframe(
			scale: number,
			color: number,
			opacity: number
		): THREE.LineSegments {
			const geometry = new THREE.BoxGeometry(2, 2, 2);
			const edges = new THREE.EdgesGeometry(geometry);
			const material = new THREE.LineBasicMaterial({
				color: color,
				opacity: opacity,
				transparent: true
			});
			const wireframe = new THREE.LineSegments(edges, material);
			wireframe.scale.set(scale, scale, scale);
			return wireframe;
		}

		// Create and layer a more subtle tesseract
		tesseract = new THREE.Group();
		tesseract.add(createSubtleWireframe(1.5, 0xff5577, 0.5)); // Outer glow with lower opacity
		tesseract.add(createSubtleWireframe(1.0, 0xff7799, 0.4)); // Middle layer
		tesseract.add(createSubtleWireframe(0.5, 0xff99bb, 0.3)); // Inner depth
		scene.add(tesseract);
		camera.position.z = 5;

		// Slower and more subtle rotation animation
		function animate() {
			requestAnimationFrame(animate);
			tesseract.rotation.x += 0.001;
			tesseract.rotation.y += 0.002;
			tesseract.rotation.z += 0.003;
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
