<script>
    import {flip} from "svelte/animate";
    import {tweened} from "svelte/motion";
    import { cubicOut} from "svelte/easing";
    import {fly} from "svelte/transition";

    let flags = []
    let canShuffle = false
    let opacity = tweened(1, {duration: 1000,easing:cubicOut})
    let iterations = 0
    let maxIterations = 30

    let difficulty = "normal"
    $:{
        if(difficulty === "easy"){
            maxDistance = 2
        }
        if(difficulty === "normal"){
            maxDistance = 4
        }
        if(difficulty === "hard"){
            maxDistance = 6
        }
    }

    let maxDistance = 2

    let selected = null

    let title = ""
    let didStart = false
    let correct = {name:"name"}
    let highlight = false



    function main(){
        didStart = true
        title = "Don't be confused"
        Array(15).fill(0).forEach((_, i) => {
            setTimeout(() => {
                flags = [...flags,
                    {
                        name: `Flag ${i + 1}`,
                        inverted: Math.random() < 0.5
                    }
                ]
            }, i * 100)

        })


        setTimeout(() => {
            opacity.set(0)
            correct = flags[Math.floor(Math.random() * flags.length)]
        },1500)

        setTimeout(() => {
            document.getElementById("correct").classList.add("border-4","border-green-500")
            setTimeout(() => {
                document.getElementById("correct").classList.remove("border-4","border-green-500")
                flags.forEach(flag=>{
                    flag.inverted = false
                })
            },2500)
        },2000)

        setTimeout(() => {
            canShuffle = true
            title = "FOCUS"

        },5000)
        setInterval(() => {
            if (!canShuffle) return
            flags = shuffleArray(flags,5,maxDistance)
            iterations += 1

            if (iterations >= maxIterations) {
                canShuffle = false
                document.getElementById("bg").classList.replace("bg-orange-200","bg-white")
                setTimeout(() => {
                    document.getElementById("bg").classList.add("transition-colors","duration-1000")
                    document.getElementById("bg").classList.replace("bg-white","bg-black")

                },100)

                title = ""
            }

        },500)
    }




    function shuffleArray(array, limit, maxDistance) {
        // Ensure limit is not greater than the length of the array
        limit = Math.min(limit, array.length);

        const shuffledIndices = new Set();

        for (let i = 0; i < limit; i++) {
            // Generate a random index to start shuffling from
            let randomIndex;
            do {
                randomIndex = Math.floor(Math.random() * array.length);
            } while (shuffledIndices.has(randomIndex));

            shuffledIndices.add(randomIndex);

            // Calculate the range for j based on maxDistance
            const minIndex = Math.max(0, randomIndex - maxDistance);
            const maxIndex = Math.min(array.length - 1, randomIndex + maxDistance);

            // Generate a random index within the specified range
            const j = Math.floor(Math.random() * (maxIndex - minIndex + 1)) + minIndex;

            [array[randomIndex], array[j]] = [array[j], array[randomIndex]];
        }
        return array;
    }

    function check() {
        if (selected.name === correct.name) {
            title = "CORRECT"
            document.getElementById("bg").classList.replace("bg-black","bg-green-500")
        }
        else{
            title = "INCORRECT"
            document.getElementById("bg").classList.replace("bg-black","bg-red-500")
            setTimeout(() => {
                highlight = true
            },1000)

        }
        setTimeout(() => {
            location.reload()
        },5000)
    }
</script>

<svelte:head>
    <title>Poland limbo</title>
</svelte:head>


{#if !didStart}
<div class="bg-orange-300 w-1/2 h-1/2 z-10 absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 rounded-3xl text-center" transition:fly={{y: 200, duration: 1000}}>
    <h1 class="text-8xl font-bold">LIMBO</h1>

    <h2 class="text-4xl">Select difficulty</h2>
    <div class="flex justify-center gap-10 mt-5">
        <button on:click={() => difficulty = "easy"} class="bg-green-700 p-5 text-2xl rounded-3xl text-white hover:bg-green-900 active:scale-105 transition">Easy</button>
        <button on:click={() => difficulty = "normal"} class="bg-orange-400 p-5 text-2xl rounded-3xl text-white hover:bg-orange-700 active:scale-105 transition">Normal</button>
        <button on:click={() => difficulty = "hard"} class="bg-red-500 p-5 text-2xl rounded-3xl text-white hover:bg-red-700 active:scale-105 transition">Hard</button>
    </div>
    <h2 class="text-4xl mt-5">Selected difficulty: <span class="font-bold">{difficulty}</span></h2>
    <button class="text-white bg-orange-500 text-3xl font-bold p-5 rounded-3xl mt-5 block mx-auto hover:scale-105 transition active:bg-orange-500" on:click={main}>Start</button>
</div>
{/if}
<div class="w-full h-screen bg-black">
<div class="w-full h-screen bg-orange-200 " id="bg" >
    <div class="w-full h-1/6 flex justify-center items-center">
        <h1 class="text-8xl font-bold">{title}</h1>
    </div>
    <div class="w-full h-5/6 flex justify-between flex-wrap p-10" >
        {#each flags as flag (flag.name)}
            <div class="w-1/5 h-1/5 text-center font-bold text-3xl" animate:flip={{duration: 450}}>
                <button class="w-full h-full " on:click={() => {if(iterations >= maxIterations && !selected){ selected = flag;check()}}}>
                    <img src="/Flag_of_Poland.svg" alt="poland"
                         class="w-full h-full drop-shadow-2xl transition
                            {selected && selected.name !== flag.name ? 'opacity-0' : ''}
                            {iterations >= maxIterations ? 'cursor-pointer hover:scale-110 active:scale-125' : ''}
                            {flag.inverted ? 'rotate-180' : 'rotate-0'}"


                         style="{flag.name === correct.name && highlight ? 'opacity: 1' : ''}" id="{correct.name === flag.name ? 'correct' : ''}">

                </button>
                <h2 style="opacity: {$opacity} " class="select-none">{flag.name} </h2>

            </div>

        {/each}
    </div>
</div>
</div>

<style>
    :global(body) {
        overflow: hidden;
    }
</style>
