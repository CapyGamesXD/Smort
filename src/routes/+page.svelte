<script>
	//@ts-nocheck
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import { Play, Pause, Settings } from 'lucide-svelte';

	let m = $state({ x: 0, y: 0 });

	let objects = [];
	let running = $state(false);
	let curiosity = $state(1);
	let speed = $state(20);
	let moves = 3000;
	let stepLength = $state(3);
	let targetPosX = $state(200);
	let targetPosY = 200;
	let obstacle = $state([{ x: 200, y: 300 }]);
	let startX = 150;
	let startY = 400;
	let stopID;
	let tick = $state(0);
	let settings = $state(true);
	let speedSettings = $state(false);

	function calculateScore(agentDist) {
		const distance = Math.hypot(startX - targetPosX, startY - targetPosY);
		const fitness = 1 - agentDist / distance;

		return fitness;
	}
	function loop() {
		running = true;

		for (let step = 0; step < speed; step++) {
			for (let i = 0; i < objects.length; i++) {
				if (objects[i].step < moves) {
					objects[i].posX += objects[i].genes[objects[i].step].x;
					objects[i].posY += objects[i].genes[objects[i].step].y;
					objects[i].step += 1;

					if (
						obstacle.some((o) => Math.hypot(objects[i].posX - o.x, objects[i].posY - o.y) <= 50)
					) {
						objects[i].posX = startX;
						objects[i].posY = startY;
						objects[i].step = moves;
					}
					if (Math.hypot(objects[i].posX - targetPosX, objects[i].posY - targetPosY) <= 10) {
						if (curiosity > 0.1) {
							curiosity -= 0.05;
							curiosity = Math.round(curiosity * 100) / 100;
						}
						if (speed > 20) {
							speed -= 10;
						}

						for (let j = 0; j < objects.length; j++) {
							objects[j].score = calculateScore(
								Math.hypot(objects[j].posX - targetPosX, objects[j].posY - targetPosY)
							);
							objects[j].posX = startX;
							objects[j].posY = startY;
						}
					}
				} else {
				}
			}
		}

		if (objects.every((o) => o.step >= moves)) {
			for (let j = 0; j < objects.length; j++) {
				objects[j].score = calculateScore(
					Math.hypot(objects[j].posX - targetPosX, objects[j].posY - targetPosY)
				);
			}

			const sortedThing = objects.slice().sort((a, b) => b.score - a.score);
			const bestOnes = sortedThing.slice(0, 20);

			for (let j = 0; j < objects.length; j++) {
				for (let i = 0; i < objects[j].genes.length; i++) {
					const randomBest = bestOnes[Math.floor(Math.random() * bestOnes.length)];
					const otherBest = bestOnes[Math.floor(Math.random() * bestOnes.length)];
					const childX =
						Math.random() > 0.5
							? otherBest.genes[i].x + (Math.random() - 0.5) * curiosity
							: randomBest.genes[i].x + (Math.random() - 0.5) * curiosity;
					const childY =
						Math.random() > 0.5
							? otherBest.genes[i].y + (Math.random() - 0.5) * curiosity
							: randomBest.genes[i].y + (Math.random() - 0.5) * curiosity;
					objects[j].genes[i] = {
						x: childX,
						y: childY
					};
				}
			}
			for (let j = 0; j < objects.length; j++) {
				objects[j].posX = startX;
				objects[j].posY = startY;
				objects[j].step = 0;
			}
			if (curiosity > 0.1) {
				curiosity -= 0.005;
				curiosity = Math.round(curiosity * 100) / 100;
			}
		}
		stopID = requestAnimationFrame(loop);

		tick += 1;
	}

	onMount(() => {
		targetPosX = window.innerWidth / 2;
		startX = window.innerWidth / 2;
		obstacle[0].x = window.innerWidth / 2;
	});

	let dragIndex;
	function handleMousemove(event) {
		if (!running) {
			m.x = event.clientX;
			m.y = event.clientY;
			if (dragIndex != null) {
				obstacle[dragIndex].x = m.x;
				obstacle[dragIndex].y = m.y;
			}
		}
	}

	$effect(() => {
		objects = Array.from({ length: 100 }, () => ({
			posX: startX,
			posY: startY,
			step: 0,
			score: 0,
			genes: Array.from({ length: moves }, () => ({
				x: (Math.random() - 0.5) * stepLength,
				y: (Math.random() - 0.5) * stepLength
			}))
		}));
	});

	function start() {
		loop();
	}
	function stop() {
		cancelAnimationFrame(stopID);
		running = false;
	}

	function hide() {
		settings = false;
		speedSettings = false;
	}
	let obstacleLimit = $state(false);

	$effect(() => {
		if (obstacle.length >= 3) {
			obstacleLimit = true;
		} else {
			obstacleLimit = false;
		}
	});

	function addObs() {
		if (obstacle.length < 3) {
			obstacle = [
				...obstacle,
				{
					x: window.innerWidth / 2 + (Math.random() - 0.5) * 200,
					y: window.innerHeight / 2 + (Math.random() - 0.5) * 200
				}
			];
		}
	}
	function subObs() {
		if (obstacle.length > 1) {
			obstacle.shift();
		}
	}
</script>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
	href="https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,200..1000;1,200..1000&display=swap"
	rel="stylesheet"
/>

<div class="centerdiv">
	{#if settings}
		<div class="middle" transition:fly={{ y: 200, duration: 300 }}>
			<h1>Hey!</h1>
			<p>Configure settings here!</p>
			<p>These can't be changed after training has begun!</p>

			<p>Step Length:</p>
			<input bind:value={stepLength} />
			<p>Curiosity:</p>
			<input bind:value={curiosity} />
			<div class="divider"></div>
			<p>Obstacles: {obstacle.length}</p>
			{#if !obstacleLimit}
				<button onclick={addObs}>Add obstacle</button>
				{#if obstacle.length <= 1}
					<button onclick={subObs} class="greyed">Remove obstacle</button>
				{:else}
					<button onclick={subObs} class="remove">Remove obstacle</button>
				{/if}
			{:else}
				<button onclick={addObs} class="greyed">Add obstacle</button>
				<button onclick={subObs} class="remove">Remove obstacle</button>
			{/if}
			<p>Adding too many obstacles prevents the AI from learning as fast.</p>

			<div class="divider"></div>
			<button class="start" onclick={hide}>Continue</button>
		</div>
	{/if}

	<div
		class="centerdiv"
		onmousemove={handleMousemove}
		onmouseup={() => (dragIndex = null)}
		role="navigation"
	>
		{#if speedSettings}
			<div class="middle" transition:fly={{ y: 200, duration: 300 }}>
				<h1>Hey!</h1>
				<p>Speed</p>
				<input bind:value={speed} />
				<button class="start" onclick={hide}>Continue</button>
			</div>
		{/if}

		<row>
			{#if running}
				<Pause onclick={stop} class="pause" />
			{:else}
				<Play onclick={start} class="play" />
			{/if}

			<Settings class="settings" onclick={() => (speedSettings = true)} />
		</row>

		<svg>
			{#each obstacle as o, i}
				<circle
					onmousedown={() => (dragIndex = i)}
					r="50"
					cx={o.x}
					cy={o.y}
					class="obstacle"
					role="navigation"
				/>
			{/each}

			{#key tick}
				{#each objects as object}
					<image href="/Bee.png" x={object.posX - 8} y={object.posY - 8} width="20" height="20" />
				{/each}
			{/key}

			<circle r="5" cx={targetPosX} cy={targetPosY} class="target" />
		</svg>
	</div>
</div>

Notes for myself: start button can be pressed multiple times. More options in settings.
