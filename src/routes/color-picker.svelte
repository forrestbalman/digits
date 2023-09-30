<script>
    import { createEventDispatcher } from "svelte";
    const dispatch = createEventDispatcher();
    export let color;

    function changeColor() {
        dispatch("changeColor", color);
    }
</script>

<div class="col mb-1">
    <button class="color btn border border-2 rounded-2" style="background: {color};" on:click={changeColor} />
</div>

<style>
    .color {
        width: 24px;
        height: 24px;
    }
</style>
