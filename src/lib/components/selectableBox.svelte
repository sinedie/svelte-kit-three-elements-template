<script>
	/**
	 * @type {string}
	 */
	export let color = 'hotpink';
	/**
	 * @type {string}
	 */
	export let hoverColor = 'yellow';
	/**
	 * @type {string}
	 */
	export let selectionColor = 'red';
	/**
	 * @type {number}
	 */
	export let scale = 1;
	/**
	 * @type {list}
	 */
	export let position = [0, 0, 0];
	/**
	 * If you pass an ID like '#geometry', it reuses the geometry instead of creatting a new one
	 * @type {'box' | string}
	 */
	export let geometry = 'box';
	/**
	 * @type {bool}
	 */
	export let selected = false;

	const onpointerenter = ({ target }) => {
		const { object, game } = target;
		/* three-elements bug here! */
		document.body.style.cursor = 'pointer';
		object.material.color.set(hoverColor);
		game.requestFrame();
	};

	const onpointerleave = ({ target }) => {
		const { object, game } = target;
		document.body.style.cursor = 'default';
		object.material.color.set(selected ? selectionColor : color);
		game.requestFrame();
	};

	const onpointerdown = ({ target }) => {
		const { object, game } = target;
		object.material.color.set(selectionColor);
		selected = !selected;
		game.requestFrame();
	};
</script>

<three-mesh {geometry} {position} {scale} {onpointerenter} {onpointerleave} {onpointerdown}>
	<!-- Only adds geometry if dont pass an ID to an reusable geometry -->
	{#if geometry === 'box' || geometry === undefined}
		<three-box-buffer-geometry />
	{/if}
	<three-mesh-standard-material {color} />
</three-mesh>
