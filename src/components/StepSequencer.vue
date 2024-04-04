<template>
    <div>
        <div>
            <div class="grid m-8" v-for="(track, trackIndex) in tracks" :key="trackIndex">

                <div v-for="(step, stepIndex) in track.value" :key="stepIndex">
                    <button @click="toggleStep(stepIndex, trackIndex)"
                        :class="step.active ? 'bg-orange-500 text-black border-orange-500' : ''">
                        {{ stepIndex }}
                    </button>
                </div>
            </div>
        </div>
        <div class="m-8 space-x-4 flex flex-row">
            <button @click="togglePlayback">
                {{ playing ? 'Stop' : 'Play' }}
            </button>
            <button @click="clearPattern">
                Clear
            </button>
            <button @click="randomPattern">
                Randomise
            </button>
        </div>
        <div class="mx-8 space-x-4 flex flex-row items-baseline ">
            <span>Speed:</span>
            <input :on-change="playing = false" type="number" min="50" max="300"
                class="bg-transparent border-orange-500 border p-2" v-model="tempo" />
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted, computed } from 'vue'
import { useSound } from '@vueuse/sound'
import drumSfx from '@/assets/sounds/909-drums.mp3'

const track_1 = ref(Array(16).fill({ active: false }))
const track_2 = ref(Array(16).fill({ active: false }))
const track_3 = ref(Array(16).fill({ active: false }))
const track_4 = ref(Array(16).fill({ active: false }))

const tracks = [track_1, track_2, track_3, track_4]

const tempo = ref(250)


const tempoAsBPM = computed(() => {
    return Math.round(60000 / tempo.value);
});

const playing = ref(false)
const counter = ref(0)
let intervalId: NodeJS.Timeout | null = null

const toggleStep = (stepIndex: number, trackIndex: number) => {
    const track = tracks[trackIndex]
    track.value = track.value.map((step, i) => {
        if (i === stepIndex) {
            return { ...step, active: !step.active }
        } else {
            return step
        }
    })
}

const { play } = useSound(drumSfx, {
    sprite: {
        kick: [0, 350],
        hihat: [374, 160],
        snare: [666, 290],
        cowbell: [968, 200],
    },
})

const playSound = (id: string) => {
    play({ id })
}

const incrementCounter = () => {
    if (playing.value) {
        counter.value = (counter.value + 1) % 16 // Loop the counter between 0 and 15
        tracks.forEach((track, trackIndex) => {
            const step = track.value[counter.value]
            if (step.active) {
                // Determine the sound ID based on trackIndex
                let soundId: string
                switch (trackIndex) {
                    case 0:
                        soundId = 'kick'
                        break
                    case 1:
                        soundId = 'hihat'
                        break
                    case 2:
                        soundId = 'snare'
                        break
                    case 3:
                        soundId = 'cowbell'
                        break
                }
                playSound(soundId)
            }
        })
    }
}

const togglePlayback = () => {
    playing.value = !playing.value
    if (playing.value) {
        counter.value = 0 // Reset counter when starting playback
        intervalId = setInterval(incrementCounter, tempo.value)
    } else {
        clearInterval(intervalId as NodeJS.Timeout)
        intervalId = null
    }
}

const clearPattern = () => {
    tracks.forEach(t => {
        t.value.forEach(c => { c.active = false })
    })
}

function setTempo(newTempo) {
    return tempo.value = newTempo;
}


// TODO: fix
const randomPattern = () => {
    clearPattern() // Call clearPattern to reset the pattern

    const probability = 0.55 // Adjust the probability as needed
    tracks.forEach((track, index) => {
        track.value.forEach((cell, ci) => {
            const rand = Math.random() * (ci / 10)
            console.log((rand) < probability)
            // Check if the randomly generated number is less than the probability
            return cell.active = (rand) < probability
        });
    });
};



onUnmounted(() => {
    if (intervalId) clearInterval(intervalId)
})
</script>

<style scoped>
.grid {
    grid-template-columns: repeat(16, 1fr);
}

.grid>div {
    min-width: 50px;
    min-height: 50px;
}

button {
    @apply w-full h-full;
}
</style>