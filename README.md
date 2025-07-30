# glow
balls glowing randomly 
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>Multi-Color Bouncing Glows</title>
    <style>
        body {
            background: #000;
            margin: 0;
            overflow: hidden;
            height: 100vh;
        }
        .glow {
            position: fixed;
            width: 52px;
            height: 52px;
            border-radius: 50%;
            pointer-events: none;
            z-index: 10;
            transition: background 1s linear;
        }
    </style>
</head>
<body>
    <script>
        const colors = [
            'radial-gradient(circle, #39ff14 40%, rgba(57,255,20,0.5) 70%, transparent 100%)', // green
            'radial-gradient(circle, #ff1493 40%, rgba(255,20,147,0.5) 70%, transparent 100%)', // pink
            'radial-gradient(circle, #14d3ff 40%, rgba(20,211,255,0.5) 70%, transparent 100%)', // blue
            'radial-gradient(circle, #ffd700 40%, rgba(255,215,0,0.5) 70%, transparent 100%)', // gold
            'radial-gradient(circle, #ff4500 40%, rgba(255,69,0,0.5) 70%, transparent 100%)', // orange
            'radial-gradient(circle, #9400d3 40%, rgba(148,0,211,0.5) 70%, transparent 100%)', // violet
            'radial-gradient(circle, #00ffea 40%, rgba(0,255,234,0.5) 70%, transparent 100%)', // cyan
            'radial-gradient(circle, #fffb00 40%, rgba(255,251,0,0.5) 70%, transparent 100%)',  // yellow
            'radial-gradient(circle, #ff6347 40%, rgba(255,99,71,0.5) 70%, transparent 100%)',  // tomato
            'radial-gradient(circle, #00fa9a 40%, rgba(0,250,154,0.5) 70%, transparent 100%)',  // medium spring green
            'radial-gradient(circle, #1e90ff 40%, rgba(30,144,255,0.5) 70%, transparent 100%)', // dodger blue
            'radial-gradient(circle, #ff00ff 40%, rgba(255,0,255,0.5) 70%, transparent 100%)',  // magenta
            'radial-gradient(circle, #ffa500 40%, rgba(255,165,0,0.5) 70%, transparent 100%)',  // orange
            'radial-gradient(circle, #00ff7f 40%, rgba(0,255,127,0.5) 70%, transparent 100%)',  // spring green
            'radial-gradient(circle, #8a2be2 40%, rgba(138,43,226,0.5) 70%, transparent 100%)', // blue violet
            'radial-gradient(circle, #dc143c 40%, rgba(220,20,60,0.5) 70%, transparent 100%)'   // crimson
        ];
        const size = 52;
        const glows = [];
        const balls = [];

        function randomVelocity() {
            let angle = Math.random() * 2 * Math.PI;
            let speed = 2 + Math.random() * 2;
            return { vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed };
        }

for (let i = 0; i < 16; i++) {
    const glow = document.createElement('div');
    glow.className = 'glow';
    glow.style.background = colors[i % colors.length];
    document.body.appendChild(glow);

    // Start at random edge
    let edge = Math.floor(Math.random() * 4);
    let pos;
    if (edge === 0) pos = { x: Math.random() * (window.innerWidth - size), y: 0 };
    else if (edge === 1) pos = { x: Math.random() * (window.innerWidth - size), y: window.innerHeight - size };
    else if (edge === 2) pos = { x: 0, y: Math.random() * (window.innerHeight - size) };
    else pos = { x: window.innerWidth - size, y: Math.random() * (window.innerHeight - size) };

    let vel = randomVelocity();

    balls.push({ x: pos.x, y: pos.y, vx: vel.vx, vy: vel.vy, el: glow, colorIndex: i % colors.length, colorStep: Math.random() * 100, active: true });
    glows.push(glow);
}

        function updatePositions() {
            for (let i = 0; i < balls.length; i++) {
                let b = balls[i];
                if (b.active) {
                    b.x += b.vx;
                    b.y += b.vy;
                }

                // Bounce off walls
                if (b.x <= 0) { b.x = 0; b.vx *= -1; }
                if (b.x >= window.innerWidth - size) { b.x = window.innerWidth - size; b.vx *= -1; }
                if (b.y <= 0) { b.y = 0; b.vy *= -1; }

                // If ball touches bottom, deactivate
                if (b.y >= window.innerHeight - size) {
                    b.y = window.innerHeight - size;
                    b.vx = 0;
                    b.vy = 0;
                    b.active = false;
                }

                // Bounce off other balls
                for (let j = i + 1; j < balls.length; j++) {
                    let b2 = balls[j];
                    let dx = b2.x - b.x;
                    let dy = b2.y - b.y;
                    let dist = Math.sqrt(dx * dx + dy * dy);
                    if (dist < size) {
                        // Simple elastic collision: swap velocities
                        let tempVx = b.vx;
                        let tempVy = b.vy;
                        b.vx = b2.vx;
                        b.vy = b2.vy;
                        b2.vx = tempVx;
                        b2.vy = tempVy;

                        // Exchange color state
                        let tempColorStep = b.colorStep;
                        b.colorStep = b2.colorStep;
                        b2.colorStep = tempColorStep;

                        // If either ball is inactive, reactivate both
                        if (!b.active || !b2.active) {
                            b.active = true;
                            b2.active = true;
                            // Give a random velocity to both
                            let v1 = randomVelocity();
                            let v2 = randomVelocity();
                            b.vx = v1.vx;
                            b.vy = v1.vy;
                            b2.vx = v2.vx;
                            b2.vy = v2.vy;
                        }

                        // Move balls apart
                        let overlap = size - dist;
                        let nx = dx / dist;
                        let ny = dy / dist;
                        b.x -= nx * overlap / 2;
                        b.y -= ny * overlap / 2;
                        b2.x += nx * overlap / 2;
                        b2.y += ny * overlap / 2;
                    }
                }

                // Color change logic
                b.colorStep += 0.5;
                let idx1 = Math.floor(b.colorStep / 50) % colors.length;
                let idx2 = (idx1 + 1) % colors.length;
                let frac = (b.colorStep % 50) / 50;
                // Interpolate between two gradients by blending backgrounds
                b.el.style.background = `radial-gradient(circle at 50% 50%,
                    ${getGradientColor(idx1, frac, idx2)} 40%,
                    ${getGradientColor(idx1, frac, idx2, 0.5)} 70%,
                    transparent 100%)`;

                b.el.style.left = b.x + 'px';
                b.el.style.top = b.y + 'px';
            }
            requestAnimationFrame(updatePositions);
        }

        // Helper to interpolate between two colors in gradients
        function getGradientColor(idx1, frac, idx2, alpha=1) {
            // Extract RGB from color string
            const colorStops = [
                [57,255,20],    // green
                [255,20,147],   // pink
                [20,211,255],   // blue
                [255,215,0],    // gold
                [255,69,0],     // orange
                [148,0,211],    // violet
                [0,255,234],    // cyan
                [255,251,0],    // yellow
                [255,99,71],    // tomato
                [0,250,154],    // medium spring green
                [30,144,255],   // dodger blue
                [255,0,255],    // magenta
                [255,165,0],    // orange
                [0,255,127],    // spring green
                [138,43,226],   // blue violet
                [220,20,60]     // crimson
            ];
            let c1 = colorStops[idx1];
            let c2 = colorStops[idx2];
            let r = Math.round(c1[0] + (c2[0] - c1[0]) * frac);
            let g = Math.round(c1[1] + (c2[1] - c1[1]) * frac);
            let b = Math.round(c1[2] + (c2[2] - c1[2]) * frac);
            return `rgba(${r},${g},${b},${alpha})`;
        }

        updatePositions();

        window.addEventListener('resize', () => {
            for (let b of balls) {
                b.x = Math.min(b.x, window.innerWidth - size);
                b.y = Math.min(b.y, window.innerHeight - size);
            }
        });
    </script>
</body>
</html>
