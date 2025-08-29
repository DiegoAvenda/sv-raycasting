<script>
	import { onMount } from 'svelte';

	let canvas;
	let ctx;

	const mapSize = 8;
	const tileSize = 50;
	const map = [
		[1, 1, 1, 1, 1, 1, 1, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1],
		[1, 1, 1, 1, 1, 1, 1, 1]
	];

	// Variables del jugador
	let playerX = 0;
	let playerY = 0;
	let angle = 0;

	// Funciones auxiliares
	const cos = (angle) => Math.cos(angle);
	const sin = (angle) => Math.sin(angle);

	function castRay(startX, startY, rayAngle) {
		const rayDirX = cos(rayAngle);
		const rayDirY = sin(rayAngle);
		const deltaDistX = rayDirX === 0 ? Infinity : Math.abs(1 / rayDirX);
		const deltaDistY = rayDirY === 0 ? Infinity : Math.abs(1 / rayDirY);

		let mapX = Math.floor(startX / tileSize);
		let mapY = Math.floor(startY / tileSize);
		let stepX, stepY;
		let sideDistX, sideDistY;

		if (rayDirX < 0) {
			stepX = -1;
			sideDistX = (startX / tileSize - mapX) * deltaDistX;
		} else {
			stepX = 1;
			sideDistX = (mapX + 1 - startX / tileSize) * deltaDistX;
		}

		if (rayDirY < 0) {
			stepY = -1;
			sideDistY = (startY / tileSize - mapY) * deltaDistY;
		} else {
			stepY = 1;
			sideDistY = (mapY + 1 - startY / tileSize) * deltaDistY;
		}

		let hit = false;
		let side; // Nueva variable: 0 para lado X (vertical), 1 para lado Y (horizontal)

		while (!hit) {
			if (sideDistX < sideDistY) {
				sideDistX += deltaDistX;
				mapX += stepX;
				side = 0; // Avanzó en X → lado vertical
			} else {
				sideDistY += deltaDistY;
				mapY += stepY;
				side = 1; // Avanzó en Y → lado horizontal
			}

			if (map[mapY]?.[mapX] === 1) {
				hit = true;
			}
		}

		// Calcular distancia perpendicular (simplificado)
		let perpWallDist;
		if (side === 0) {
			perpWallDist = sideDistX - deltaDistX;
		} else {
			perpWallDist = sideDistY - deltaDistY;
		}

		// Calcular punto exacto de intersección
		const hitX = startX + perpWallDist * rayDirX * tileSize;
		const hitY = startY + perpWallDist * rayDirY * tileSize;

		return { x: hitX, y: hitY };
	}

	function render() {
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		// Dibujar mapa
		ctx.strokeStyle = '#555';
		for (let y = 0; y < mapSize; y++) {
			for (let x = 0; x < mapSize; x++) {
				ctx.strokeRect(x * tileSize, y * tileSize, tileSize, tileSize);
				if (map[y][x] === 1) {
					ctx.fillStyle = '#888';
					ctx.fillRect(x * tileSize, y * tileSize, tileSize, tileSize);
				}
			}
		}

		// Dibujar jugador
		ctx.fillStyle = 'blue';
		ctx.beginPath();
		ctx.arc(playerX, playerY, 5, 0, Math.PI * 2);
		ctx.fill();

		// Lanzar y dibujar rayo
		const ray = castRay(playerX, playerY, angle);
		ctx.strokeStyle = '#ff0000';
		ctx.beginPath();
		ctx.moveTo(playerX, playerY);
		ctx.lineTo(ray.x, ray.y);
		ctx.stroke();
	}

	function loop() {
		render();
		requestAnimationFrame(loop);
	}

	onMount(() => {
		ctx = canvas.getContext('2d');

		// Inicializar posición del jugador
		playerX = 1.5 * tileSize;
		playerY = 1.5 * tileSize;

		loop();
	});
</script>

<canvas bind:this={canvas} width="400" height="400"></canvas>
