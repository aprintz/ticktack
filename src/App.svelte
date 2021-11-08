<script>
  import { fade } from "svelte/transition";
  import { cubicInOut } from "svelte/easing";
  import fadeScale from "./fadeScale.js";
  export let name;
  let running = false;
  let w;
  let showingTick = false;
  let showingBoom = false;
  let bomb;
  let tickStyle = "top:50%;left:50%;";
  let timerInstance = undefined;
  let rotation = 0;
  let translation = 0;
  let canvas = undefined;
  let tick = new Audio("/sounds/tick.mp3");
  let boom = new Audio("/sounds/boom1.mp3");
  $: buttonText = running ? "Stopp" : "Start";
  let tickText;
  const colors = ["#ffc000", "#ff3b3b", "#ff8400"];
  const bubbles = 25;
  function startExplosions(x, y) {
    let ratio = window.devicePixelRatio;
    canvas = document.createElement("canvas");
    canvas.style.position = "absolute";
    canvas.style.left = x - 100 + "px";
    canvas.style.top = y - 100 + "px";
    canvas.style.pointerEvents = "none";
    canvas.style.width = 200 + "px";
    canvas.style.height = 200 + "px";
    canvas.style.zIndex = 100;
    canvas.width = 200 * ratio;
    canvas.height = 200 * ratio;
    document.body.appendChild(canvas);
    explode(x, y, canvas);
  }
  const explode = (x, y) => {
    let particles = [];
    let ctx = canvas.getContext("2d");

    for (var i = 0; i < bubbles; i++) {
      particles.push({
        x: canvas.width / 2,
        y: canvas.height / 2,
        radius: r(20, 30),
        color: colors[Math.floor(Math.random() * colors.length)],
        rotation: r(0, 360, true),
        speed: r(8, 12),
        friction: 0.9,
        opacity: r(0, 0.5, true),
        yVel: 0,
        gravity: 0.1,
      });
    }

    render(particles, ctx, canvas.width, canvas.height);
    setTimeout(() => {
      if (running) {
        explode(x, y);
      }
    }, 150);
  };

  const render = (particles, ctx, width, height) => {
    requestAnimationFrame(() => render(particles, ctx, width, height));
    ctx.clearRect(0, 0, width, height);

    particles.forEach((p, i) => {
      p.x += p.speed * Math.cos((p.rotation * Math.PI) / 180);
      p.y += p.speed * Math.sin((p.rotation * Math.PI) / 180);

      p.opacity -= 0.01;
      p.speed *= p.friction;
      p.radius *= p.friction;
      p.yVel += p.gravity;
      p.y += p.yVel;

      if (p.opacity < 0 || p.radius < 0) return;

      ctx.beginPath();
      ctx.globalAlpha = p.opacity;
      ctx.fillStyle = p.color;
      ctx.arc(p.x, p.y, p.radius, 0, 2 * Math.PI, false);
      ctx.fill();
    });

    return ctx;
  };

  const r = (a, b, c) =>
    parseFloat(
      (Math.random() * ((a ? a : 1) - (b ? b : 0)) + (b ? b : 0)).toFixed(
        c ? c : 0
      )
    );

  function resetState() {
    clearInterval(timerInstance);
    timerInstance = undefined;
    running = false;
    showingTick = false;
  }
  function playBoom() {
    boom.play();
    showingBoom = true;
    rotation = 0;
    translation = 50;
    resetState();
  }
  function playTick(time) {
    tickText = time % 2 === 0 ? "tick" : "tack";
    translation = 0;
    const top = 10 + Math.floor(Math.random() * 60);
    const left = 10 + Math.floor(Math.random() * 60);
    rotation = Math.floor(Math.random() * 45) * (Math.random() < 0.5 ? -1 : 1);
    tickStyle = `top:${top}%;left:${left}%;transform: rotate(${rotation}deg);`;
    showingTick = true;
    setTimeout(function () {
      showingTick = false;
    }, 500);

    tick.play();
  }
  function getRandomTime() {
    return Math.floor(Math.random() * (6 - 1) + 1) * 10;
  }
  function stopTimer() {
    if (timerInstance) {
      resetState();
    }
  }
  function getOffset(el) {
    return {
      left: el.offsetLeft + el.clientWidth - 50,
      top: el.offsetTop + 50,
    };
  }
  function startTimer(duration) {
    var start = Date.now();
    function timer() {
      var diff = duration - (((Date.now() - start) / 1000) | 0);
      console.log(diff);
      if (diff <= 0) {
        playBoom();
      } else {
        playTick(diff);
      }
    }
    timer();
    timerInstance = setInterval(timer, 1000);
  }

  function changeRunning() {
    running = !running;
    if (!running) {
      stopTimer();
      if (canvas) {
        document.body.removeChild(canvas);
        canvas = undefined;
      }
    } else {
      startTimer(getRandomTime());
      const pos = getOffset(bomb);
      startExplosions(pos.left, pos.top);
    }
  }
</script>

<main>
  <h1 class="visually-hidden">Tick tack boom!</h1>
  <div class="col">
    <div />
    <img bind:this={bomb} class="bomb" alt="da bomb" src="/images/bomb.png" />
    <button on:click={changeRunning}> <span>{buttonText}</span></button>
  </div>
  {#if showingTick}
    <span
      bind:clientWidth={w}
      class="tick"
      style={tickStyle}
      transition:fadeScale={{
        delay: 0,
        rotation: rotation,
        translation: translation,
        duration: 300,
        easing: cubicInOut,
        baseScale: 0,
      }}>{tickText}</span
    >
  {/if}
  {#if showingBoom}
    <img
      alt="boom!"
      src="/images/boom.png"
      style=" position: absolute;top:50%;left:50%;width:90%;height:auto;transform:translate(-50%, -50%) rotate(0deg);"
      transition:fadeScale={{
        delay: 0,
        rotation: rotation,
        translation: translation,
        duration: 300,
        easing: cubicInOut,
        baseScale: 0,
      }}
    />
  {/if}
</main>

<style>
  .visually-hidden {
    clip: rect(0 0 0 0);
    clip-path: inset(50%);
    height: 1px;
    overflow: hidden;
    position: absolute;
    white-space: nowrap;
    width: 1px;
  }
  .tick {
    position: absolute;
    padding: 10px 40px;
    color: #e3d310;
    font-size: 3em;
    font-weight: 700;
    background-image: url(/images/backbubble.png);
    background-repeat: no-repeat;
    background-size: contain;
  }
  main {
    text-align: center;
    max-width: 540px;
    margin: 0 auto;
    height: 100%;
  }
  .square {
    position: relative;
    width: 90%;
  }
  .col {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    height: 100%;
  }
  .bomb {
    margin-left: 20%;
    max-width: 80%;
  }
  button {
    transition: transform 0.2s ease;
    cursor: pointer;
    padding: 10px 20px;
    font-weight: 600;
    font-size: 2em;
    text-transform: uppercase;
    border-style: none;
    background-color: #1d181c;
    color: #e3d310;
    background-image: radial-gradient(#1d181c 20%, transparent 20%);
    background-color: #0f357d;
    background-position: 0 0, 20px 20px;
    background-size: 20px 20px;
    border: 7px solid #1d181c;
    border-top-left-radius: 20px 40px;
    border-bottom-right-radius: 20px 40px;
    border-top-right-radius: 40px 20px;
    border-bottom-left-radius: 40px 20px;
  }
  button:hover {
    transform: scale(0.9);
    outline: none;
    background-color: none;
  }
  img {
    max-width: 100%;
  }
  .row {
    display: flex;
    width: 100%;
    flex-direction: row;
    justify-content: space-between;
    align-self: stretch;
    align-items: center;
  }
</style>
