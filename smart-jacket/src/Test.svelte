<script>
    import img from './assets/LUVvsTheWorld2.jpeg'
</script>

<img class="cd-image" src={img} alt="LUV vs The World album artwork" />

<style>
    .cd-image {
        display: block;
        width: min(80vw, 28rem);
        aspect-ratio: 1;
        object-fit: cover;
        border-radius: 50%;
        -webkit-mask-image: radial-gradient(
            circle,
            transparent 0 11%,
            #000 11.5% 100%
        );
        mask-image: radial-gradient(
            circle,
            transparent 0 11%,
            #000 11.5% 100%
        );
    }

</style>
