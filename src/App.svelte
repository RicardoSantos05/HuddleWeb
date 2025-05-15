<script lang="ts">
	import { fade } from 'svelte/transition';

	let x = 300;
	let y = 300;
	let vx = 0;
	let vy = 0;
	let ax = 0;
	let ay = 0;

	let keys = { w: false, a: false, s: false, d: false };
	const speed = 0.4;
	const maxSpeed = 4;
	const drag = 0.98;

	let gameOver = false;
	let showEndScreen = false;

	let fishX = 0;
	let fishY = 0;
	let showFish = false;
	let caughtCount = 0;
	let message = '';
	let showMessage = false;

	let scoreSaved = false;

	let facingLeft = false;

	let seals = [];
	let sealSpeed = 6;

	let showStartScreen = true;

	 import { onMount } from 'svelte';
  let joystickX = 0;
  let joystickY = 0;
  let isDragging = false;
  let startX = 0;
  let startY = 0;

  function handleTouchStart(event) {
    isDragging = true;
    const touch = event.touches[0];
    startX = touch.clientX;
    startY = touch.clientY;
  }

  function handleTouchMove(event) {
    if (!isDragging) return;
    const touch = event.touches[0];
    joystickX = touch.clientX - startX;
    joystickY = touch.clientY - startY;
  }

  function handleTouchEnd() {
    isDragging = false;
    joystickX = 0;
    joystickY = 0;
  }

	function startGame() {
		showStartScreen = false;
		setTimeout(spawnFish, 3000);
		updatePosition();
	}

function touchControl(direction, isPressed) {
  if (direction in keys) {
    keys[direction] = isPressed;
  }
}
	function spawnFish() {
		fishX = Math.random() * (window.innerWidth - 64);
		fishY = Math.random() * (window.innerHeight - 64);
		showFish = true;
	}

	function showCatchMessage() {
		showMessage = true;
		message = `${caughtCount}`;
		setTimeout(() => {
			showMessage = false;
		}, 1500);
	}

	function checkSealCollision() {
	if (gameOver) return;

	const penguinCenterX = x + 50;
	const penguinCenterY = y + 50;

	for (let seal of seals) {
		const sealCenterX = seal.x + 225;
		const sealCenterY = seal.y + 225;
		const dx = penguinCenterX - sealCenterX;
		const dy = penguinCenterY - sealCenterY;
		const distance = Math.sqrt(dx * dx + dy * dy);

		if (distance < 100) {
			gameOver = true;
			fadeOut();
			break;
		}
	}
	}

	function checkCollision() {
		if (!showFish) return;

		const penguinCenterX = x + 32;
		const penguinCenterY = y + 32;
		const fishCenterX = fishX + 24;
		const fishCenterY = fishY + 24;

		const dx = penguinCenterX - fishCenterX;
		const dy = penguinCenterY - fishCenterY;
		const distance = Math.sqrt(dx * dx + dy * dy);

		const collisionRadius = 60;

		if (distance < collisionRadius) {
			caughtCount++;
			showFish = false;
			showCatchMessage();
			sealSpeed += 0.3;
			setTimeout(spawnFish, 3000);
		}
	}

		function fadeOut() {
		setTimeout(() => {
			showEndScreen = true;
			loadLeaderboard();
		}, 300); // pequeno atraso opcional para suavizar
	}

	function spawnSeal() {
		const fromLeft = Math.random() < 0.5;
		const y = Math.random() * (window.innerHeight - 128); // posição vertical aleatória
		const direction = fromLeft ? 1 : -1;
		const x = fromLeft ? -200 : window.innerWidth + 200; // fora do ecrã

		seals.push({
			x,
			y,
			direction,
			speed: sealSpeed + Math.random() * 1.5 // velocidade ligeiramente variável
		});
	}


	function updatePosition() {
	// Se estiver a usar o joystick, converte os valores para aceleração
	if (isDragging) {
		const normalizedX = joystickX / 40; // 40 é o limite do stick
		const normalizedY = joystickY / 40;
		ax = normalizedX * speed;
		ay = normalizedY * speed;
	} else {
		ax = 0;
		ay = 0;
		if (keys.w) ay = -speed;
		if (keys.s) ay = speed;
		if (keys.a) ax = -speed;
		if (keys.d) ax = speed;
	}

	vx += ax;
	vy += ay;

	// Limita velocidade
	vx = Math.max(-maxSpeed, Math.min(maxSpeed, vx));
	vy = Math.max(-maxSpeed, Math.min(maxSpeed, vy));

	// Aplica drag
	vx *= drag;
	vy *= drag;

	x += vx;
	y += vy;

	const penguinWidth = 100;
	const penguinHeight = 100;

	x = Math.max(0, Math.min(window.innerWidth - penguinWidth, x));
	y = Math.max(0, Math.min(window.innerHeight - penguinHeight, y));

	// Atualiza direção do pinguim
	if (vx < -0.5) facingLeft = true;
	else if (vx > 0.5) facingLeft = false;

	if (!gameOver) {
		checkCollision();
		checkSealCollision();
	}

	// Movimento das focas
	for (let i = seals.length - 1; i >= 0; i--) {
		seals[i].x += seals[i].speed * seals[i].direction;

		if (
			(seals[i].direction === 1 && seals[i].x > window.innerWidth + 200) ||
			(seals[i].direction === -1 && seals[i].x < -200)
		) {
			seals.splice(i, 1);
		}
	}

	requestAnimationFrame(updatePosition);
}

	onMount(() => {
		window.addEventListener("keydown", (e) => {
			if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = true;
		});
		window.addEventListener("keyup", (e) => {
			if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = false;
		});

		const sealInterval = setInterval(() => {
			if (!gameOver) spawnSeal();
		}, 3000);

		updatePosition();

		setTimeout(spawnFish, 3000);
	});

		let rockSetups = Array.from({ length: 6 }, () => ({
		left: Math.random() * window.innerWidth,
		scale: 0.5 + Math.random() * 1.2,
		opacity: 0.5 + Math.random() * 0.5
	}));

		let algaeSetups = Array.from({ length: 15 }, () => ({
		left: Math.random() * window.innerWidth,
		height: 60 + Math.random() * 100,
		scale: 0.5 + Math.random() * 1.5,
		opacity: 0.6 + Math.random() * 0.4
	}));

		let coralSetups = Array.from({ length: 10 }, () => ({
		left: Math.random() * window.innerWidth,
		height: 60 + Math.random() * 100,
		scale: 0.5 + Math.random() * 2,
		opacity: 0.6 + Math.random() * 0.4
		}));

			let playerName = '';
	let leaderboard = [];

	function loadLeaderboard() {
		const data = localStorage.getItem('penguin_leaderboard');
		if (data) {
			leaderboard = JSON.parse(data);
		}
	}

	function saveScore() {
		if (scoreSaved || !playerName.trim()) return;

		leaderboard.push({ name: playerName.trim(), score: caughtCount });
		leaderboard.sort((a, b) => b.score - a.score);
		leaderboard = leaderboard.slice(0, 3); // Top 3

		localStorage.setItem('penguin_leaderboard', JSON.stringify(leaderboard));
		playerName = '';
		scoreSaved = true;
	}

</script>

<style>

:global(html, body) {
	margin: 0;
	padding: 0;
	overflow: hidden;
	width: 100%;
	height: 100%;
}

.hidden {
	opacity: 0;
	pointer-events: none;
	transition: opacity 0.3s ease;
}

.endscreen {
	position: fixed;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	color: white;
	font-size: 2em;
	background: rgba(0, 0, 0, 0.8);
	padding: 40px;
	border-radius: 20px;
	text-align: center;
	z-index: 1100;
}

.endscreen button {
	margin-top: 20px;
	padding: 10px 20px;
	font-size: 1em;
	cursor: pointer;
	border: none;
	border-radius: 10px;
	background: white;
	color: black;
	transition: background 0.3s;
}

.endscreen button:hover {
	background: #ddd;
}

.world {
	width: 100%;
	height: 100%;
	position: fixed;
	top: 0;
	left: 0;
	background: linear-gradient(to bottom, #00334d, #004466 30%, #001e2d 80%, #000 100%);
	overflow: hidden;
}

	.seal {
		width: 450px;
		height: 450px;
		position: absolute;
		background: url("/seal.png") no-repeat center center / contain;
		filter: drop-shadow(0 0 12px rgba(255, 0, 0, 0.3));
		transform-origin: center;
	}

	.penguin {
		width: 100px;
		height: 100px;
		position: absolute;
		background: url("/Pinguim.png") no-repeat center center / contain;
		transition: transform 0.15s linear;
		filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.2));
	}

	.fish {
		width: 70px;
		height: 70px;
		position: absolute;
		background: url("/fish.png") no-repeat center center / contain;
	}

	.message {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		color: white;
		font-size: 5em;
		background: rgba(0, 0, 0, 0.5);
		padding: 10px 20px;
		border-radius: 10px;
		animation: fadeout 1.5s forwards;
		pointer-events: none;
	}

	@keyframes fadeout {
		0% { opacity: 1; }
		100% { opacity: 0; }
	}

	.counter {
		position: absolute;
		top: 10px;
		left: 10px;
		color: white;
		font-size: 1.5em;
		background: rgba(0, 0, 0, 0.5);
		padding: 8px 16px;
		border-radius: 8px;
	}

	.bubble {
		position: absolute;
		bottom: -30px;
		width: 10px;
		height: 10px;
		background: rgba(255, 255, 255, 0.3);
		border-radius: 50%;
		animation: rise 6s infinite ease-in;
		overflow: hidden;
		pointer-events: none;
	}

	@keyframes rise {
		0% {
			transform: translateY(0) scale(1);
			opacity: 0.6;
		}
		100% {
			transform: translateY(-110vh) scale(1.5); /* sobe acima do ecrã */
			opacity: 0;
		}
	}

	.rock {
		position: absolute;
		bottom: -10px;
		width: 80px;
		height: 60px;
		background: radial-gradient(circle at 30% 30%, #555, #222);
		border-radius: 50% 50% 40% 40%;
		box-shadow: inset -5px -5px 10px rgba(0, 0, 0, 0.4);
	}

	.algae {
		position: absolute;
		bottom: -3px;
		width: 32px;
		height: var(--height, 80px);
		background: url("/algas.png") no-repeat center bottom / contain;
		animation: sway 2s infinite alternate ease-in-out;
		transform-origin: bottom center;
		pointer-events: none;
	}	

		.coral {
		position: absolute;
		bottom: -3px;
		width: 32px;
		height: var(--height, 80px);
		background: url("/coral.png") no-repeat center bottom / contain;
		animation: sway 2s infinite alternate ease-in-out;
		transform-origin: bottom center;
		pointer-events: none;
	}	

	.endscreen input {
	margin-top: 10px;
	padding: 8px;
	font-size: 1em;
	border-radius: 6px;
	border: none;
	width: 80%;
	max-width: 200px;
	text-align: center;
}

.endscreen ul {
	list-style: none;
	padding: 0;
	margin-top: 10px;
	color: #ffecb3;
}

.endscreen li {
	margin: 4px 0;
}

	@keyframes sway {
		0% { transform: rotate(3deg); }
		100% { transform: rotate(-3deg); }
	}

	@media screen and (orientation: portrait) {
	.rotate-warning {
		display: flex;
	}
	.world {
		display: none;
	}
}

.rotate-warning {
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background: #001e2d;
	color: white;
	font-size: 2em;
	justify-content: center;
	align-items: center;
	text-align: center;
	padding: 40px;
	z-index: 2000;
	display: none;
}

 .joystick-container {
    position: fixed;
    bottom: 20px;
    left: 20px;
    width: 100px;
    height: 100px;
    background-color: rgba(100, 100, 100, 0.3);
    border-radius: 50%;
    touch-action: none;
    user-select: none;
    overflow: hidden;
  }

  .joystick-thumb {
    position: absolute;
    width: 40px;
    height: 40px;
    background-color: rgba(255, 255, 255, 0.7);
    border-radius: 50%;
    transform: translate(-50%, -50%);
    pointer-events: none;
  }
</style>

<div class="world">

	<div class="rotate-warning">
	Por favor, rode o dispositivo na horizontal para jogar!

	<div
	id="joystick-base"
	on:touchstart|passive={handleTouchStart}
	on:touchmove|passive={handleTouchMove}
	on:touchend={handleTouchEnd}
	style="position: fixed; bottom: 80px; left: 80px; width: 120px; height: 120px; background: rgba(255,255,255,0.1); border-radius: 50%; touch-action: none; z-index: 999;">
	<div

  class="joystick-container"
  on:touchstart={handleTouchStart}
  on:touchmove={handleTouchMove}
  on:touchend={handleTouchEnd}
>
  <div
    class="joystick-thumb"
    style="left: calc(50% + {joystickX}px); top: calc(50% + {joystickY}px);"
  />
</div>
		style="position: absolute; left: 50%; top: 50%; width: 40px; height: 40px; margin-left: -20px; margin-top: -20px; background: white; border-radius: 50%; transform: translate({joystickX}px, {joystickY}px); transition: transform 0.05s;">
	</div>
</div>
</div>



<!-- Ecrã final -->
{#if showEndScreen}
	<div class="endscreen" transition:fade>
		<p>Foste apanhado!</p>
		<p>Peixes: {caughtCount}</p>

		{#if leaderboard.length < 3 || caughtCount > leaderboard[2].score}
			<p>Leaderboard</p>

			{#if !scoreSaved}
				<input
					type="text"
					bind:value={playerName}
					placeholder="O teu nome"
					maxlength="12"
				/>
				<button on:click={saveScore}>Guardar Score</button>
			{/if}
		{/if}

		<h3>🏆 Leaderboard</h3>
		<ul>
			{#each leaderboard as entry, index}
				<li>{index + 1}. {entry.name} — {entry.score} peixe(s)</li>
			{/each}
		</ul>

		<button on:click={() => location.reload()}>Tentar outra vez</button>
	</div>
{/if}

	<!-- Rochas -->
	{#each rockSetups as rock}
		<div
			class="rock"
			style="
				left: {rock.left}px;
				transform: scale({rock.scale});
				opacity: {rock.opacity};
			"
		></div>
	{/each}

	<!-- Algas -->
	{#each algaeSetups as algae}
		<div
			class="algae"
			style="
				left: {algae.left}px;
				--height: {algae.height}px;
				transform: scaleX({algae.scale});
				opacity: {algae.opacity};
			"
		></div>
	{/each}

		<!-- Coral -->
	{#each coralSetups as coral}
		<div
			class="coral"
			style="
				left: {coral.left}px;
				--height: {coral.height}px;
				transform: scaleX({coral.scale});
				opacity: {coral.opacity};
			"
		></div>
	{/each}


	<!-- Bolhas -->
	{#each Array(100) as _, i}
		<div class="bubble" style="left: {Math.random() * 100}vw; animation-delay: {i * 0.1}s;"></div>
	{/each}

	<!-- Foca Monstruosa -->
	{#each seals as seal}
		<div
			class="seal"
			style="
				left: {seal.x}px;
				top: {seal.y}px;
				transform: scaleX({seal.direction});
			"
		></div>
	{/each}

	<!-- Pinguim -->
<div
	class="penguin"
	class:hidden={gameOver}
	style="left: {x}px; top: {y}px; transform: scaleX({facingLeft ? -1 : 1});">
</div>

	<!-- Peixe -->
	{#if showFish}
		<div class="fish" style="left: {fishX}px; top: {fishY}px;"></div>
	{/if}

	<!-- Mensagem -->
	{#if showMessage}
		<div class="message">{message}</div>
	{/if}

	<!-- Contador -->
	<div class="counter">Peixes: {caughtCount}</div>
