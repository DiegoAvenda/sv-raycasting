<script>
	import { onMount } from 'svelte';

	const VELOCITY = 3;
	const FOV = Math.PI / 3;
	const RAYS = 100;
	const STAGGER_FRAME = 6;

	const CANVAS_SIZE = 600;

	let canvas;
	let canvasWidth = CANVAS_SIZE;
	let canvasHeight = CANVAS_SIZE;
	let squareSize = CANVAS_SIZE / 10;
	let playerRadius = squareSize / 4;
	let angle = $state(0);
	let enemyX = $state(squareSize * 7);
	let enemyY = $state(squareSize * 7);
	let gameFrame = $state(0);
	let playerX = $state(squareSize * 1.5);
	let playerY = $state(squareSize * 1.5);
	let middleY = CANVAS_SIZE / 2;
	let enemyFrameX = $state(0);
	let enemyFrameY = $state(0);
	let playerFrameX = $state(0);
	let isPlayerFiring = $state(false);
	let enemyImageLoaded = $state(false);
	let keysPressed = $state({
		ArrowRight: false,
		ArrowLeft: false,
		ArrowUp: false,
		ArrowDown: false
	});

	const ENEMY_SPRITE_WIDTH = 71;
	const ENEMY_SPRITE_HEIGHT = 66;
	const ENEMY_SPRITE_COLUMNS = 4;
	const ENEMY_ATTACKING_SPRITE_COLUMNS = 2;
	const PLAYER_SPRITE_WIDTH = 204;
	const PLAYER_SPRITE_HEIGHT = 216;
	const PLAYER_SPRITE_COLUMNS = 3;
	const PLAYER_FRAME_Y = 0;

	let enemyImage;
	let playerImage;

	const map = [
		[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 1, 0, 1],
		[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		[1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
	];

	const cos = (angle) => Math.cos(angle);
	const sin = (angle) => Math.sin(angle);

	function calculateDistance(x1, y1, x2, y2) {
		const dx = x2 - x1;
		const dy = y2 - y1;
		return Math.sqrt(dx * dx + dy * dy);
	}

	function getEnemyDistanceAndDirection() {
		const dx = playerX - enemyX;
		const dy = playerY - enemyY;
		const distance = calculateDistance(enemyX, enemyY, playerX, playerY);
		return { dx, dy, distance };
	}

	function detectCollision(x, y) {
		const cellX = Math.floor(x / squareSize);
		const cellY = Math.floor(y / squareSize);
		return map[cellY]?.[cellX] === 1;
	}

	function checkCircleCollision(newX, newY) {
		const numPoints = 8;
		for (let i = 0; i < numPoints; i++) {
			const checkAngle = (i / numPoints) * 2 * Math.PI;
			const checkX = newX + cos(checkAngle) * playerRadius;
			const checkY = newY + sin(checkAngle) * playerRadius;

			if (detectCollision(checkX, checkY)) {
				return true;
			}
		}
		return false;
	}

	function handleKeydown(event) {
		if (event.key in keysPressed) {
			keysPressed[event.key] = true;
		}
		if (event.key === 'f') {
			isPlayerFiring = true;
		}
	}

	function handleKeyup(event) {
		if (event.key in keysPressed) {
			keysPressed[event.key] = false;
		}
	}

	let currentEnemyState = $state('walking');

	const enemyStateHandlers = {
		walking: () => {
			const { dx, dy, distance } = getEnemyDistanceAndDirection();
			const ENEMY_VELOCITY = 1.5;

			if (distance <= playerRadius + 15) {
				currentEnemyState = 'attacking';
				enemyFrameX = 0;
				return;
			}

			if (distance > 0) {
				const normalizedDx = dx / distance;
				const normalizedDy = dy / distance;

				const newEnemyX = enemyX + normalizedDx * ENEMY_VELOCITY;
				const newEnemyY = enemyY + normalizedDy * ENEMY_VELOCITY;

				if (!detectCollision(newEnemyX, enemyY)) {
					enemyX = newEnemyX;
				}
				if (!detectCollision(enemyX, newEnemyY)) {
					enemyY = newEnemyY;
				}

				if (gameFrame % STAGGER_FRAME === 0) {
					enemyFrameX = (enemyFrameX + 1) % ENEMY_SPRITE_COLUMNS;
				}
			}
		},

		attacking: () => {
			const { distance } = getEnemyDistanceAndDirection();
			if (distance > playerRadius + 20) {
				currentEnemyState = 'walking';
				enemyFrameX = 0;
				return;
			}
			enemyFrameY = 1;
			if (gameFrame % (STAGGER_FRAME * 4) === 0) {
				enemyFrameX = (enemyFrameX + 1) % ENEMY_ATTACKING_SPRITE_COLUMNS;
			}
		},
		taking_damage: () => {
			enemyFrameY = 2;
			if (gameFrame % STAGGER_FRAME === 0) {
				enemyFrameX++;
				if (enemyFrameX >= ENEMY_SPRITE_COLUMNS) {
					currentEnemyState = 'dead';
					enemyFrameX = 3;
				}
			}
		},
		dead: () => {
			enemyFrameY = 2;
			enemyFrameX = 3;
		}
	};

	function updatePlayerRotation() {
		if (keysPressed.ArrowRight) angle += 0.05;
		if (keysPressed.ArrowLeft) angle -= 0.05;
	}

	function updatePlayerMovement() {
		let deltaX = 0;
		let deltaY = 0;

		if (keysPressed.ArrowUp) {
			deltaX = cos(angle) * VELOCITY;
			deltaY = sin(angle) * VELOCITY;
		}
		if (keysPressed.ArrowDown) {
			deltaX = -cos(angle) * VELOCITY;
			deltaY = -sin(angle) * VELOCITY;
		}

		const nextX = playerX + deltaX;
		const nextY = playerY + deltaY;

		if (!checkCircleCollision(nextX, playerY)) playerX = nextX;
		if (!checkCircleCollision(playerX, nextY)) playerY = nextY;
	}

	function updatePlayerAnimation() {
		if (isPlayerFiring && gameFrame % STAGGER_FRAME === 0) {
			playerFrameX++;
			if (playerFrameX >= PLAYER_SPRITE_COLUMNS) {
				isPlayerFiring = false;
				playerFrameX = 0;
			}
		}
	}

	function drawPlayer(ctx) {
		const PLAYER_IMAGE_SIZE = 128 * 2;
		ctx.drawImage(
			playerImage,
			playerFrameX * PLAYER_SPRITE_WIDTH,
			PLAYER_FRAME_Y * PLAYER_SPRITE_HEIGHT,
			PLAYER_SPRITE_WIDTH,
			PLAYER_SPRITE_HEIGHT,
			CANVAS_SIZE / 2 - PLAYER_IMAGE_SIZE / 2,
			CANVAS_SIZE - PLAYER_IMAGE_SIZE,
			PLAYER_IMAGE_SIZE,
			PLAYER_IMAGE_SIZE
		);
	}

	function damageEnemy() {
		if (currentEnemyState !== 'dead') {
			currentEnemyState = 'taking_damage';
			enemyFrameX = 0;
		}
	}

	function processPlayerShooting() {
		if (!isPlayerFiring || currentEnemyState === 'dead' || currentEnemyState === 'taking_damage')
			return;

		const dx = enemyX - playerX;
		const dy = enemyY - playerY;
		const enemyAngle = Math.atan2(dy, dx);
		const angleDifference = enemyAngle - angle;

		if (angleDifference < FOV / 32) {
			damageEnemy();
		}
	}

	function drawEnemy(ctx, enemySpriteX, enemySpriteY, enemySize) {
		ctx.drawImage(
			enemyImage,
			enemyFrameX * ENEMY_SPRITE_WIDTH,
			enemyFrameY * ENEMY_SPRITE_HEIGHT,
			ENEMY_SPRITE_WIDTH,
			ENEMY_SPRITE_HEIGHT,
			enemySpriteX,
			enemySpriteY,
			enemySize,
			enemySize
		);
	}

	function castRay(rayAngle) {
		const rayDirX = cos(rayAngle);
		const rayDirY = sin(rayAngle);

		const deltaDistX = Math.abs(1 / rayDirX);
		const deltaDistY = Math.abs(1 / rayDirY);

		let mapX = Math.floor(playerX / squareSize);
		let mapY = Math.floor(playerY / squareSize);

		let stepX, stepY;
		let sideDistX, sideDistY;

		if (rayDirX < 0) {
			stepX = -1;
			sideDistX = (playerX / squareSize - mapX) * deltaDistX;
		} else {
			stepX = 1;
			sideDistX = (mapX + 1.0 - playerX / squareSize) * deltaDistX;
		}

		if (rayDirY < 0) {
			stepY = -1;
			sideDistY = (playerY / squareSize - mapY) * deltaDistY;
		} else {
			stepY = 1;
			sideDistY = (mapY + 1.0 - playerY / squareSize) * deltaDistY;
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

			if (map[mapY]?.[mapX] === 1) {
				hit = true;
			}
		}

		let perpWallDist;
		if (side === 0) {
			perpWallDist = sideDistX - deltaDistX;
		} else {
			perpWallDist = sideDistY - deltaDistY;
		}

		const hitX = playerX + perpWallDist * rayDirX * squareSize;
		const hitY = playerY + perpWallDist * rayDirY * squareSize;

		const distance = perpWallDist * squareSize;

		const correctedDistance = distance * cos(rayAngle - angle);

		return {
			x: hitX,
			y: hitY,
			distance: correctedDistance,
			side: side
		};
	}

	function drawWallColumn(ctx, columnIndex, wallDistance) {
		const columnHeight = (squareSize * CANVAS_SIZE) / wallDistance;
		const columnWidth = CANVAS_SIZE / RAYS;
		const columnX = columnIndex * columnWidth;

		ctx.fillRect(columnX, CANVAS_SIZE / 2 - columnHeight / 2, columnWidth, columnHeight);
	}

	function renderEnemy(ctx, wallDistancePerRay) {
		const dx = enemyX - playerX;
		const dy = enemyY - playerY;
		const enemyDistance = calculateDistance(playerX, playerY, enemyX, enemyY);
		const enemyAngle = Math.atan2(dy, dx);

		processPlayerShooting();

		let angleDifference = enemyAngle - angle;
		while (angleDifference > Math.PI) angleDifference -= 2 * Math.PI;
		while (angleDifference < -Math.PI) angleDifference += 2 * Math.PI;

		if (Math.abs(angleDifference) < FOV / 2) {
			const enemySize = (squareSize * CANVAS_SIZE) / enemyDistance;
			const enemySpriteX = (angleDifference / FOV + 0.5) * CANVAS_SIZE - enemySize / 2;
			const enemySpriteY = middleY - enemySize / 2;

			drawEnemy(ctx, enemySpriteX, enemySpriteY, enemySize);
		}
	}

	function performRaycasting(ctx) {
		const rayStep = FOV / RAYS;
		const wallDistancePerRay = [];

		for (let i = 0; i < RAYS; i++) {
			const rayAngle = angle - FOV / 2 + i * rayStep;
			const wallResult = castRay(rayAngle);
			wallDistancePerRay[i] = wallResult.distance;
			drawWallColumn(ctx, i, wallResult.distance);
		}

		renderEnemy(ctx, wallDistancePerRay);
	}

	function gameLoop() {
		const ctx = canvas.getContext('2d');
		ctx.clearRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

		updatePlayerRotation();
		updatePlayerMovement();
		enemyStateHandlers[currentEnemyState]?.();
		updatePlayerAnimation();
		gameFrame++;

		performRaycasting(ctx);
		drawPlayer(ctx);

		requestAnimationFrame(gameLoop);
	}

	onMount(() => {
		playerImage = new Image();
		playerImage.src = '/weapon.png';
		enemyImage = new Image();
		enemyImage.src = '/boss.png';

		gameLoop();
	});
</script>

<canvas bind:this={canvas} width={CANVAS_SIZE} height={CANVAS_SIZE}></canvas>
<svelte:window onkeydown={handleKeydown} onkeyup={handleKeyup} />
