<script>
    import adidasImage from './assets/Adidas.jpg';
    import eternalAtakeImage from './assets/EternalAtake.jpg';
    import luvIsRageImage from './assets/LuvisRage.jpg';
    import luvVsTheWorld2Image from './assets/LUVvsTheWorld2.jpeg';

    let imgPath = $state(adidasImage);
</script>

<div>
    <button
        style="background: red"
        onclick={() => imgPath = adidasImage}
    ></button>
    <button
        style="background: green"
        onclick={() => imgPath = eternalAtakeImage}
    ></button>
</div>
<div>
    <button
        style="background: blue"
        onclick={() => imgPath = luvIsRageImage}
    ></button>
    <button
        style="background: white"
        onclick={() => imgPath = luvVsTheWorld2Image}
    ></button>
</div>
<div>
    <img src={imgPath}>
</div>

<style>
    button {
		aspect-ratio: 1;
		border-radius: 10%;
		transition: all 0.1s;
		color: black;
		font-weight: 700;
		font-size: 2rem;
        width: 100px;
	}

    img {
        margin-top: 20px;
        width: 200px;
    }
</style>
