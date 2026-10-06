<script>
    import adidasImage from './assets/Adidas.jpg';
    import eternalAtakeImage from './assets/EternalAtake.jpg';
    import luvIsRageImage from './assets/LuvisRage.jpg';
    import Dial from './Dial.svelte';
    import MobileApp from './MobileApp.svelte';
    import MovementVisualizer from './MovementVisualizer.svelte';
    import Screen from './Screen.svelte';

    let selectedMode = $state(0);
    let movement = $state({ x: 50, y: 50, speed: 0, angle: 0, id: 0 });
    let presets = $state({
        1: { type: 'image', value: adidasImage },
        2: { type: 'image', value: eternalAtakeImage },
        3: { type: 'image', value: luvIsRageImage },
        4: null,
        5: null,
        6: null
    });

    function handleMovement(position) {
        movement = { ...position, id: movement.id + 1 };
    }

    function handlePresetChange(mode, preset) {
        presets = { ...presets, [mode]: preset };
    }
</script>

<main class="interface">
    <MobileApp {presets} onPresetChange={handlePresetChange} />
    <MovementVisualizer onMove={handleMovement} />
    <Dial
        {selectedMode}
        onModeChange={(mode) => selectedMode = mode}
    />
    <Screen {selectedMode} {movement} {presets} />
</main>