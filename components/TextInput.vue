<script setup>
import { ref } from 'vue'
const props = defineProps({
    text: {
        type: String,
        default: ''
    },
    showIcon: {
        type: Boolean,
        default: false
    },
    customWidth: {
        type: String,
        default: ''
    },
    customHeight: {
        type: String,
        default: ''
    },
    options: {
        type: Array,
        default: () => []
    }
})
const selected = ref('')
const model = defineModel()
const show = props.showIcon ? ref(false) : ref(true)
</script>
<template>
    <div class="flex flex-col">
        <span class="text-xs md:text-xl 2xl:text-2xl">{{ text }}</span>
        <div v-if="options.length === 0" class="flex relative">
            <input v-model="model" :type="show ? 'text' : 'password'"
                :class="[customWidth ? customWidth : ' w-[clamp(0px,40vmin,9999px)]', '  h-[clamp(0px,6vmin,9999px)] bg-white border rounded-[clamp(0px,1vmin,40px)] pl-1 md:pl-2 2xl:pl-4 pr-7 md:pr-10 2xl:pr-14 text-xs md:text-xl 2xl:text-2xl']" />
            <NuxtImg v-if="showIcon" @click="show = !show" src="/component_img/eye_icon.svg" title="顯示/隱藏"
                alt="eye_icon"
                class=" cursor-pointer absolute size-4 md:size-5 2xl:size-7 right-2 2xl:right-5 top-1/2 -translate-y-1/2" />
        </div>
        <div v-if="options.length !== 0" class="flex relative ">
            <select v-model="model" :class="customWidth ? customWidth : ' w-[clamp(0px,40vmin,9999px)]'"
                class="cursor-pointer h-[clamp(0px,6vmin,9999px)] bg-white border rounded-[clamp(0px,1vmin,40px)] pl-1 md:pl-2 2xl:pl-4 pr-7 md:pr-10 2xl:pr-14 text-xs md:text-xl 2xl:text-2xl">
                >
                <option v-for="(item, index) in options" :key="index" :value="item" class="">{{ item }}</option>
            </select>
        </div>
    </div>
</template>