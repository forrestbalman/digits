<script>
    import Tile from "./tile.svelte";
    import ColorPicker from "./color-picker.svelte";
    import { fade } from "svelte/transition";
    import { colorsBackground, colorsRing } from "$lib/index.js";

    let time = 0;
    let minutes, seconds, start, end, min, max, modal, articulation, dynamic, updateTimeTimeout;
    let timeouts = [];
    let showArticulation = true;
    let showDynamic = true;
    let numberOrder = "ascending";
    let isTimerRunning = false;
    let color = "#222222";

    const numbers = [
        {
            number: 1,
            active: false,
            duration: 1500,
        },
        {
            number: 2,
            active: false,
            duration: 1500,
        },
        {
            number: 3,
            active: false,
            duration: 1500,
        },
        {
            number: 4,
            active: false,
            duration: 1500,
        },
        {
            number: 5,
            active: false,
            duration: 1500,
        },
    ];

    //when the piece starts
    $: if (start) {
        //start the timer
        startTimer();
        //set the first articulation
        if (showArticulation) setRandomArticulation();
        //set the first dynamic
        if (showDynamic) setRandomDynamic();
        //start the first number instruction, but give the player a few seconds to adjust
        setTimeout(() => {
            setRandomActive();
        }, 3000);
    }

    //pacings for when the numbers should appear
    //it builds as time goes on and slows in a cadential fashion
    $: if (time > 30 && time <= 45) {
        min = randomNumber(2000, 3000);
        max = randomNumber(4000, 4500);
    } else if (time > 45 && time <= 75) {
        min = randomNumber(1800, 2000);
        max = randomNumber(2200, 3000);
    } else if (time > 75 && time <= 220) {
        min = randomNumber(900, 1200);
        max = randomNumber(1200, 1500);
    } else if (time > 220 && time <= 250) {
        min = randomNumber(1200, 2000);
        max = randomNumber(2200, 3000);
    } else if (time > 250 && time <= 280) {
        min = randomNumber(2000, 3000);
        max = randomNumber(4000, 4500);
    } else {
        min = randomNumber(3000, 4000);
        max = randomNumber(5000, 5500);
    }

    //formatting the clock as the timer value changes
    $: {
        //minutes are the seconds divided by 60, rounded down
        minutes = Math.floor(time / 60);
        //seconds are the remainder of the seconds divided by 60, rounded down
        seconds = Math.floor(time % 60);
        //if the seconds are less than 10, add a 0 to the front
        if (seconds < 10) seconds = "0" + seconds;
    }

    //end the piece at 5 minutes
    $: if (time === 300) {
        end = true;
        stopTimer();
        setTimeout(() => {
            start = false;
            end = false;
            time = 0;
        }, 4000);
    }

    function startTimer() {
        if (!isTimerRunning) {
            isTimerRunning = true;
            updateTime();
        }
    }

    function stopTimer() {
        isTimerRunning = false;
        clearTimeout(updateTimeTimeout);
        timeouts.forEach((timeoutId) => clearTimeout(timeoutId));
        timeouts = [];
    }

    //generate a random number between min and max
    function randomNumber(min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    //update the timer every second
    function updateTime() {
        time++;
        if (isTimerRunning) {
            updateTimeTimeout = setTimeout(updateTime, 1000);
        }
    }

    //change the color of the background and rings
    //this is forwarded from the ColorPicker component
    function changeColor(event) {
        color = event.detail;
    }

    //set a random number to be active for a random duration
    function setRandomActive() {
        const timeoutId = setTimeout(() => {
            //generate a random index
            const randomIndex = randomNumber(0, numbers.length - 1);
            //generate a random duration
            const randomDuration = randomNumber(1800, 2000);
            //set the number to be active
            if (!numbers[randomIndex].active) {
                numbers[randomIndex].active = true;
                //set the duration
                numbers[randomIndex].duration = randomDuration;
                //set a timeout to set active to false after the duration
                setTimeout(() => {
                    numbers[randomIndex].active = false;
                }, randomDuration);
            }

            if (Math.random() >= 0.25) {
                setRandomArticulation();
                setRandomDynamic();
            }
            //call the function recursively
            setRandomActive();
            //the time in between each call is a random number between min and max which are paced above
        }, Math.floor(randomNumber(min, max)));

        timeouts.push(timeoutId); // Store the timeout ID
    }

    //set a random articulation
    function setRandomArticulation() {
        articulation = Math.random();
    }

    //set a random articulation
    function setRandomDynamic() {
        //the value that represents the articulation
        dynamic = Math.random();
    }
</script>

<a class="stylized position-absolute m-3 fs-4 text-decoration-none text-light" href="https://forrestbalman.com">Forrest Balman</a>
<section class="vh-100 overflow-hidden" style="background: {color}">
    {#if !end}
        <div class="container h-100 position-relative d-flex justify-content-center align-items-center text-light" transition:fade>
            <div class="d-flex flex-column align-items-center gap-2">
                {#if start}
                    <div class="d-flex flex-column align-items-center gap-3" transition:fade>
                        <div class="row flex-column flex-sm-row justify-content-center align-items-center gap-sm-2">
                            {#if numberOrder === "ascending"}
                                {#each numbers as { number, active, duration }}
                                    <Tile {number} {active} {duration} backgroundColor={color} ringColor={$colorsRing[$colorsBackground.indexOf(color)]} />
                                {/each}
                            {:else}
                                {#each [...numbers].reverse() as { number, active, duration }}
                                    <Tile {number} {active} {duration} backgroundColor={color} ringColor={$colorsRing[$colorsBackground.indexOf(color)]} />
                                {/each}
                            {/if}
                        </div>
                        <div class="d-flex justify-content-center align-items-center gap-3">
                            {#if showArticulation}
                                <h2 class="display-2 p-3 {articulation <= 0.5 ? 'fw-light' : 'fw-medium'}">
                                    {#if articulation <= 0.5}
                                        Short
                                    {:else}
                                        Long
                                    {/if}
                                </h2>
                            {/if}
                            {#if showArticulation && showDynamic}
                                <h2 class="display-2">+</h2>
                            {/if}
                            {#if showDynamic}
                                <h2 class="dynamic display-2 p-3 {dynamic <= 0.5 ? 'fw-light' : 'fw-medium'}" style="background: #f8f8f8; color: {color};">
                                    {#if dynamic <= 0.5}
                                        Quiet
                                    {:else}
                                        Loud
                                    {/if}
                                </h2>
                            {/if}
                        </div>
                    </div>
                {:else}
                    <div class="d-flex flex-column justify-content-center align-items-center">
                        <h1 class="stylized display-1">Digits</h1>
                        <div class="d-flex flex-column justify-content-center">
                            <div class="row">
                                {#each $colorsBackground as color}
                                    <ColorPicker {color} on:changeColor={changeColor} />
                                {/each}
                            </div>
                        </div>
                        <div class="d-flex flex-column gap-2 mb-3">
                            <div class="form-check d-flex align-items-center gap-2 m-0">
                                <input class="form-check-input" type="checkbox" id="showArticulation" bind:checked={showArticulation} />
                                <label class="form-check-label fs-4 text-light" for="showArticulation">Show duration</label>
                            </div>
                            <div class="form-check d-flex align-items-center gap-2 m-0">
                                <input class="form-check-input" type="checkbox" id="showDynamic" bind:checked={showDynamic} />
                                <label class="form-check-label fs-4 text-light" for="showDynamic">Show volume</label>
                            </div>
                            <label class="form-label fs-4 text-light text-center mb-0" for="numberOrder">Number order</label>
                            <select class="form-select" aria-label="Number order" bind:value={numberOrder}>
                                <option value="ascending">Ascending</option>
                                <option value="descending">Descending</option>
                            </select>
                        </div>
                        <div class="d-flex gap-2">
                            <button class="btn btn-warning fs-3" on:click={() => (start = true)}>Start</button>
                            <button class="btn btn-info fs-3" on:click={() => (modal = !modal)}>More info</button>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    {/if}
    {#if start && !end}
        <p class="fw-light position-absolute bottom-0 start-50 translate-middle-x my-3 text-light fs-3 font-monospace" transition:fade>{minutes}:{seconds}</p>
    {/if}
</section>
{#if modal}
    <aside class="min-vh-100 w-100 position-absolute top-0 start-0" style="background: {color};" transition:fade>
        <div class="container text-light">
            <button class="btn-close btn-close-white btn-lg position-absolute top-0 end-0 m-5" on:click={() => (modal = !modal)} />
            <h1 class="display-3 text-center mt-5 mb-3">How to play <span class="stylized">Digits</span></h1>
            <hr />
            <p class="fs-4"><span class="stylized">Digits</span> is a piece for keyboard 1+ hands. Although you can play this piece alone, the fun (or at least what I consider the fun) is crowding around a keyboard with what some might deem an "unreasonable" amount of people. The goal is (hopefully 😬) simple; play as many notes at the same time as you are instructed to play. No prior keyboard experience needed!</p>
            <div class="d-flex justify-content-center align-items-center mb-3">
                <video class="placeholder w-100 border border-2 rounded-2" autoplay loop muted>
                    <source src="/digits/preview.mp4" type="video/mp4" />
                </video>
            </div>
            <p class="fs-4">The score portion of the UI consists of a row (or column on smaller devices), of numbers 1 through 5. These numbers are intended to correspond with the number of keys you'll be playing. Occassionally, progress rings will appear around the numbers. Upon a ring's complete rotation, you play the instructed number of keys.</p>
            <p class="fs-4">Below the row of numbers is an instruction for how long each time you play the keys is supposed to last and the volume at which you play. There are 2 possible instructions for both duration and volume: short/long, and loud/quiet. If you see short on screen, press the keys and quickly pick your fingers up from the keyboard. If you see long, hold your fingers down until you are instructed to play another set of keys. How loud is too loud, and how quiet is too quiet is left up to the player. These instructions are enabled by default, but you can disable them by checking which options you want at the top of the page. Disabling these options can be good to get used to the pacing of the piece, at first, but as you get better at reading the score, consider adding them back!</p>
            <p class="fs-4"><span class="stylized">Digits</span> is the first piece in my <span class="fst-italics">Keyboard Community</span> series, which is a number of pieces that are all zero-barrier-to-entry works geared to getting musicially curious people to partake in the music making experience! If you are a music teacher, play the piece with your students! If you have a keyboard, and are looking for something fun to do with your friends, play the piece together!</p>
            <hr />
            <h3 class="display-4 text-center mt-5 mb-3">Frequently asked questions</h3>
            <p class="fs-3 text-decoration-underline">How do I know what keys to play?</p>
            <ul>
                <li class="fs-4">You have complete freedom over which keys you choose. Depending on how many people you're playing with, it may be more practical to limit each person's range to avoid bumping hands 🙏. But who knows? Maybe the combat over keyboard real estate 🎹 is what will make your interpretation special.</li>
            </ul>
            <p class="fs-3 text-decoration-underline">How long is the piece?</p>
            <ul>
                <li class="fs-4">The piece is programmed to last about 5 minutes, give or take a few seconds to get everyone coordinated.</li>
            </ul>
            <p class="fs-3 text-decoration-underline">Do we all play from the same score, or do we all use our own?</p>
            <ul>
                <li class="fs-4">Originally, I imagined the piece being performed with everyone having their own score, but either interpretation is fine. Is everyone playing in sync, or does the piece turn into a chaotic mess? Maybe some people are sharing a score, and others aren't. Who knows?</li>
            </ul>
            <p class="fs-3 text-decoration-underline">Do we all play from the same score, or do we all use our own?</p>
            <ul>
                <li class="fs-4">Originally, I imagined the piece being performed with everyone having their own score, but either interpretation is fine. Is everyone playing in sync, or does the piece turn into a chaotic mess. Maybe somepeople are sharing a score, and others aren't. Who knows?</li>
            </ul>
        </div>
    </aside>
{/if}

<style>
    @import url("https://fonts.googleapis.com/css2?family=Anton&family=Inconsolata:wght@200;300;400;500;600;700;800;900&display=swap");

    video {
        max-width: 500px;
    }

    .dynamic {
        background: #f8f8f8;
    }

    .stylized {
        font-family: "Anton", sans-serif;
    }

    h1, h2, p, li {
        user-select: none;
    }
</style>
