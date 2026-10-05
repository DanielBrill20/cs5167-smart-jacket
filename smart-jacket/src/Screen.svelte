<script>
    import adidasImage from './assets/Adidas.jpg';
    import eternalAtakeImage from './assets/EternalAtake.jpg';
    import luvIsRageImage from './assets/LuvisRage.jpg';
    import luvVsTheWorld2Image from './assets/LUVvsTheWorld2.jpeg';

    let { selectedMode = 0 } = $props();

    const images = [
        null,
        adidasImage,
        eternalAtakeImage,
        luvIsRageImage,
        luvVsTheWorld2Image,
        adidasImage,
        eternalAtakeImage
    ];
</script>

<div class="screen">
    {#if images[selectedMode]}
        <img src={images[selectedMode]} alt="" />
    {/if}
</div>

<style>
    .screen {
        width: min(24rem, 45vw);
        aspect-ratio: 3 / 5;
        background: #000;
    }

    img {
        display: block;
        width: 100%;
        height: 100%;
        object-fit: contain;
        object-position: center;
    }

    @media (max-width: 720px) {
        .screen {
            width: min(20rem, 80vw);
        }
    }
</style>
