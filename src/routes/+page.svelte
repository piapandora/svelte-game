<script>
  import { onMount } from "svelte";

  // Player state
  let x = $state(100);
  let y = $state(100);
  const playerSize = 50;

  // Collectible state
  let coinX = $state(400);
  let coinY = $state(300);
  const coinSize = 25;

  let score = $state(0);
  let keys = { w: false, a: false, s: false, d: false, shift: false };

  // Updated speed settings
  const baseSpeed = 8;
  const dashMultiplier = 1.5;

  function toggleFullscreen() {
    if (!document.fullscreenElement) {
      document.documentElement.requestFullscreen().catch((err) => {
        console.error(`Error attempting to enable fullscreen: ${err.message}`);
      });
    } else {
      if (document.exitFullscreen) {
        document.exitFullscreen();
      }
    }
  }

  function date() {
    if (typeof window !== "undefined") {
      window.scrollTo(0, 0);
    }

    const gamepads = navigator.getGamepads();
    const gp = gamepads[0];

    // 1. Dash Logic (X button) & Fullscreen Toggle (Triangle button - index 3)
    const isDashing = keys.shift || (gp && gp.buttons[0].pressed);
    const speed = isDashing ? baseSpeed * dashMultiplier : baseSpeed;

    // Gamepad Triangle button toggle with debounce
    if (gp && gp.buttons[3].pressed) {
      if (!window.isToggling) {
        toggleFullscreen();
        window.isToggling = true;
        setTimeout(() => (window.isToggling = false), 500);
      }
    }

    // 2. Movement Input
    let dx = 0;
    let dy = 0;
    if (keys.w) dy -= 1;
    if (keys.s) dy += 1;
    if (keys.a) dx -= 1;
    if (keys.d) dx += 1;

    if (gp) {
      if (Math.abs(gp.axes[0]) > 0.1) dx = gp.axes[0];
      if (Math.abs(gp.axes[1]) > 0.1) dy = gp.axes[1];
    }

    const length = Math.hypot(dx, dy);
    if (length > 1) {
      dx /= length;
      dy /= length;
    }

    // 3. Move & Clamp
    x = Math.max(0, Math.min(window.innerWidth - playerSize, x + dx * speed));
    y = Math.max(0, Math.min(window.innerHeight - playerSize, y + dy * speed));

    // 4. Collision
    const playerCenterX = x + playerSize / 2;
    const playerCenterY = y + playerSize / 2;
    const coinCenterX = coinX + coinSize / 2;
    const coinCenterY = coinY + coinSize / 2;
    const distance = Math.hypot(
      playerCenterX - coinCenterX,
      playerCenterY - coinCenterY,
    );

    if (distance < playerSize / 2 + coinSize / 2) {
      score++;
      coinX = Math.random() * (window.innerWidth - coinSize);
      coinY = Math.random() * (window.innerHeight - coinSize);
    }

    requestAnimationFrame(update);
  }

  onMount(() => {
    requestAnimationFrame(update);
  });
</script>

<svelte:window
  onkeydown={(e) => {
    if (e.key === "Shift") keys.shift = true;
    else keys[e.key.toLowerCase()] = true;
  }}
  onkeyup={(e) => {
    if (e.key === "Shift") keys.shift = false;
    else keys[e.key.toLowerCase()] = false;
  }}
/>

<div class="game-container" ondblclick={toggleFullscreen} role="application">
  <div class="score">Scorefff: {score}</div>
  <div class="player" style:transform="translate({x}px, {y}px)"></div>
  <div class="coin" style:transform="translate({coinX}px, {coinY}px)"></div>
</div>

<style>
  :global(html, body, *) {
    /* background: #0f172a; */
    background: #300000;
    margin: 0;
    padding: 0;
    overflow: hidden;
    font-family: "Courier New", Courier, monospace;
    user-select: none;
    touch-action: none;
    cursor: none !important;
  }

  .game-container {
    width: 100vw;
    height: 100vh;
    position: relative;
  }

  .player {
    width: 50px;
    height: 50px;
    background: #38bdf8;
    position: absolute;
    border-radius: 8px;
    box-shadow: 0 0 25px rgba(56, 189, 248, 0.6);
    will-change: transform;
    z-index: 5;
  }

  .coin {
    width: 25px;
    height: 25px;
    background: #fbbf24;
    position: absolute;
    border-radius: 50%;
    box-shadow: 0 0 20px rgba(251, 191, 36, 0.8);
    will-change: transform;
    z-index: 4;
  }

  .score {
    position: absolute;
    top: 20px;
    left: 20px;
    color: #f8fafc;
    font-size: 2rem;
    font-weight: bold;
    z-index: 10;
    pointer-events: none;
    opacity: 0.8;
    background: transparent;
  }
</style>
