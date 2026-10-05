<script>
    const modeCount = 7;

    let { selectedMode = 0, onModeChange = () => {} } = $props();
    let dialElement;
    let isDragging = $state(false);

    const angleForMode = (mode) => mode * (360 / modeCount);

    function selectMode(mode) {
        onModeChange(Math.max(0, Math.min(modeCount - 1, mode)));
    }

    function updateModeFromPointer(event) {
        if (!dialElement) return;

        const bounds = dialElement.getBoundingClientRect();
        const centerX = bounds.left + bounds.width / 2;
        const centerY = bounds.top + bounds.height / 2;
        const angle = Math.atan2(event.clientY - centerY, event.clientX - centerX);
        const dialAngle = (angle * 180 / Math.PI + 90 + 360) % 360;

        selectMode(Math.round(dialAngle / (360 / modeCount)) % modeCount);
    }

    function startDragging(event) {
        isDragging = true;
        dialElement?.setPointerCapture(event.pointerId);
        updateModeFromPointer(event);
    }

    function stopDragging(event) {
        isDragging = false;
        if (dialElement?.hasPointerCapture(event.pointerId)) {
            dialElement.releasePointerCapture(event.pointerId);
        }
    }

    function handleKeydown(event) {
        if (event.key === 'ArrowRight' || event.key === 'ArrowUp') {
            event.preventDefault();
            selectMode((selectedMode + 1) % modeCount);
        } else if (event.key === 'ArrowLeft' || event.key === 'ArrowDown') {
            event.preventDefault();
            selectMode((selectedMode + modeCount - 1) % modeCount);
        }
    }
</script>

<div
    class:dragging={isDragging}
    class="dial"
    bind:this={dialElement}
    role="slider"
    tabindex="0"
    aria-label="Jacket mode"
    aria-valuemin="0"
    aria-valuemax="6"
    aria-valuenow={selectedMode}
    onpointerdown={startDragging}
    onpointermove={(event) => isDragging && updateModeFromPointer(event)}
    onpointerup={stopDragging}
    onpointercancel={stopDragging}
    onkeydown={handleKeydown}
>
    <div class="puck">
        <div
            class="indicator"
            style={`--dial-angle: ${angleForMode(selectedMode)}deg`}
        ></div>
    </div>
</div>

<style>
    .dial {
        position: relative;
        width: min(20rem, 40vw);
        aspect-ratio: 1;
        perspective: 40rem;
        cursor: grab;
        touch-action: none;
        user-select: none;
    }

    .dial.dragging {
        cursor: grabbing;
    }

    .dial:focus-visible {
        outline: 2px solid currentColor;
        outline-offset: 0.4rem;
    }

    .puck {
        position: absolute;
        inset: 0;
        border: 0.25rem solid #4a4a4a;
        border-radius: 50%;
        background: #c8c8c8;
        transform: rotateX(14deg) rotateY(-12deg);
        transform-style: preserve-3d;
    }

    .puck::before {
        position: absolute;
        inset: 0;
        border-radius: 50%;
        background: #777;
        content: '';
        transform: translateX(-1.5rem) translateY(-1.5rem) translateZ(-0.1rem);
    }

    .indicator {
        position: absolute;
        top: 7%;
        left: calc(50% - 0.45rem);
        width: 0.9rem;
        height: 43%;
        border-radius: 50%;
        color: #222;
        transform: rotate(var(--dial-angle));
        transform-origin: 50% 100%;
        transition: transform 180ms ease;
    }

    .indicator::before {
        position: absolute;
        top: 0;
        width: 0.9rem;
        height: 0.9rem;
        border-radius: 50%;
        background: currentColor;
        content: '';
    }

    .dragging .indicator {
        transition: none;
    }

    @media (max-width: 720px) {
        .dial {
            width: min(18rem, 70vw);
        }
    }
</style>
