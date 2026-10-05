<script>
    import Dial from './Dial.svelte';
    import MovementVisualizer from './MovementVisualizer.svelte';
    import Screen from './Screen.svelte';

    let selectedMode = $state(0);
    let movement = $state({ x: 50, y: 50, speed: 0, angle: 0, id: 0 });

    function handleMovement(position) {
        movement = { ...position, id: movement.id + 1 };
    }
</script>

<main class="interface">
    <MovementVisualizer onMove={handleMovement} />
    <Dial
        {selectedMode}
        onModeChange={(mode) => selectedMode = mode}
    />
    <Screen {selectedMode} {movement} />
</main>