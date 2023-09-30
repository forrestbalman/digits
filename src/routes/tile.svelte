<script>
    import { tweened } from "svelte/motion";

    export let number;
    export let active;
    export let ringColor;
    export let backgroundColor;
    export let duration;

    let progressRight, progressLeft;

    $: if (active) {
        progressRight = tweened(0, {
            duration: duration,
        });
        progressLeft = tweened(100, {
            duration: duration,
        });
        progressRight.set(100);
        progressLeft.set(0);
    }

    $: if ($progressRight === 100) {
        progressRight = tweened(0, {
            duration: 0,
        });
        progressLeft = tweened(100, {
            duration: 0,
        });
    }
</script>

<div class="col w-100 text-center p-0 text-light">
    <div class="outer-ring d-flex justify-content-center align-items-center rounded-circle p-1" style="background: conic-gradient({ringColor} {$progressRight}%, 0, {backgroundColor} {$progressLeft}%);">
        <div class="inner-ring w-100 h-100 d-flex justify-content-center align-items-center rounded-circle" style="background: {backgroundColor}">
            <h1 class="display-1 m-0">{number}</h1>
        </div>
    </div>
</div>

<style>
    * {
        font-family: "Inconsolata", monospace;
    }
    
    .outer-ring {
        width: 100px;
        height: 100px;
    }

    @media (max-width: 576px) {
        .outer-ring {
            width: 60px;
            height: 60px;
        }
    }
</style>
