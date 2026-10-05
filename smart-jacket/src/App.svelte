<script>
    import Dial from './Dial.svelte';
    import Screen from './Screen.svelte';

    let selectedMode = $state(0);
</script>

<main class="interface">
    <Dial
        {selectedMode}
        onModeChange={(mode) => selectedMode = mode}
    />
    <Screen {selectedMode} />
</main>