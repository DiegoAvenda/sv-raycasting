<script>
	import { onMount } from 'svelte';

	let canvas;
	let angle = $state(Math.PI);
	const squareSize = 50;

    let playerX = $state(0)
    let playerY

    const cos = (angle) => Math.cos(angle)
    const sin = (angle) => Math.sin(angle)

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
		ctx.arc(squareSize * 2, squareSize * 2, squareSize / 2, 0, Math.PI * 2);
		ctx.stroke();
	}

    function dda(ctx) {
const rayDirX = cos(angle)
const rayDirY = sin(angle)

let deltaX = 1 / rayDirX
let deltaY = 1 / rayDirY

let mapX = playerX / squareSize
let mapY = playerY / squareSize

let firstX
let firstY

if (rayDirX < 1) {
    firstX = (playerX / squareSize - mapX) * deltaX
} else {
    firstX = ()
}

    }

	function loop() {
		const ctx = canvas.getContext('2d');
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		drawMap(ctx);
		drawPlayer(ctx);

		requestAnimationFrame(loop);
	}

	onMount(() => {
		loop();
	});
</script>

<canvas bind:this={canvas} width="600" height="600"></canvas>
