<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fireworks Generator</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --panel-bg: rgba(30, 41, 59, 0.9);
            --text-color: #e2e8f0;
            --accent-color: #38bdf8;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
        }

        body {
            background-color: #000;
            color: var(--text-color);
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            overflow: hidden;
            height: 100vh;
            width: 100vw;
        }

        /* Canvas Layer */
        #canvas {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
            cursor: crosshair;
        }

        /* UI Layer */
        #ui-container {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            width: 280px;
            background: var(--panel-bg);
            backdrop-filter: blur(10px);
            padding: 20px;
            border-radius: 12px;
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
            transition: transform 0.3s ease;
        }

        #ui-container.collapsed {
            transform: translateX(-320px);
        }

        .toggle-btn {
            position: absolute;
            right: -40px;
            top: 0;
            width: 40px;
            height: 40px;
            background: var(--panel-bg);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-left: none;
            border-radius: 0 8px 8px 0;
            color: white;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            font-size: 1.2rem;
        }

        h1 {
            font-size: 1.2rem;
            margin-bottom: 15px;
            color: var(--accent-color);
            text-transform: uppercase;
            letter-spacing: 1px;
            font-weight: 800;
        }

        .control-group {
            margin-bottom: 15px;
        }

        label {
            display: flex;
            justify-content: space-between;
            font-size: 0.85rem;
            margin-bottom: 5px;
            font-weight: 600;
            color: #94a3b8;
        }

        span.value-display {
            color: var(--accent-color);
        }

        input[type="range"] {
            width: 100%;
            height: 6px;
            background: #334155;
            border-radius: 3px;
            outline: none;
            -webkit-appearance: none;
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: var(--accent-color);
            border-radius: 50%;
            cursor: pointer;
            transition: background 0.2s;
        }

        input[type="range"]::-webkit-slider-thumb:hover {
            background: #fff;
        }

        .checkbox-group {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-bottom: 15px;
        }

        input[type="checkbox"] {
            width: 16px;
            height: 16px;
            accent-color: var(--accent-color);
            cursor: pointer;
        }

        .button-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 20px;
        }

        button {
            padding: 10px;
            background: #334155;
            border: none;
            border-radius: 6px;
            color: white;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            font-size: 0.85rem;
        }

        button:hover {
            background: #475569;
        }

        button.primary {
            background: var(--accent-color);
            color: #0f172a;
        }

        button.primary:hover {
            background: #7dd3fc;
        }

        .color-picker {
            display: flex;
            gap: 8px;
            margin-top: 5px;
        }

        .color-option {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid transparent;
            transition: transform 0.2s;
        }

        .color-option.selected {
            border-color: white;
            transform: scale(1.2);
        }

        .instructions {
            position: absolute;
            bottom: 20px;
            width: 100%;
            text-align: center;
            color: rgba(255,255,255,0.3);
            pointer-events: none;
            font-size: 0.9rem;
            z-index: 5;
        }

    </style>
</head>
<body>

    <div id="ui-container">
        <div class="toggle-btn" id="togglePanel">⚙️</div>
        <h1>Pyro Generator</h1>

        <div class="control-group">
            <label>Hue (Color) <span id="hueVal" class="value-display">Multi</span></label>
            <input type="range" id="hue" min="0" max="361" value="361">
            <div class="color-picker" id="quickColors">
                <div class="color-option" style="background: #ff0000" data-hue="0"></div>
                <div class="color-option" style="background: #ffff00" data-hue="60"></div>
                <div class="color-option" style="background: #00ff00" data-hue="120"></div>
                <div class="color-option" style="background: #00ffff" data-hue="180"></div>
                <div class="color-option" style="background: #0000ff" data-hue="240"></div>
                <div class="color-option" style="background: #ff00ff" data-hue="300"></div>
                <div class="color-option" style="background: linear-gradient(45deg, red, blue);" data-hue="361"></div>
            </div>
        </div>

        <div class="control-group">
            <label>Particles <span id="partVal" class="value-display">100</span></label>
            <input type="range" id="particles" min="30" max="400" value="100">
        </div>

        <div class="control-group">
            <label>Explosion Force <span id="forceVal" class="value-display">5</span></label>
            <input type="range" id="force" min="1" max="15" value="5">
        </div>

        <div class="control-group">
            <label>Gravity <span id="gravVal" class="value-display">Normal</span></label>
            <input type="range" id="gravity" min="0.05" max="0.3" step="0.01" value="0.12">
        </div>

        <div class="control-group">
            <label>Trail Length <span id="trailVal" class="value-display">Long</span></label>
            <input type="range" id="trail" min="0.05" max="0.5" step="0.01" value="0.15">
        </div>

        <div class="checkbox-group">
            <input type="checkbox" id="autoFire">
            <label for="autoFire" style="margin:0; cursor:pointer;">Auto-Fire Mode</label>
        </div>

        <div class="button-group">
            <button onclick="clearCanvas()">Clear Sky</button>
            <button class="primary" onclick="launchRandom()">Launch Now</button>
        </div>
    </div>

    <div class="instructions">Tap or Click anywhere to launch • Desktop & Mobile Supported</div>
    <canvas id="canvas"></canvas>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        // Settings State
        const settings = {
            hue: 361, // 361 = multicolor/random
            particleCount: 100,
            explosionForce: 5,
            gravity: 0.12,
            trail: 0.15, // Lower number = longer trails (less opacity on clearRect)
            autoFire: false
        };

        // Resize Canvas
        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Arrays to hold active objects
        let fireworks = [];
        let particles = [];

        // Utility Functions
        const random = (min, max) => Math.random() * (max - min) + min;
        
        // Classes
        class Firework {
            constructor(sx, sy, tx, ty) {
                this.x = sx;
                this.y = sy;
                this.sx = sx;
                this.sy = sy;
                this.tx = tx;
                this.ty = ty;
                this.distanceToTarget = Math.hypot(sx - tx, sy - ty);
                this.distanceTraveled = 0;
                
                // Calculate angle and speed
                this.angle = Math.atan2(ty - sy, tx - sx);
                this.speed = 2;
                this.acceleration = 1.05;
                this.brightness = random(50, 70);
                
                // Determined by settings at launch time
                this.targetHue = settings.hue === 361 ? random(0, 360) : settings.hue;
            }

            update(index) {
                // Accelerate
                this.speed *= this.acceleration;
                
                // Calculate velocity
                const vx = Math.cos(this.angle) * this.speed;
                const vy = Math.sin(this.angle) * this.speed;
                
                this.distanceTraveled = Math.hypot(this.sx - this.x, this.sy - this.y);
                
                // Reached target?
                if (this.distanceTraveled >= this.distanceToTarget) {
                    createParticles(this.tx, this.ty, this.targetHue);
                    fireworks.splice(index, 1);
                } else {
                    this.x += vx;
                    this.y += vy;
                }
            }

            draw() {
                ctx.beginPath();
                ctx.moveTo(
                    this.x - Math.cos(this.angle) * (this.speed * 0.5), // Trail effect
                    this.y - Math.sin(this.angle) * (this.speed * 0.5)
                );
                ctx.lineTo(this.x, this.y);
                ctx.strokeStyle = `hsl(${this.targetHue}, 100%, ${this.brightness}%)`;
                ctx.lineWidth = 3;
                ctx.stroke();
            }
        }

        class Particle {
            constructor(x, y, hue) {
                this.x = x;
                this.y = y;
                this.hue = hue;
                this.brightness = random(50, 80);
                this.alpha = 1;
                this.decay = random(0.01, 0.03);
                
                // Explosion physics
                const angle = random(0, Math.PI * 2);
                const speed = random(1, settings.explosionForce * 1.5); // Scaled by force setting
                this.vx = Math.cos(angle) * speed;
                this.vy = Math.sin(angle) * speed;
                
                // Gravity setting
                this.gravity = settings.gravity; 
                this.friction = 0.95;
            }

            update(index) {
                this.vx *= this.friction;
                this.vy *= this.friction;
                this.vy += this.gravity;
                
                this.x += this.vx;
                this.y += this.vy;
                this.alpha -= this.decay;

                // Slight color shift for shimmer
                if(Math.random() > 0.8) this.brightness = 100;
                else this.brightness = random(50, 80);

                if (this.alpha <= this.decay) {
                    particles.splice(index, 1);
                }
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, 2, 0, Math.PI * 2);
                ctx.fillStyle = `hsla(${this.hue}, 100%, ${this.brightness}%, ${this.alpha})`;
                ctx.fill();
            }
        }

        function createParticles(x, y, hue) {
            let count = parseInt(settings.particleCount);
            // If multicolor mode, particles can have slight hue variance
            for (let i = 0; i < count; i++) {
                let pHue = hue;
                if(settings.hue === 361) {
                    // If random mode, add variance to individual particles
                    pHue = random(hue - 20, hue + 20);
                }
                particles.push(new Particle(x, y, pHue));
            }
        }

        // Main Animation Loop
        function loop() {
            requestAnimationFrame(loop);

            // Create trails by filling with semi-transparent black
            // The lower the alpha (trail setting), the longer the trails persist
            ctx.globalCompositeOperation = 'destination-out';
            ctx.fillStyle = `rgba(0, 0, 0, ${settings.trail})`;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.globalCompositeOperation = 'lighter';

            // Update Fireworks
            for (let i = fireworks.length - 1; i >= 0; i--) {
                fireworks[i].draw();
                fireworks[i].update(i);
            }

            // Update Particles
            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].draw();
                particles[i].update(i);
            }

            // Auto Fire Logic
            if (settings.autoFire && fireworks.length < 3 && Math.random() < 0.05) {
                launchRandom();
            }
        }

        // User Interaction
        function launch(tx, ty) {
            // Start from bottom center-ish
            const startX = canvas.width / 2 + random(-200, 200); 
            const startY = canvas.height;
            fireworks.push(new Firework(startX, startY, tx, ty));
        }

        function launchRandom() {
            const margin = canvas.width * 0.2;
            const tx = random(margin, canvas.width - margin);
            const ty = random(canvas.height * 0.1, canvas.height * 0.5);
            launch(tx, ty);
        }

        canvas.addEventListener('mousedown', (e) => {
            launch(e.clientX, e.clientY);
        });

        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            for (let i = 0; i < e.touches.length; i++) {
                launch(e.touches[i].clientX, e.touches[i].clientY);
            }
        }, {passive: false});

        // UI Event Listeners
        document.getElementById('hue').addEventListener('input', (e) => {
            settings.hue = parseInt(e.target.value);
            const val = settings.hue > 360 ? "Multi" : settings.hue;
            document.getElementById('hueVal').innerText = val;
            
            // Update color picker visual state
            document.querySelectorAll('.color-option').forEach(el => el.classList.remove('selected'));
        });

        // Quick Color Picker Logic
        document.querySelectorAll('.color-option').forEach(opt => {
            opt.addEventListener('click', (e) => {
                const h = parseInt(e.target.getAttribute('data-hue'));
                settings.hue = h;
                document.getElementById('hue').value = h;
                document.getElementById('hueVal').innerText = h > 360 ? "Multi" : h;
                
                document.querySelectorAll('.color-option').forEach(el => el.classList.remove('selected'));
                e.target.classList.add('selected');
            });
        });

        document.getElementById('particles').addEventListener('input', (e) => {
            settings.particleCount = e.target.value;
            document.getElementById('partVal').innerText = e.target.value;
        });

        document.getElementById('force').addEventListener('input', (e) => {
            settings.explosionForce = e.target.value;
            document.getElementById('forceVal').innerText = e.target.value;
        });

        document.getElementById('gravity').addEventListener('input', (e) => {
            settings.gravity = parseFloat(e.target.value);
            let text = "Normal";
            if(settings.gravity < 0.1) text = "Low (Space)";
            if(settings.gravity > 0.2) text = "High";
            document.getElementById('gravVal').innerText = text;
        });
        
        document.getElementById('trail').addEventListener('input', (e) => {
            // Invert logic for display: Low value = Long trail
            settings.trail = parseFloat(e.target.value);
            let text = "Normal";
            if(settings.trail < 0.1) text = "Long";
            if(settings.trail > 0.3) text = "Short";
            document.getElementById('trailVal').innerText = text;
        });

        document.getElementById('autoFire').addEventListener('change', (e) => {
            settings.autoFire = e.target.checked;
        });

        document.getElementById('togglePanel').addEventListener('click', () => {
            document.getElementById('ui-container').classList.toggle('collapsed');
        });

        function clearCanvas() {
            particles = [];
            fireworks = [];
            // Force a clear
            ctx.globalCompositeOperation = 'source-over';
            ctx.fillStyle = 'black';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        // Start
        // Select Multi by default in UI
        document.querySelector('.color-option:last-child').classList.add('selected');
        loop();

    </script>
</body>
</html>
