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
    let showInfo = $state(false);
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
    <header class="project-header">
        <div>
            <h1>CS 5167 Smart Jacket</h1>
            <p>Daniel Brill</p>
        </div>
        <a href="https://github.com/DanielBrill20/cs5167-smart-jacket" target="_blank" rel="noreferrer">
            Project repository
        </a>
        <button type="button" aria-expanded={showInfo} onclick={() => showInfo = !showInfo}>
            {showInfo ? 'Close info' : 'Info'}
        </button>
    </header>

    {#if showInfo}
        <aside class="info-panel">
            <h2>How to use</h2>
            <p>Rotate the wrist dial to change the jacket's display mode.</p>
            <p>Use the phone app to program the dial by selecting modes, assigning them images or a movement visualizer, and clearing them.</p>
            <p>Drag the hand to simulate movement. The movement visualizer responds on the jacket screen.</p>
        </aside>
    {/if}

    <div class="jacket-display">
        <img src={jacketImage} alt="Back of a jacket" />
        <Screen {selectedMode} {movement} {presets} />
    </div>
    <MovementVisualizer
        {selectedMode}
        onModeChange={(mode) => selectedMode = mode}
        onMove={handleMovement}
    />
    <MobileApp {presets} onPresetChange={handlePresetChange} />
</main>

<style>
    .project-header {
        position: fixed;
        z-index: 10;
        top: 1rem;
        left: 1rem;
        display: flex;
        align-items: center;
        gap: 1rem;
        color: #182131;
    }

    .project-header h1,
    .project-header p {
        margin: 0;
    }

    .project-header h1 {
        font-size: 1.45rem;
    }

    .project-header p {
        color: #526176;
        font-size: 1.05rem;
    }

    .project-header a,
    .project-header button {
        border: 1px solid #526176;
        border-radius: 0.4rem;
        background: #182131;
        color: #dce6f2;
        font: inherit;
        font-size: 0.85rem;
        padding: 0.45rem 0.7rem;
        text-decoration: none;
    }

    .project-header button {
        cursor: pointer;
    }

    .info-panel {
        position: fixed;
        z-index: 10;
        top: 5rem;
        left: 1rem;
        width: min(20rem, calc(100vw - 2rem));
        padding: 1rem;
        border: 1px solid #526176;
        border-radius: 0.6rem;
        background: #0e1522;
        color: #dce6f2;
        font-size: 0.9rem;
    }

    .info-panel h2,
    .info-panel p {
        margin: 0;
    }

    .info-panel h2 {
        margin-bottom: 0.6rem;
        font-size: 1rem;
    }

    .info-panel p + p {
        margin-top: 0.6rem;
    }

    .jacket-display {
        position: relative;
        flex: 0 0 auto;
        width: min(88rem, 70vw, calc(100vh - 2rem));
        aspect-ratio: 1;
        transform: translateX(-3vw);
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
        .interface {
            padding-top: 5rem;
        }

        .jacket-display {
            width: min(33rem, 94vw, calc(100vh - 2rem));
        }
    }
</style>