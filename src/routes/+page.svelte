<script>
	//@ts-nocheck
	import { onMount } from 'svelte';

	let objects = $state([]);

	let curiosity = $state(1.25);
	let speed = $state(20);
	let moves = $state(3000);
	const stepLength = 4;
	let targetPosX = $state(200);
	let targetPosY = 200;
	let obstacleX = $state(300);
	let obstacleY = 300;
	let startX = 150;
	let startY = 400;
	let stopID;
	let boost = 0;
	let boostY = 0;
	let boostAmount = 0.05;

	function calculateScore(agentDist) {
		const distance = Math.hypot(startX - targetPosX, startY - targetPosY);
		const fitness = 1 - agentDist / distance;

		return fitness;
	}
	function loop() {
		for (let step = 0; step < speed; step++) {
			for (let i = 0; i < objects.length; i++) {
				if (objects[i].step < moves) {
					if (objects[i].posX < targetPosX) {
						boost = boostAmount;
					} else {
						boost = -boostAmount;
					}
					if (objects[i].posY > targetPosY) {
						boostY = -boostAmount;
					} else {
						boostY = boostAmount;
					}
					objects[i].posX += objects[i].genes[objects[i].step].x + boost;
					objects[i].posY += objects[i].genes[objects[i].step].y + boostY;
					objects[i].step += 1;
					if (Math.hypot(objects[i].posX - obstacleX, objects[i].posY - obstacleY) <= 50) {
						objects[i].posX = startX;
						objects[i].posY = startY;
						objects[i].step = moves;
					}
					if (Math.hypot(objects[i].posX - targetPosX, objects[i].posY - targetPosY) <= 10) {
						if (curiosity > 0.3) {
							curiosity -= 0.05;
							curiosity = curiosity.toFixed(2);
						}
						for (let j = 0; j < objects.length; j++) {
							//Automatically adjust curiosity to learn better.

							objects[j].score =
								calculateScore(
									Math.hypot(objects[j].posX - targetPosX, objects[j].posY - targetPosY)
								) + 0.2;
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
			const bestOnes = sortedThing.slice(0, 8);

			for (let j = 0; j < objects.length; j++) {
				const randomBest = bestOnes[Math.floor(Math.random() * bestOnes.length)];
				for (let i = 0; i < objects[j].genes.length; i++) {
					objects[j].genes[i] = {
						x: randomBest.genes[i].x + (Math.random() - 0.5) * curiosity,
						y: randomBest.genes[i].y + (Math.random() - 0.5) * curiosity
					};
				}
			}
			for (let j = 0; j < objects.length; j++) {
				objects[j].posX = startX;
				objects[j].posY = startY;
				objects[j].step = 0;
			}
		}
		stopID = requestAnimationFrame(loop);
	}

	onMount(() => {
		targetPosX = window.innerWidth / 2;
		startX = window.innerWidth / 2;
		obstacleX = window.innerWidth / 2;

		objects = Array.from({ length: 40 }, () => ({
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
	}
</script>

<div class="centerdiv">
	<svg>
		<circle r="50" cx={obstacleX} cy={obstacleY} class="obstacle" />
		{#each objects as object}
			<image href="/Bee.png" x={object.posX - 8} y={object.posY - 8} width="20" height="20" />
		{/each}

		<circle r="5" cx={targetPosX} cy={targetPosY} class="target" />
	</svg>
	<button onclick={start}>Start</button>
	<button onclick={stop}>Stop</button>
	<input bind:value={speed} />
	<input bind:value={curiosity} />
</div>
