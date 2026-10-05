<script>
    import adidasImage from './assets/Adidas.jpg';

    let { onMove = () => {} } = $props();
    let wristElement = $state();
    let isDragging = $state(false);
    let position = $state({ x: 50, y: 50 });
    let previousPosition = { x: 50, y: 50, time: 0 };

    function moveWrist(event) {
        if (!wristElement) return;

        const bounds = wristElement.parentElement.getBoundingClientRect();
        const nextPosition = {
            x: Math.max(12, Math.min(88, ((event.clientX - bounds.left) / bounds.width) * 100)),
            y: Math.max(12, Math.min(88, ((event.clientY - bounds.top) / bounds.height) * 100))
        };
        const now = performance.now();
        const elapsed = Math.max(16, now - previousPosition.time);
        const distance = Math.hypot(
            nextPosition.x - previousPosition.x,
            nextPosition.y - previousPosition.y
        );

        position = nextPosition;
        onMove({
            ...nextPosition,
            speed: Math.min(1, distance / elapsed * 12),
            angle: Math.atan2(
                nextPosition.y - previousPosition.y,
                nextPosition.x - previousPosition.x
            ),
            id: now
        });
        previousPosition = { ...nextPosition, time: now };
    }

    function startDragging(event) {
        event.preventDefault();
        isDragging = true;
        wristElement.setPointerCapture(event.pointerId);
        moveWrist(event);
    }

    function stopDragging(event) {
        isDragging = false;
        if (wristElement.hasPointerCapture(event.pointerId)) {
            wristElement.releasePointerCapture(event.pointerId);
        }
    }
</script>

<div class="movement-area">
    <img
        class:dragging={isDragging}
        class="wrist"
        bind:this={wristElement}
        src={adidasImage}
        alt="Draggable wrist movement control"
        draggable="false"
        style={`left: ${position.x}%; top: ${position.y}%`}
        onpointerdown={startDragging}
        onpointermove={(event) => isDragging && moveWrist(event)}
        onpointerup={stopDragging}
        onpointercancel={stopDragging}
        ondragstart={(event) => event.preventDefault()}
    />
</div>

<style>
    .movement-area {
        position: relative;
        width: min(16rem, 25vw);
        height: min(24rem, 60vh);
        border: 1px solid currentColor;
        overflow: hidden;
    }

    .wrist {
        position: absolute;
        width: 7rem;
        height: 7rem;
        object-fit: cover;
        transform: translate(-50%, -50%);
        cursor: grab;
        touch-action: none;
        user-select: none;
        -webkit-user-drag: none;
    }

    .wrist.dragging {
        cursor: grabbing;
    }

    @media (max-width: 720px) {
        .movement-area {
            width: min(20rem, 80vw);
            height: 10rem;
        }
    }
</style>
