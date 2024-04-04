<template>
    <div class="flex py-4 items-center justify-center gap-4">
        <SoundButton @click="playSound('kick')" label="Kick">1</SoundButton>
        <SoundButton @click="playSound('hihat')" label="Hi-hat">2</SoundButton>
        <SoundButton @click="playSound('snare')" label="Snare">3</SoundButton>
        <SoundButton @click="playSound('cowbell')" label="Cowbell">4</SoundButton>
    </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import { useSound } from '@vueuse/sound'
import drumSfx from '@/assets/sounds/909-drums.mp3'
import SoundButton from '@/components/SoundButton.vue'

const useKeyboardBindings = (map) => {
    const handlePress = (ev) => {
        const handler = map[ev.key]
        if (typeof handler === 'function') {
            handler()
        }
    }

    onMounted(() => {
        window.addEventListener('keydown', handlePress)
    })

    onUnmounted(() => {
        window.removeEventListener('keydown', handlePress)
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

const playSound = (id) => {
    play({ id })
}

// useKeyboardBindings({
//     '1': () => playSound('kick'),
//     '2': () => playSound('hihat'),
//     '3': () => playSound('snare'),
//     '4': () => playSound('cowbell'),
// })

</script>