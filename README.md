<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>I Love You komeil122</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #000;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    overflow: hidden;
    font-family: 'Georgia', serif;
  }

  #canvas-container {
    position: relative;
    width: 500px;
    height: 500px;
  }

  canvas {
    position: absolute;
    top: 0; left: 0;
  }
</style>
</head>
<body>
<div id="canvas-container">
  <canvas id="heartCanvas" width="500" height="500"></canvas>
</div>

<script>
const canvas = document.getElementById('heartCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const cx = W / 2;
const cy = H / 2 + 20;

// Heart parametric: x = 16sin³t, y = -(13cost - 5cos2t - 2cos3t - cos4t)
function heartPoint(t) {
  const scale = 11.5;
  const x = cx + scale * 16 * Math.pow(Math.sin(t), 3);
  const y = cy - scale * (13 * Math.cos(t) - 5 * Math.cos(2*t) - 2 * Math.cos(3*t) - Math.cos(4*t));
  return { x, y };
}

// Build heart path points
const HEART_STEPS = 300;
const heartPoints = [];
for (let i = 0; i <= HEART_STEPS; i++) {
  const t = (i / HEART_STEPS) * Math.PI * 2;
  heartPoints.push(heartPoint(t));
}

// Compute total arc length
const arcLengths = [0];
for (let i = 1; i < heartPoints.length; i++) {
  const dx = heartPoints[i].x - heartPoints[i-1].x;
  const dy = heartPoints[i].y - heartPoints[i-1].y;
  arcLengths.push(arcLengths[i-1] + Math.sqrt(dx*dx + dy*dy));
}
const totalLength = arcLengths[arcLengths.length - 1];

// Get point at arc distance
function getPointAtLength(dist) {
  dist = ((dist % totalLength) + totalLength) % totalLength;
  let lo = 0, hi = arcLengths.length - 1;
  while (lo < hi - 1) {
    const mid = (lo + hi) >> 1;
    if (arcLengths[mid] < dist) lo = mid; else hi = mid;
  }
  const t = (dist - arcLengths[lo]) / (arcLengths[hi] - arcLengths[lo]);
  return {
    x: heartPoints[lo].x + t * (heartPoints[hi].x - heartPoints[lo].x),
    y: heartPoints[lo].y + t * (heartPoints[hi].y - heartPoints[lo].y),
    angle: Math.atan2(
      heartPoints[hi].y - heartPoints[lo].y,
      heartPoints[hi].x - heartPoints[lo].x
    )
  };
}

// Layers config: offset from center path (negative = inside, positive = outside)
const layers = [
  { offset: -28, alpha: 0.35, size: 10 },
  { offset: -14, alpha: 0.55, size: 11.5 },
  { offset:   0, alpha: 0.85, size: 13 },
  { offset:  14, alpha: 0.55, size: 11.5 },
  { offset:  28, alpha: 0.35, size: 10 },
];

const text = "I love you komeil122 ";
let animOffset = 0;

function drawLayer(layer, timeOffset) {
  const charSpacing = layer.size * 5.2;
  const numChars = Math.ceil(totalLength / charSpacing) + 2;

  for (let i = 0; i < numChars; i++) {
    const dist = (i * charSpacing + animOffset * (totalLength / 600) + timeOffset) % totalLength;
    const base = getPointAtLength(dist);

    // Normal direction (perpendicular to tangent)
    const nx = -Math.sin(base.angle);
    const ny =  Math.cos(base.angle);

    const px = base.x + nx * layer.offset;
    const py = base.y + ny * layer.offset;

    const ch = text[i % text.length];

    ctx.save();
    ctx.translate(px, py);
    ctx.rotate(base.angle + Math.PI / 2);

    ctx.font = layer.size + 'px Georgia, serif';
    ctx.fillStyle = 'rgba(234, 128, 176, ' + layer.alpha + ')';
    ctx.shadowColor = '#fff';
    ctx.shadowBlur = 8;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(ch, 0, 0);
    ctx.restore();
  }
}

function animate() {
  ctx.clearRect(0, 0, W, H);

  layers.forEach((layer, i) => {
    const timeOffset = i * (totalLength / layers.length) * 0.18;
    drawLayer(layer, timeOffset);
  });

  animOffset++;
  requestAnimationFrame(animate);
}

animate();
</script>
</body>
</html>
