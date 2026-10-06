<script>
    import { onMount } from 'svelte';
    let {
        selectedMode = 0,
        movement = { x: 50, y: 50, speed: 0, angle: 0, id: 0 },
        presets = {}
    } = $props();
    let canvasElement = $state();

    let isVisualizer = $derived(presets[selectedMode]?.type === 'visualizer');

    const visualizerConfig = {
        contourDensity: 22,
        contourAmplitude: 15,
        contourSpeed: 0.00012,
        fieldScale: 0.012,
        fieldStrength: 1,
        motionSensitivity: 1.8,
        motionSmoothing: 0.12,
        waveSpeed: 0.00045,
        waveStrength: 22,
        waveDecay: 0.002,
        waveRadius: 1.6,
        fiberDensity: 120,
        fiberLength: 12,
        fiberResponse: 0.7,
        idleMotion: 0.00018
    };

    let latestMovement = { x: 50, y: 50, speed: 0, angle: 0, id: 0 };
    let previousMovement = latestMovement;

    $effect(() => {
        latestMovement = movement;
    });

    onMount(() => {
        if (!canvasElement) return;

        const context = canvasElement.getContext('2d');
        const waves = [];
        const fibers = Array.from({ length: visualizerConfig.fiberDensity }, (_, index) => ({
            x: ((index * 73) % 100) / 100,
            y: ((index * 137) % 100) / 100,
            phase: index * 1.71
        }));
        let width = 1;
        let height = 1;
        let pixelRatio = 1;
        let elapsed = 0;
        let previousTime = performance.now();
        let lastWaveTime = -Infinity;
        let animationFrame;

        const resize = () => {
            const bounds = canvasElement.getBoundingClientRect();
            pixelRatio = Math.min(window.devicePixelRatio || 1, 2);
            width = Math.max(1, bounds.width);
            height = Math.max(1, bounds.height);
            canvasElement.width = width * pixelRatio;
            canvasElement.height = height * pixelRatio;
            context.setTransform(pixelRatio, 0, 0, pixelRatio, 0, 0);
        };

        const fieldAt = (x, y, time) => {
            let value =
                Math.sin(x * visualizerConfig.fieldScale * 1.7 + time * visualizerConfig.idleMotion) +
                Math.sin(y * visualizerConfig.fieldScale * 1.3 - time * visualizerConfig.idleMotion * 0.8) +
                Math.sin((x + y) * visualizerConfig.fieldScale * 0.9 + time * visualizerConfig.idleMotion * 0.6);

            for (const wave of waves) {
                const dx = x - wave.x;
                const dy = y - wave.y;
                const distance = Math.hypot(dx, dy);
                const ring = distance - wave.radius;
                const influence = Math.exp(-(ring * ring) / (wave.width * wave.width)) * wave.strength;
                const direction = (dx * wave.directionX + dy * wave.directionY) / Math.max(distance, 1);
                value += influence * (0.65 + direction * 0.35);
            }

            return value * visualizerConfig.fieldStrength;
        };

        const addWave = (force) => {
            const angle = latestMovement.angle || 0;
            waves.push({
                x: (latestMovement.x / 100) * width,
                y: (latestMovement.y / 100) * height,
                directionX: Math.cos(angle),
                directionY: Math.sin(angle),
                strength: Math.min(2.5, force) * visualizerConfig.waveStrength,
                radius: 0,
                width: 22,
                age: 0,
                lifetime: 2600
            });
        };

        const drawContours = (time) => {
            context.lineWidth = 0.8;
            for (let line = 0; line < visualizerConfig.contourDensity; line += 1) {
                const baseY = (line + 0.5) * height / visualizerConfig.contourDensity;
                context.beginPath();
                for (let step = 0; step <= 50; step += 1) {
                    const x = step * width / 50;
                    const field = fieldAt(x, baseY, time);
                    const y = baseY +
                        Math.sin(field * 1.7 + time * visualizerConfig.contourSpeed) *
                        visualizerConfig.contourAmplitude +
                        Math.sin(x * 0.025 + line * 0.7 + time * visualizerConfig.contourSpeed * 1.5) * 5;
                    if (step === 0) context.moveTo(x, y);
                    else context.lineTo(x, y);
                }
                context.strokeStyle = `rgba(116, 209, 195, ${0.14 + (line % 4) * 0.025})`;
                context.stroke();
            }
        };

        const drawFibers = (time) => {
            context.lineWidth = 0.55;
            for (const fiber of fibers) {
                const x = fiber.x * width;
                const y = fiber.y * height;
                const field = fieldAt(x, y, time);
                const flow = fieldAt(x + 4, y, time) - fieldAt(x - 4, y, time);
                const angle = fiber.phase + field * visualizerConfig.fiberResponse + flow * 0.8;
                const length = visualizerConfig.fiberLength;
                context.beginPath();
                context.moveTo(x, y);
                context.lineTo(x + Math.cos(angle) * length, y + Math.sin(angle) * length);
                context.strokeStyle = 'rgba(201, 235, 223, 0.18)';
                context.stroke();
            }
        };

        const drawWaves = () => {
            for (const wave of waves) {
                const alpha = Math.max(0, 1 - wave.age / wave.lifetime);
                context.save();
                context.translate(wave.x, wave.y);
                context.rotate(Math.atan2(wave.directionY, wave.directionX));
                context.scale(1 + alpha * 0.8, 0.75 + alpha * 0.25);
                context.beginPath();
                context.ellipse(0, 0, wave.radius, wave.radius * 0.45, 0, 0, Math.PI * 2);
                context.strokeStyle = `rgba(255, 191, 92, ${alpha * 0.35})`;
                context.lineWidth = 1.2;
                context.stroke();
                context.restore();
            }
        };

        const render = (time) => {
            const delta = Math.min(50, time - previousTime);
            previousTime = time;
            elapsed += delta;

            if (isVisualizer) {
                const movementDelta = Math.hypot(
                    latestMovement.x - previousMovement.x,
                    latestMovement.y - previousMovement.y
                );
                if (
                    latestMovement.id !== previousMovement.id &&
                    movementDelta > 0.8 &&
                    time - lastWaveTime > 140
                ) {
                    addWave(Math.max(0.35, latestMovement.speed * visualizerConfig.motionSensitivity));
                    lastWaveTime = time;
                    previousMovement = { ...latestMovement };
                }

                for (let index = waves.length - 1; index >= 0; index -= 1) {
                    const wave = waves[index];
                    wave.age += delta;
                    wave.radius += delta * visualizerConfig.waveSpeed * height;
                    wave.strength *= 1 - visualizerConfig.waveDecay * delta / 1000;
                    if (wave.age > wave.lifetime || wave.strength < 0.5) waves.splice(index, 1);
                }

                context.clearRect(0, 0, width, height);
                context.fillStyle = '#071b2a';
                context.fillRect(0, 0, width, height);
                drawContours(elapsed);
                drawFibers(elapsed);
                drawWaves();
            }

            animationFrame = requestAnimationFrame(render);
        };

        const resizeObserver = new ResizeObserver(resize);
        resizeObserver.observe(canvasElement);
        resize();
        animationFrame = requestAnimationFrame(render);

        return () => {
            cancelAnimationFrame(animationFrame);
            resizeObserver.disconnect();
        };
    });
</script>

<div class:visualizer={isVisualizer} class="screen">
    <canvas bind:this={canvasElement} aria-label="Movement visualizer"></canvas>
    {#if !isVisualizer && presets[selectedMode]?.type === 'image'}
        <img src={presets[selectedMode].value} alt="" />
    {/if}
</div>

<style>
    .screen {
        position: relative;
        overflow: hidden;
        width: min(24rem, 45vw);
        aspect-ratio: 3 / 5;
        background: #000;
    }

    img,
    canvas {
        display: block;
        width: 100%;
        height: 100%;
    }

    canvas {
        position: absolute;
        inset: 0;
        visibility: hidden;
    }

    .visualizer canvas {
        visibility: visible;
    }

    img {
        object-fit: contain;
        object-position: center;
    }

    .visualizer {
        cursor: crosshair;
    }

    @media (max-width: 720px) {
        .screen {
            width: min(20rem, 80vw);
        }
    }
</style>
