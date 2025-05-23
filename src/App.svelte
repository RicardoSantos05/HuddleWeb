<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

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
	let sealInterval;

	let showStartScreen = true;
	let gameStarted = false;

	let joystickX = 0;
	let joystickY = 0;
	let isDragging = false;
	let startX = 0;
	let startY = 0;

	let isPortrait = false;
	let isMobile = false;

	onMount(() => {
		isMobile = /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
		checkOrientation();
		window.addEventListener("resize", checkOrientation);
		window.addEventListener("keydown", (e) => {
			if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = true;
		});
		window.addEventListener("keyup", (e) => {
			if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = false;
		});
		scaleGame();
		window.addEventListener('resize', scaleGame);
	});

	function checkOrientation() {
		isPortrait = window.innerHeight > window.innerWidth;
		scaleGame();

	}

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
		requestFullscreenIfMobile();

		const isLandscape = window.innerWidth > window.innerHeight;
		if (!isLandscape) {
			alert("Roda o telemóvel na horizontal para jogar!");
			return;
		}
		showStartScreen = false;
		gameStarted = true;

		setTimeout(spawnFish, 3001);
		updatePosition();

		sealInterval = setInterval(() => {
			if (!gameOver) spawnSeal();
		}, 3000);
	}
  
	
function requestFullscreenIfMobile() {
	if (isMobile) {
  const docElm = document.documentElement;
  if (docElm.requestFullscreen) {
    docElm.requestFullscreen();
  }
}
}

	function spawnFish() {
		fishX = Math.random() * (1920 - 64);
		fishY = Math.random() * (1080 - 64);
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
			const sealCenterX = seal.x + 300;
			const sealCenterY = seal.y + 160;

			const dx = Math.abs(penguinCenterX - sealCenterX);
			const dy = Math.abs(penguinCenterY - sealCenterY);

			if (dx < 280 && dy < 80) {
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

		const collisionRadius = 150;

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
		}, 300);
	}

	function spawnSeal() {
		const fromLeft = Math.random() < 0.5;
		const y = Math.random() * (1080 - 300);
		const direction = fromLeft ? 1 : -1;
		const x = fromLeft ? -500 : 1920 + 500;

		seals.push({
			x,
			y,
			direction,
			speed: sealSpeed + Math.random() * 1.5
		});
	}

	function updatePosition() {
		if (isDragging) {
			const normalizedX = joystickX / 40;
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

		vx = Math.max(-maxSpeed, Math.min(maxSpeed, vx));
		vy = Math.max(-maxSpeed, Math.min(maxSpeed, vy));

		vx *= drag;
		vy *= drag;

		x += vx;
		y += vy;

		const penguinWidth = 100;
		const penguinHeight = 100;

		x = Math.max(0, Math.min(1920 - penguinWidth, x));
		y = Math.max(0, Math.min(1080 - penguinHeight, y));

		if (vx < -0.5) facingLeft = true;
		else if (vx > 0.5) facingLeft = false;

		if (!gameOver) {
			checkCollision();
			checkSealCollision();
		}

		for (let i = seals.length - 1; i >= 0; i--) {
			seals[i].x += seals[i].speed * seals[i].direction;

			if (
				(seals[i].direction === 1 && seals[i].x > 1920 + 200) ||
				(seals[i].direction === -1 && seals[i].x < -600)
			) {
				seals.splice(i, 1);
			}
		}

		requestAnimationFrame(updatePosition);
	}

	function scaleGame() {
		const game = document.querySelector('.fixed-resolution-wrapper');
		if (!(game instanceof HTMLElement)) return;

		const scaleX = window.innerWidth / 1920;
		const scaleY = window.innerHeight / 1080;
		const scale = Math.min(scaleX, scaleY);

		game.style.transform = `translate(-50%, -50%) scale(${scale})`;
	}

	let rockSetups = Array.from({ length: 6 }, () => ({
		left: Math.random() * 1920,
		scale: 0.5 + Math.random() * 1.2,
		opacity: 0.5 + Math.random() * 0.5
	}));

	let algaeSetups = Array.from({ length: 15 }, () => ({
		left: Math.random() * 1920,
		height: 60 + Math.random() * 100,
		scale: 0.5 + Math.random() * 1.5,
		opacity: 0.6 + Math.random() * 0.4
	}));

	let coralSetups = Array.from({ length: 10 }, () => ({
		left: Math.random() * 1920,
		height: 60 + Math.random() * 100,
		scale: 0.5 + Math.random() * 2,
		opacity: 0.6 + Math.random() * 0.4
	}));

	let bubbleSetups = Array.from({ length: 20 }, () => ({
		left: Math.random() * 1920,
		delay: Math.random() * 5,
		scale: 0.1 + Math.random() * 0.7,
		opacity: 0.3 + Math.random() * 0.7
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
		leaderboard = leaderboard.slice(0, 3);

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
	background-color: black;

}

.fixed-resolution-wrapper {
  position: fixed;
  top: 50%;
  left: 50%;
  width: 1920px;
  height: 1080px;
  transform-origin: center center;
  pointer-events: none;
  z-index: 1000;
}

.scaled-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  pointer-events: auto;
  /* será escalado com JS */
}

.hidden {
	opacity: 0;
	pointer-events: none;
	transition: opacity 0.3s ease;
}

.portrait-warning {
position: absolute;
	top: 0;
	left: 0;
	width: 100vw;
	height: 100vh;
	background: black;
	color: white;
	display: flex;
	justify-content: center;
	align-items: center;
	font-size: 0.7rem;
	text-align: center;
	z-index: 9999;
}

.start-title {
	position: absolute;
	top: 100px;
	left: 50%;
	transform: translateX(-50%);
	font-size: 180px;
	font-family: 'Arial Black', sans-serif;
	color: white;
	text-shadow: 40px 40px 25px rgba(0, 0, 0, 0.7);
	z-index: 1001;
	pointer-events: none;
}

.catch-message {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	font-size: 200px;
	color: white;
	text-shadow: 4px 4px 10px rgba(0, 0, 0, 0.7);
	font-family: 'Comic Sans MS', cursive, sans-serif;
	z-index: 2000;
	animation: pop 1.5s ease;
	pointer-events: none;
}

@keyframes pop {
	0% {
		transform: translate(-50%, -50%) scale(0.6);
		opacity: 0;
	}
	10% {
		transform: translate(-50%, -50%) scale(1.1);
		opacity: 1;
	}
	100% {
		transform: translate(-50%, -50%) scale(1);
		opacity: 0;
	}
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
	background: #222222;
}

.world {
	width: 100%;
	height: 100%;
	position: absolute;
	top: 0;
	left: 0;
	background: linear-gradient(to bottom, #00334d, #004466 30%, #001e2d 80%, #000 100%);
	overflow: hidden;
}


/* seal: 450px / 1280 = 35.16% width, altura igual à largura para manter proporção */
.seal {
	width: 35.16%;
	height: 35.16%;
	position: absolute;
	background: url("/seal.png") no-repeat center center / contain;
	filter: drop-shadow(0 0 12px rgba(255, 0, 0, 0.3));
	transform-origin: center;
}

/* penguin: 100px / 1280 = 7.81% width e height */
.penguin {
	width: 7.81%;
	height: 7.81%;
	position: absolute;
	background: url("/Pinguim.png") no-repeat center center / contain;
	transition: transform 0.15s linear;
	filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.2));
}

/* fish: 70px / 1280 = 5.47% width e height */
.fish {
	width: 9%;
	height: 9%;
	position: absolute;
	background: url("/fish.png") no-repeat center center / contain;
}

@keyframes fadeout {
	0% { opacity: 1; }
	100% { opacity: 0; }
}

.counter {
	position: absolute;
	top: 20px;
	left: 20px;
	color: white;
	font-size: 3em;
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
	animation: rise 7s infinite ease-in;
	overflow: hidden;
	pointer-events: none;
}

@keyframes rise {
	0% {
		transform: translateY(0) scale(1);
		opacity: 0.6;
	}
	100% {
		transform: translateY(-250vh) scale(1.5);
		opacity: 0;
	}
}

.rock {
	position: absolute;
	bottom: -10px;
	/* 80px / 1280 = 6.25% width, 60px / 720 = 8.33% height */
	width: 6.25%;
	height: 8.33%;
	background: radial-gradient(circle at 30% 30%, #555, #222);
	border-radius: 50% 50% 40% 40%;
	box-shadow: inset -5px -5px 10px rgba(0, 0, 0, 0.4);
}

.algae {
	position: absolute;
	bottom: -3px;
	/* 32px / 1280 = 2.5% width, height relativa via variável CSS já definida */
	width: 2.5%;
	height: var(--height, 11.11%); /* 80px / 720 = 11.11% */
	background: url("/algas.png") no-repeat center bottom / contain;
	animation: sway 2s infinite alternate ease-in-out;
	transform-origin: bottom center;
	pointer-events: none;
}

.coral {
	position: absolute;
	bottom: -3px;
	width: 2.5%;
	height: var(--height, 11.11%);
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

.start-screen {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	text-align: center;
	background: rgba(0, 0, 0, 0.7);
	color: white;
	padding: 100%;
	border-radius: 20px;
	z-index: 10;
}

.start-screen button {
	padding: 20px 40px;
	font-size: 3em;
	cursor: pointer;
	background-color: #1d1d1d;
	color: #dcdcdc;
	border: none;
	border-radius: 12px;
	box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
	transition: transform 0.2s, background 0.3s;
}

.start-screen button:hover {
	transform: scale(1.05);
	background-color: #252525;
}

.joystick-container {
	position: fixed;
	bottom: 100px;
	left: 100px;
	width: 7.81%; /* 100px / 1280 */
	height: 13.89%; /* 100px / 720 */
	background-color: rgba(100, 100, 100, 0.3);
	border-radius: 50%;
	touch-action: none;
	user-select: none;
	overflow: hidden;
}

.joystick-thumb {
	position: absolute;
	width: 3.13%; /* 40px / 1280 */
	height: 5.56%; /* 40px / 720 */
	background-color: rgba(255, 255, 255, 0.7);
	border-radius: 50%;
	transform: translate(-50%, -50%);
	pointer-events: none;
}
</style>

    <div>
      <!-- Aviso para modo retrato -->
      {#if isPortrait}
        <div class="portrait-warning">
          Por favor, roda o dispositivo na horizontal para jogar.
        </div>
      {/if}
  	</div>

  <div class="fixed-resolution-wrapper">
  	<div class="scaled-wrapper">
    <div class="world">

      <!-- Joystick -->
      {#if isMobile && !isPortrait && !showStartScreen}
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
      {/if}

      <!-- Ecrã Inicial -->
      {#if showStartScreen}
		<div class="start-title">HUDDLE</div>
        <div class="start-screen">
          {#if !isPortrait}
            <button on:click={startGame}>START</button>
          {/if}
        </div>
      {/if}

		{#if showMessage}
		<div class="catch-message">
	{message}
</div>
	{/if}

      <!-- Ecrã final -->
      {#if showEndScreen}
        <div class="endscreen" transition:fade>
          <p>Foste apanhado!</p>
          <p>Peixes: {caughtCount}</p>

          {#if leaderboard.length < 3 || caughtCount > leaderboard[2].score}

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
	  
		<!-- Bublles-->
		{#each bubbleSetups as bubble}
	<img
		src="bubble.png"
		alt="Bubble"
		class="bubble"
		style="
			left: {bubble.left}px;
			animation-delay: {bubble.delay}s;
			transform: scale({bubble.scale});
			opacity: {bubble.opacity};
		"
		/>
	{/each}

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


      <!-- Contador -->
      <div class="counter">Peixes: {caughtCount}</div>

	</div>
	</div>
</div>

