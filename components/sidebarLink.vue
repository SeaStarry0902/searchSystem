<script setup>
const props = defineProps({
    text: {
        type: String,
        default: ''
    },
    link: {
        type: String,
        default: '/'
    },
    imageUrl: {
        type: String,
        default: ''
    }
})   
</script>
<template>
    <div class="w-full px-2">
        <NuxtLink :to="link"
            class="flex w-full h-auto items-center rounded-2xl px-4 py-2 md:py-4 gap-1 md:gap-2 hover:bg-[#E2E2E2] ">
            <NuxtImg v-if="imageUrl" :src="imageUrl" alt="search_icon" class="size-5 md:size-6" />
            <span class="text-xl md:text-2xl">{{ text }}</span>
        </NuxtLink>
    </div>
</template>