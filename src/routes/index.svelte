<script>
	import { onMount } from 'svelte';
	import { Box3, Sphere } from 'three';
	import { browser } from '$app/env';

	import SelectableElement from '$lib/components/SelectableBox.svelte';
	import TextElement from '$lib/components/TextElement.svelte';

	onMount(async () => {
		if (browser) {
			await import('three-elements');
			await import('@three-elements/text');
		}
	});

	let game;
	let scene;
	let controls;
	let perspCamera;
	let orthoCamera;
	let camera = 'perspective';
	let nCubes = 100;
	let domain = 100;

	$: seceneProps = {
		'ref:camera': `#${camera}`
	};

	// TODO check again, on perspective camera sometimes mess up raypicking
	/**
	 * Zoom camera to selection
	 * @param {THREE.Camera} camera Current camera of scene
	 * @param {THREE.Controls} controls Controls of scene (orbit probably)
	 * @param {[THREE.Object]} selection List of objects to fit
	 */
	function zoomCameraToSelection(camera, controls, selection) {
		const box = new Box3();
		for (const object of selection) box.expandByObject(object);
		const sphere = box.getBoundingSphere(new Sphere());

		let distance = sphere.radius;

		if (camera.isPerspectiveCamera) {
			// Perspective camera
			const fitHeightDistance = sphere.radius / (2 * Math.atan((Math.PI * camera.fov) / 360));
			const fitWidthDistance = fitHeightDistance / camera.aspect;
			distance = 2 * Math.max(fitHeightDistance, fitWidthDistance);

			controls.maxDistance = distance * 10;
			camera.near = distance / 100;
			camera.far = distance * 100;
		} else {
			// Orthographic camera
			camera.zoom = 1 / sphere.radius;
		}

		const direction = controls.target
			.clone()
			.sub(camera.position)
			.normalize()
			.multiplyScalar(distance);

		controls.target.copy(sphere.center);
		camera.position.copy(controls.target).sub(direction);
		camera.updateProjectionMatrix();
		controls.update();
		game.requestFrame();
	}

	/**
	 * Zoom camera to fit all elements on scene
	 */
	function zoomExtent() {
		zoomCameraToSelection(scene.camera, controls.object, scene.object.children);
	}

	/**
	 * Toggle between perspective and orthographic camera
	 */
	function toggleCamera() {
		orthoCamera.object.position.copy(scene.camera.position);
		perspCamera.object.position.copy(scene.camera.position);
		camera = camera == 'perspective' ? 'orthographic' : 'perspective';
		orthoCamera.object.updateProjectionMatrix();
		perspCamera.object.updateProjectionMatrix();
		controls.object.update();
		game.requestFrame();
	}
</script>

<div class="viewport">
	{#if browser}
		<three-game bind:this={game}>
			<three-scene bind:this={scene} background-color="#000" {...seceneProps}>
				<three-perspective-camera
					bind:this={perspCamera}
					id="perspective"
					far={700}
					near={0.1}
					up={[0, 0, 1]}
					position={[0, 0, 100]}
				/>

				<three-orthographic-camera
					bind:this={orthoCamera}
					id="orthographic"
					far={500}
					near={-500}
					up={[0, 0, 1]}
					position={[0, 0, 100]}
				/>
				<three-orbit-controls bind:this={controls} />

				<!-- Cannot use reusable geometry cause raypicking dies -->
				<!-- <three-box-buffer-geometry id="geometry" /> -->
				{#each new Array(nCubes) as _}
					<SelectableElement
						position={new Array(3).fill(0).map((v) => Math.random(1) * domain)}
						scale="10"
					/>
				{/each}

				<!-- Text test -->
				<TextElement />

				<three-ambient-light intensity="0.2" />
				<three-directional-light intensity="0.8" position="10, 10, 50" />
			</three-scene>
		</three-game>
	{/if}
</div>

<div class="menu">
	<button on:click={toggleCamera}> Change Camera </button>
	<button on:click={zoomExtent}> Zoom extent </button>
	<div>
		Number of cubes {nCubes}
		<input type="number" min="1" max="1000" bind:value={nCubes} />
	</div>
</div>

<style>
	:global(html, body) {
		margin: 0;
		height: 100%;
		width: 100%;
		color: white;
	}
	.viewport {
		height: 100vh;
	}
	.menu {
		display: flex;
		position: fixed;
		right: 0;
		bottom: 0;
	}
</style>
