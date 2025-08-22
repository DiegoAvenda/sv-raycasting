<script>
	import { onMount } from 'svelte';

	let canvas;

	const mapSize = 8;
	const tileSize = 50;
	const worldMap = [
		[1, 1, 1, 1, 1, 1, 1, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 1, 1, 1, 1, 1, 1, 1]
	];

	let player = {
		x: 1.5 * tileSize,
		y: 1.5 * tileSize,
		angle: 0
	};

	function castRayDDA(angle) {
		let rayDirX = Math.cos(angle);
		let rayDirY = Math.sin(angle);

		let mapX = Math.floor(player.x / tileSize);
		let mapY = Math.floor(player.y / tileSize);

		let deltaDistX = Math.abs(1 / rayDirX);
		let deltaDistY = Math.abs(1 / rayDirY);

		let stepX, stepY;
		let sideDistX, sideDistY;

		if (rayDirX < 0) {
			stepX = -1;
			sideDistX = (player.x / tileSize - mapX) * deltaDistX;
		} else {
			stepX = 1;
			sideDistX = (mapX + 1 - player.x / tileSize) * deltaDistX;
		}
		if (rayDirY < 0) {
			stepY = -1;
			sideDistY = (player.y / tileSize - mapY) * deltaDistY;
		} else {
			stepY = 1;
			sideDistY = (mapY + 1 - player.y / tileSize) * deltaDistY;
		}

		let hit = false;
		let side;

		while (!hit) {
			if (sideDistX < sideDistY) {
				sideDistX += deltaDistX;
				mapX += stepX;
				side = 0;
			} else {
				sideDistY += deltaDistY;
				mapY += stepY;
				side = 1;
			}
			if (worldMap[mapY][mapX] === 1) hit = true;
		}

		let perpWallDist;
		if (side === 0) {
			perpWallDist = (mapX - player.x / tileSize + (1 - stepX) / 2) / rayDirX;
		} else {
			perpWallDist = (mapY - player.y / tileSize + (1 - stepY) / 2) / rayDirY;
		}

		let hitX = player.x + perpWallDist * rayDirX * tileSize;
		let hitY = player.y + perpWallDist * rayDirY * tileSize;

		return { hitX, hitY };
	}

	function render(ctx) {
		ctx.clearRect(0, 0, 400, 400);

		// Mapa
		ctx.strokeStyle = '#555';
		for (let y = 0; y < mapSize; y++) {
			for (let x = 0; x < mapSize; x++) {
				ctx.strokeRect(x * tileSize, y * tileSize, tileSize, tileSize);
				if (worldMap[y][x] === 1) {
					ctx.fillStyle = '#888';
					ctx.fillRect(x * tileSize, y * tileSize, tileSize, tileSize);
				}
			}
		}

		ctx.fillStyle = 'blue';
		ctx.beginPath();
		ctx.arc(player.x, player.y, 5, 0, Math.PI * 2);
		ctx.fill();

		let ray = castRayDDA(player.angle);
		ctx.strokeStyle = 'red';
		ctx.beginPath();
		ctx.moveTo(player.x, player.y);
		ctx.lineTo(ray.hitX, ray.hitY);
		ctx.stroke();
	}

	function loop() {
		const ctx = canvas.getContext('2d');
		render(ctx);
		requestAnimationFrame(loop);
	}

	onMount(() => {
		loop();
	});
</script>

<canvas bind:this={canvas} width="400" height="400"></canvas>

<style>
	canvas {
		display: block;
		margin: auto;
		background: #222;
		border: 1px solid #555;
	}
</style>
