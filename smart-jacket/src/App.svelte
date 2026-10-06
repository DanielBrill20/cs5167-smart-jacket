<script>
    import heartbreakImage from './assets/808s_heartbreak.webp';
    import cubeDesignImage from './assets/cube_design.jpg';
    import floralDesignImage from './assets/floral_design.jpg';
    import waveDesignImage from './assets/wave_design.png';
    import MobileApp from './MobileApp.svelte';
    import MovementVisualizer from './MovementVisualizer.svelte';
    import Screen from './Screen.svelte';
    import jacketImage from './assets/jacket.png';

    let selectedMode = $state(0);
    let movement = $state({ x: 50, y: 50, speed: 0, angle: 0, id: 0 });
    let presets = $state({
        1: { type: 'image', value: cubeDesignImage },
        2: { type: 'image', value: floralDesignImage },
        3: { type: 'image', value: waveDesignImage },
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
    <MovementVisualizer
        {selectedMode}
        onModeChange={(mode) => selectedMode = mode}
        onMove={handleMovement}
    />
    <div class="jacket-display">
        <img src={jacketImage} alt="Back of a jacket" />
        <Screen {selectedMode} {movement} {presets} />
    </div>
</main>

<style>
    .jacket-display {
        position: relative;
        flex: 0 0 auto;
        width: min(88rem, 96vw, calc(100vh - 2rem));
        aspect-ratio: 1;
    }

    .jacket-display > img {
        display: block;
        width: 100%;
        height: 100%;
        object-fit: contain;
    }

    .jacket-display :global(.screen) {
        position: absolute;
        top: 27%;
        left: 50%;
        width: 31%;
        transform: translateX(-50%);
    }

    @media (max-width: 720px) {
        .jacket-display {
            width: min(33rem, 94vw, calc(100vh - 2rem));
        }
    }
</style>