<script>
	//@ts-nocheck
	import { onMount } from 'svelte';

	let objects = $state();

	function loop() {
		for (let i = 0; i < objects.length; i++) {
			objects[i].posX += Math.floor(Math.random() * 10 - 5);
			objects[i].posY += Math.floor(Math.random() * 10 - 5);
		}
		requestAnimationFrame(loop);
	}

	onMount(() => {
		objects = Array.from({ length: 50 }, () => ({
			posX: 150,
			posY: window.innerHeight / 2
		}));
		loop();
	});
</script>

<div class="centerdiv">
	<svg>
		{#each objects as object}
			<circle r="5" cx={object.posX} cy={object.posY} fill="red" />
		{/each}
	</svg>
</div>
