<script>
    import heartbreakImage from './assets/808s_heartbreak.webp';
    import cubeDesignImage from './assets/cube_design.jpg';
    import floralDesignImage from './assets/floral_design.jpg';
    import luvVsTheWorld2Image from './assets/LUVvsTheWorld2.jpeg';
    import monaLisaImage from './assets/mona_lisa.webp';
    import randomDogImage from './assets/random_dog.jpg';
    import murakamiOneImage from './assets/takashi_murakami1.jpg';
    import murakamiTwoImage from './assets/takashi_murakami2.avif';
    import waveDesignImage from './assets/wave_design.png';
    import dialSegments from './assets/dial-segments.svg';

    const galleryImages = [
        cubeDesignImage,
        floralDesignImage,
        waveDesignImage,
        monaLisaImage,
        murakamiOneImage,
        murakamiTwoImage,
        luvVsTheWorld2Image,
        heartbreakImage,
        randomDogImage,
    ];

    const segmentPaths = [
        'M265.175 0C305.194 0 343.142 8.86493 377.163 24.7383L332.174 117.995C311.761 108.688 289.074 103.5 265.175 103.5C241.239 103.5 218.518 108.703 198.08 118.038L153.227 24.7676C187.222 8.86118 225.164 0 265.175 0Z',
        'M383.467 27.7832C452.468 62.2337 504.343 125.883 522.906 202.543C522.818 202.556 522.73 202.573 522.642 202.593L421.986 225.682C410.509 179.964 379.567 141.975 338.479 121.037L383.467 27.7832Z',
        'M524.458 209.346C528.315 227.345 530.348 246.022 530.348 265.174C530.348 326.392 509.601 382.765 474.758 427.646C474.749 427.639 474.74 427.631 474.731 427.624L393.744 363.207C414.512 336.012 426.849 302.034 426.849 265.174C426.849 253.982 425.71 243.056 423.545 232.505L524.207 209.415C524.292 209.395 524.376 209.371 524.458 209.346Z',
        'M470.374 433.103C470.38 433.108 470.387 433.112 470.394 433.117C422.417 491.673 349.875 529.323 268.506 530.325V426.811C317.072 425.829 360.38 403.433 389.375 368.677L470.374 433.103Z',
        'M140.974 368.677C169.902 403.352 213.076 425.725 261.506 426.804V530.321C180.323 529.221 107.951 491.638 60.04 433.224L140.974 368.677Z',
        'M5.87012 209.584L106.77 232.669C104.626 243.169 103.501 254.04 103.501 265.174C103.501 302.034 115.837 336.012 136.604 363.207L55.6719 427.753C20.7786 382.854 4.14095e-05 326.44 0 265.174C1.07385e-05 246.104 2.0137 227.504 5.83887 209.575C5.84924 209.578 5.8597 209.582 5.87012 209.584Z',
        'M146.918 27.8018L191.776 121.084C150.686 142.057 119.757 180.088 108.32 225.843L7.43066 202.761C7.41746 202.758 7.40383 202.755 7.39062 202.752C25.9124 125.99 77.8288 62.2506 146.904 27.7715C146.909 27.7814 146.913 27.7918 146.918 27.8018Z'
    ];
    const segmentLabels = ['Off', '1', '2', '3', '4', '5', '6'];
    const labelPositions = segmentLabels.map((_, index) => {
        const angle = (index * 360 / segmentLabels.length - 90) * Math.PI / 180;
        const radius = 44;
        return {
            left: 50 + Math.cos(angle) * radius,
            top: 50 + Math.sin(angle) * radius
        };
    });
    let { presets, onPresetChange = () => {} } = $props();
    let selectedMode = $state(null);
    let drawer = $state('closed');
    let drawerOffset = $state(0);
    let isDraggingDrawer = $state(false);
    let drawerStartY = 0;

    function selectMode(mode) {
        if (selectedMode === mode) {
            closeDrawer();
            return;
        }
        selectedMode = mode;
        drawer = 'options';
    }

    function closeDrawer() {
        drawer = 'closed';
        selectedMode = null;
        drawerOffset = 0;
    }

    function setPreset(value) {
        if (selectedMode === null) return;
        onPresetChange(selectedMode, value);
        closeDrawer();
    }

    function startDrawerDrag(event) {
        drawerStartY = event.clientY;
        isDraggingDrawer = true;
        event.currentTarget.setPointerCapture(event.pointerId);
    }

    function dragDrawer(event) {
        if (isDraggingDrawer) {
            drawerOffset = Math.max(0, event.clientY - drawerStartY);
        }
    }

    function stopDrawerDrag(event) {
        if (!isDraggingDrawer) return;
        isDraggingDrawer = false;
        event.currentTarget.releasePointerCapture(event.pointerId);
        if (drawerOffset > 70) closeDrawer();
        else drawerOffset = 0;
    }

    function centerLabel() {
        if (selectedMode === null) return '';
        const preset = presets[selectedMode];
        return preset?.type === 'image' ? '' : preset?.type === 'visualizer' ? 'Movement Visualizer' : 'Not Set';
    }

    function segmentFill(index) {
        if (index === 0) return '#46505f';
        const preset = presets[index];
        if (preset?.type === 'image') return `url(#preset-image-${index})`;
        if (preset?.type === 'visualizer') return '#ffbf5c';
        return 'transparent';
    }
</script>

<section class="phone" aria-label="Smart jacket mobile app">
    <div class="donut-area">
        <div class="donut">
            <div class="donut-center">
                {#if selectedMode !== null && presets[selectedMode]?.type === 'image'}
                    <img src={presets[selectedMode].value} alt="" />
                {:else}
                    <span>{centerLabel()}</span>
                {/if}
            </div>
            <img class="donut-image" src={dialSegments} alt="" />
            <svg class="segment-controls" viewBox="0 0 531 531" aria-label="Jacket preset modes">
                <defs>
                    {#each segmentPaths as _, index}
                        {#if presets[index]?.type === 'image'}
                            <pattern id={`preset-image-${index}`} patternUnits="userSpaceOnUse" width="531" height="531">
                                <image
                                    href={presets[index].value}
                                    x="0"
                                    y="0"
                                    width="531"
                                    height="531"
                                    preserveAspectRatio="xMidYMid slice"
                                />
                            </pattern>
                        {/if}
                    {/each}
                </defs>
                {#each segmentPaths as path, index}
                    <!-- svelte-ignore a11y_no_noninteractive_tabindex a11y_click_events_have_key_events a11y_no_static_element_interactions -->
                    <path
                        class:selected={selectedMode === index}
                        class:off={index === 0}
                        d={path}
                        style={`fill: ${segmentFill(index)}`}
                        role={index === 0 ? undefined : 'button'}
                        tabindex={index === 0 ? -1 : 0}
                        aria-label={index === 0 ? 'Off' : `Customize mode ${index}`}
                        onclick={index === 0 ? undefined : () => selectMode(index)}
                        onkeydown={index === 0 ? undefined : (event) => event.key === 'Enter' && selectMode(index)}
                    />
                {/each}
            </svg>
            <div class="segment-labels" aria-hidden="true">
                {#each segmentLabels as label, index}
                    <span
                        class:selected={selectedMode === index}
                        class={`label label-${index}`}
                        style={`left: ${labelPositions[index].left}%; top: ${labelPositions[index].top}%`}
                    >{label}</span>
                {/each}
            </div>
        </div>
        {#if selectedMode === null}
            <p class="prompt">Select a mode to customize</p>
        {/if}
    </div>

    {#if drawer !== 'closed'}
        <div
            class:dragging={isDraggingDrawer}
            class="drawer"
            style={`--drawer-offset: ${drawerOffset}px`}
        >
            <div
                class="handle"
                role="button"
                tabindex="0"
                aria-label="Close drawer"
                onpointerdown={startDrawerDrag}
                onpointermove={dragDrawer}
                onpointerup={stopDrawerDrag}
                onpointercancel={stopDrawerDrag}
                onkeydown={(event) => event.key === 'Enter' && closeDrawer()}
            ></div>
            {#if drawer === 'options'}
                <div class="drawer-options">
                    <button type="button" onclick={() => drawer = 'gallery'}>Gallery</button>
                    <button type="button" onclick={() => setPreset({ type: 'visualizer' })}>Movement Visualizer</button>
                    <button class="clear-mode" type="button" onclick={() => setPreset(null)}>Clear Mode</button>
                </div>
            {:else}
                <div class="gallery">
                    {#each galleryImages as image, index}
                        <button type="button" aria-label={`Select gallery image ${index + 1}`} onclick={() => setPreset({ type: 'image', value: image })}>
                            <img src={image} alt="" />
                        </button>
                    {/each}
                </div>
            {/if}
        </div>
    {/if}
</section>

<style>
    .phone {
        width: min(17rem, 22vw);
        min-width: 16rem;
        aspect-ratio: 9 / 18;
        overflow: hidden;
        padding: 2rem 1.2rem 0;
        border: 0.35rem solid #111827;
        border-radius: 1.5rem;
        background: #182131;
        color: #dce6f2;
        font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
        position: relative;
    }

    .donut-area {
        display: grid;
        justify-items: center;
        gap: 2rem;
    }

    .donut {
        position: relative;
        width: 100%;
        aspect-ratio: 1;
    }

    .donut-center {
        position: absolute;
        z-index: 2;
        inset: 31.8%;
        display: grid;
        place-items: center;
        overflow: hidden;
        border-radius: 50%;
        background: #182131;
        color: #e5edf6;
        font-size: clamp(0.55rem, 1.1vw, 0.85rem);
        line-height: 1.1;
        text-align: center;
        text-wrap: balance;
    }

    .donut-center img {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }

    .donut-image,
    .segment-labels {
        position: absolute;
        inset: 0;
        width: 100%;
        height: 100%;
    }

    .donut-image,
    .segment-controls {
        position: absolute;
        inset: 12%;
        width: 76%;
        height: 76%;
    }

    .donut-image {
        z-index: 1;
    }

    .segment-controls {
        z-index: 2;
    }

    .segment-controls path {
        fill: transparent;
        cursor: pointer;
        outline: none;
    }

    .segment-controls path.selected {
        filter: brightness(1.12);
    }

    .segment-controls path:focus-visible {
        filter: brightness(1.12);
        outline: none;
    }

    .segment-controls path.off {
        cursor: not-allowed;
        pointer-events: none;
    }

    .segment-labels {
        z-index: 3;
        color: #e5edf6;
        font-size: 1.1rem;
        pointer-events: none;
    }

    .segment-labels .label {
        position: absolute;
        transform: translate(-50%, -50%);
    }

    .segment-labels .label.selected {
        color: #62d9e8;
    }

    .prompt {
        margin: 0;
        color: #dce6f2;
        font-size: 1.25rem;
        text-align: center;
    }

    .drawer {
        position: absolute;
        right: 0;
        bottom: 0;
        left: 0;
        height: 35%;
        min-height: 11rem;
        padding: 2rem 1rem 1rem;
        border-radius: 1.4rem 1.4rem 0 0;
        background: #0e1522;
        overflow: hidden;
        transform: translateY(var(--drawer-offset));
        transition: transform 180ms ease;
    }

    .drawer.dragging {
        transition: none;
    }

    .handle {
        position: absolute;
        top: 0.7rem;
        left: calc(50% - 2rem);
        width: 4rem;
        height: 0.3rem;
        border-radius: 1rem;
        background: #dce6f2;
        cursor: grab;
        touch-action: none;
    }

    .drawer.dragging .handle {
        cursor: grabbing;
    }

    .drawer-options {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 1rem;
    }

    .drawer-options .clear-mode {
        grid-column: 1 / -1;
        min-height: 3rem;
    }

    .drawer button {
        min-height: 6rem;
        border: 0;
        border-radius: 1rem;
        background: #dce6f2;
        color: #182131;
        font: inherit;
        font-size: 1.2rem;
        cursor: pointer;
    }

    .gallery {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 0.7rem;
        height: 100%;
        grid-auto-rows: min-content;
        align-content: start;
        overflow-y: auto;
        padding-bottom: 0.5rem;
        scrollbar-width: none;
        -ms-overflow-style: none;
        touch-action: pan-y;
    }

    .gallery::-webkit-scrollbar {
        display: none;
    }

    .gallery button {
        width: 100%;
        min-height: 0;
        aspect-ratio: 1;
        padding: 0;
        overflow: hidden;
        background: #cbd7e5;
    }

    .gallery img {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }

    @media (max-width: 1100px) {
        .phone {
            width: min(19rem, 42vw);
        }
    }

    @media (max-width: 720px) {
        .phone {
            width: min(19rem, 90vw);
        }
    }
</style>
