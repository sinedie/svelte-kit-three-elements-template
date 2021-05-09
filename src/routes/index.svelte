<script>
	import { onMount } from 'svelte';
	import SelectableElement from '$lib/components/selectableBox.svelte';

	let loaded;
	onMount(async () => {
		const three_elements = await import('three-elements');
		loaded = true;
	});

	let camera = 'ortographic';
	let nCubes = 100;
	let domain = 100;
</script>

<div style="height: 90vh">
	{#if loaded}
		<three-game>
			<three-scene background-color="#eee">
				{#if camera == 'perspective'}
					<three-perspective-camera id="camera" position="0, 0, 2000" />
				{:else}
					<three-orthographic-camera id="camera" position="0, 0, 2000" />
				{/if}

				<three-orbit-controls />

				<three-box-buffer-geometry id="geometry" />
				{#each new Array(nCubes) as _}
					<SelectableElement
						geometry="#geometry"
						position={new Array(3).fill(0).map((v) => Math.random(1) * domain)}
						scale="10"
					/>
				{/each}

				<three-ambient-light intensity="0.2" />
				<three-directional-light intensity="0.8" position="10, 10, 50" />
			</three-scene>
		</three-game>
	{/if}
</div>

<button
	on:click={() => {
		camera = camera == 'perspective' ? 'orthographic' : 'perspective';
	}}
>
	Change Camera
</button>
<div>
	Number of cubes
	<input type="range" min="10" max="1000" bind:value={nCubes} />
</div>

<style>
	:global(html, body) {
		margin: 0;
		height: 100%;
		width: 100%;
	}
</style>
