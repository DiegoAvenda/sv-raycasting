<script>
	import { onMount } from 'svelte';

	let canvas;
	let angle = $state(0);
	const squareSize = 50;

	let playerX = $state(squareSize * 2);
	let playerY = $state(squareSize * 1.5);

	const cos = (angle) => Math.cos(angle);
	const sin = (angle) => Math.sin(angle);

	const map = [
		[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 1, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
	];

	function drawMap(ctx) {
		for (let y = 0; y < map.length; y++) {
			for (let x = 0; x < map[y].length; x++) {
				if (map[y][x] === 1) {
					ctx.fillRect(squareSize * x, squareSize * y, squareSize, squareSize);
				} else {
					ctx.strokeRect(squareSize * x, squareSize * y, squareSize, squareSize);
				}
			}
		}
	}

	function drawPlayer(ctx) {
		ctx.beginPath();
		ctx.arc(playerX, playerY, squareSize / 2, 0, Math.PI * 2);
		ctx.stroke();
	}

	function castRay() {
		const rayDirX = cos(angle);
		const rayDirY = sin(angle);

		let deltaDistX = Math.abs(1 / rayDirX);
		let deltaDistY = Math.abs(1 / rayDirY);

		let mapX = Math.floor(playerX / squareSize);
		let mapY = Math.floor(playerY / squareSize);

		let sideDistX;
		let sideDistY;
		let stepX;
		let stepY;

		if (rayDirX < 0) {
			sideDistX = (playerX / squareSize - mapX) * deltaDistX;
			stepX = -1;
		} else {
			sideDistX = (mapX + 1 - playerX / squareSize) * deltaDistX;
			stepX = 1;
		}

		if (rayDirY < 0) {
			sideDistY = (playerY / squareSize - mapY) * deltaDistY;
			stepY = -1;
		} else {
			sideDistY = (mapY + 1 - playerY / squareSize) * deltaDistY;
			stepY = 1;
		}

		let hit = false;
		let side;

		while (!hit) {
			if (sideDistX < sideDistY) {
				sideDistX += deltaDistX;
				side = 0;
				mapX += stepX;
			} else {
				sideDistY += deltaDistY;
				side = 1;
				mapY += stepY;
			}

			if (map[mapY][mapX] === 1) {
				hit = true;
			}
		}

		let perpWallDist;

		if (side === 0) {
			perpWallDist = sideDistX - deltaDistX;
		} else {
			perpWallDist = sideDistY - deltaDistY;
		}

		let hitX = playerX + perpWallDist * rayDirX * squareSize;
		let hitY = playerY + perpWallDist * rayDirY * squareSize;

		return {
			x: hitX,
			y: hitY
		};
	}

	function loop() {
		const ctx = canvas.getContext('2d');
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		drawMap(ctx);
		drawPlayer(ctx);

		const ray = castRay();

		ctx.beginPath();
		ctx.moveTo(playerX, playerY);
		ctx.lineTo(ray.x, ray.y);
		ctx.stroke();

		requestAnimationFrame(loop);
	}

	onMount(() => {
		loop();
	});
</script>

<canvas bind:this={canvas} width="600" height="600"></canvas>
