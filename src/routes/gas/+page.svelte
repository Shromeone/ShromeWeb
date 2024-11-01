<script>
  class Particle {
    constructor(x, y, dirInRadians) {
      this.x = x;
      this.y = y;
      this.dirInRadians = dirInRadians;
    }
  }
  let alreadySetup = false;

  /** @type {HTMLCanvasElement} */
  let particle = new Particle(100, 100, 5);
  /** @type {Array<Particle>} */
  let particles = [];
  let canvas = $state();
  setInterval(drawCanvas, 10);
  const particleSpeed = 6;
  const particleRadius = 5;
  /** @type {CanvasRenderingContext2D} */
  let ctx;
  function setup() {
    if (!canvas) return;

    for (let i = 0; i < 20; i++) {
      let randomDirection = Math.random() * 2 * Math.PI;
      let randomX = Math.random() * canvas.width;
      let randomY = Math.random() * canvas.height;
      particles.push(new Particle(randomX, randomY, randomDirection));
    }
    alreadySetup = true;
    particle.x = canvas.width / 2;
    particle.y = 50;
    ctx = canvas.getContext("2d");
  }

  function drawBall() {
    for (let p of particles) {
      ctx.beginPath();
      ctx.arc(p.x, p.y, particleRadius, 0, Math.PI * 2);
      ctx.fillStyle = "yellow";
      ctx.fill();
      ctx.closePath();
    }
    // ctx.rect(20, 40, 50, 50);
  }

  function updateParticlePosition(p) {
    const dx = Math.cos(p.dirInRadians) * particleSpeed;
    const dy = Math.sin(p.dirInRadians) * particleSpeed;

    if (
      p.y + dy + particleRadius > canvas.height ||
      p.y + dy + particleRadius < 0
    ) {
      p.dirInRadians *= -1;
    }

    if (
      p.x + dx + particleRadius > canvas.width ||
      p.x + dx + particleRadius < 0
    ) {
      p.dirInRadians = Math.PI - p.dirInRadians;
    }
    p.x += dx;
    p.y += dy;
  }
  function drawCanvas() {
    if (!alreadySetup) {
      setup();
      return;
    }
    if (!canvas) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBall();
    for (let p of particles) {
      updateParticlePosition(p);
    }
  }
</script>

<canvas bind:this={canvas} width="420" , height="420"></canvas>

<button onclick={drawCanvas}>Draw Canvas</button>

<style>
  canvas {
    background: var(--dark);
  }
</style>
