<script>
    import wristImage from './assets/hand.png';
    import Dial from './Dial.svelte';

    let { selectedMode = 0, onModeChange = () => {}, onMove = () => {} } = $props();
    let movementArea;
    let wristElement = $state();
    let isDragging = $state(false);
    let position = $state({ x: 57, y: 50 });
    let previousPosition = { x: 50, y: 50, time: 0 };

    function moveWrist(event) {
        if (!wristElement) return;

        const bounds = movementArea.getBoundingClientRect();
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

<div class="movement-area" bind:this={movementArea}>
    <div
        class:dragging={isDragging}
        class="wrist"
        style={`left: ${position.x}%; top: ${position.y}%`}
    >
        <img
            bind:this={wristElement}
            src={wristImage}
            alt="Draggable wrist movement control"
            draggable="false"
            onpointerdown={startDragging}
            onpointermove={(event) => isDragging && moveWrist(event)}
            onpointerup={stopDragging}
            onpointercancel={stopDragging}
            ondragstart={(event) => event.preventDefault()}
        />
        <div class="wrist-dial">
            <Dial {selectedMode} onModeChange={onModeChange} />
        </div>
    </div>
</div>

<style>
    .movement-area {
        position: fixed;
        z-index: 5;
        inset: 0;
        width: 100vw;
        height: 100vh;
        pointer-events: none;
    }

    .wrist {
        position: absolute;
        width: var(--hand-size, 30rem);
        transform: translate(-50%, -50%);
        pointer-events: none;
    }

    .wrist img {
        display: block;
        width: 100%;
        height: auto;
        cursor: grab;
        pointer-events: auto;
        touch-action: none;
        user-select: none;
        -webkit-user-drag: none;
    }

    .wrist.dragging {
        cursor: grabbing;
    }

    .wrist-dial {
        position: absolute;
        top: var(--dial-top, 34%);
        left: var(--dial-left, 26%);
        width: var(--dial-size, 7rem);
        transform: translate(-50%, -50%);
        pointer-events: auto;
    }

    .wrist-dial :global(.dial) {
        width: 100%;
    }
</style>
