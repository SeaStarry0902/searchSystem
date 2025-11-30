<script setup>
const props = defineProps({
    text: {
        type: String,
        default: ''
    },
    showIcon: {
        type: Boolean,
        default: false
    }
})
const show = props.showIcon ? ref(false) : ref(true)
</script>
<template>
    <div class="flex flex-col">
        <span class="text-xs md:text-xl 2xl:text-2xl">{{ text }}</span>
        <div class="flex relative">
            <input :type="show ? 'text' : 'password'"
                class=" w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] bg-white border rounded-[clamp(0px,1vmin,40px)] pl-1 md:pl-2 2xl:pl-4 pr-7 md:pr-10 2xl:pr-14 text-xs md:text-xl 2xl:text-2xl" />
            <NuxtImg v-if="showIcon" @click="show = !show" src="/component_img/eye_icon.svg" title="顯示/隱藏"
                alt="eye_icon"
                class=" cursor-pointer absolute size-4 md:size-5 2xl:size-7 right-2 2xl:right-5 top-1/2 -translate-y-1/2" />
        </div>

    </div>
</template>