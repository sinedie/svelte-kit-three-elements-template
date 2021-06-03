<script>
	export let text = 'Hello world';
	export let position = [0, 0, 0];
	export let rotation = [0, 0, 0];
	export let visible = true;

	export let halign = 'center';
	export let valign = 'center';

	export let fontSize = 1;

	const aligns = {
		left: '0%',
		center: '50%',
		right: '100%'
	};

	let target;

	$: console.log(target);

	$: anchorY = aligns[valign];
	$: anchorX = aligns[halign];
</script>

<three-text
	bind:this={target}
	{visible}
	{text}
	anchor-x={anchorX}
	anchor-y={anchorY}
	font-size={fontSize}
	{position}
	{rotation}
/>
