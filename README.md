/* ============================================================
   P5 ambient — kernel field visualization
   ============================================================ */
let t = 0;
let pts = [];

function setup() {
  const c = createCanvas(window.innerWidth, window.innerHeight);
  c.parent('stage');
  pixelDensity(Math.min(2, window.devicePixelRatio || 1));
  colorMode(RGB, 255, 255, 255, 1);

  const N = 220;
  for (let i = 0; i < N; i++) {
    const a = random(TWO_PI);
    const r = random(0.25, 1.0);
    pts.push({
      a,
      r,
      sp: random(0.0006, 0.0018),
      ph: random(TWO_PI),
      sz: random(1.0, 2.4),
      p: random([0.6, 1.0, 1.75, 2.0, 3.0])
    });
  }
  noStroke();
}

function pnorm(a, r, p) {
  const ca = Math.cos(a), sa = Math.sin(a);
  const x = Math.sign(ca) * Math.pow(Math.abs(ca), 2 / p) * r;
  const y = Math.sign(sa) * Math.pow(Math.abs(sa), 2 / p) * r;
  return [x, y];
}

function draw() {
  background(2, 6, 23, 0.18);
  const cx = width * 0.5;
  const cy = height * 0.55;
  const R = Math.min(width, height) * 0.42;

  t += 0.0028;

  const g = drawingContext.createRadialGradient(cx, cy, 0, cx, cy, R * 1.6);
  g.addColorStop(0, 'rgba(34,211,238,0.06)');
  g.addColorStop(0.5, 'rgba(192,132,252,0.02)');
  g.addColorStop(1, 'rgba(2,6,23,0)');
  drawingContext.fillStyle = g;
  drawingContext.fillRect(0, 0, width, height);

  for (const pt of pts) {
    pt.a += pt.sp;
    const drift = Math.sin(t * 2 + pt.ph) * 0.06;
    const [x, y] = pnorm(pt.a + drift, pt.r * R, pt.p);

    const depth = 0.4 + 0.6 * (1 - pt.r);
    const alpha = 0.15 + 0.55 * depth * (0.6 + 0.4 * Math.sin(t * 1.4 + pt.ph));

    fill(34, 211, 238, alpha * 0.9);
    circle(cx + x, cy + y, pt.sz);

    if (pt.sz > 2.0) {
      noFill();
      stroke(34, 211, 238, alpha * 0.12);
      strokeWeight(0.6);
      circle(cx + x, cy + y, pt.sz * 6);
      noStroke();
    }
  }

  const coreR = 6 + Math.sin(t * 3) * 2;
  fill(251, 191, 36, 0.35);
  circle(cx, cy, coreR * 4);
  fill(251, 191, 36, 0.85);
  circle(cx, cy, coreR);

  const shellR = ((t * 0.35) % 1) * R * 1.15;
  if (shellR > 0) {
    noFill();
    stroke(192, 132, 252, 0.10 * (1 - shellR / (R * 1.15)));
    strokeWeight(0.8);
    circle(cx, cy, shellR * 2);
    noStroke();
  }
}

function windowResized() {
  resizeCanvas(window.innerWidth, window.innerHeight);
}

const prog = document.getElementById('prog');
const nav = document.getElementById('nav');

function onScroll() {
  const h = document.documentElement;
  const max = h.scrollHeight - h.clientHeight;
  const pct = max > 0 ? (h.scrollTop / max) * 100 : 0;
  prog.style.width = pct + '%';
  nav.classList.toggle('scrolled', h.scrollTop > 40);
}
window.addEventListener('scroll', onScroll, { passive: true });
onScroll();

const io = new IntersectionObserver((entries) => {
  for (const e of entries) {
    if (e.isIntersecting) {
      e.target.classList.add('in');
      io.unobserve(e.target);
    }
  }
}, { rootMargin: '0px 0px -8% 0px', threshold: 0.08 });

document.querySelectorAll('.reveal').forEach((el) => io.observe(el));
