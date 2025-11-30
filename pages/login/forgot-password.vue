<script setup>
import textInput from '~/components/TextInput.vue'


definePageMeta({
    layout: 'custom'
})
</script>
<template>
    <div
        class="relative w-full flex flex-col justify-center items-center mt-[clamp(0px,4vmin,9999px)] gap-[clamp(0px,4vmin,9999px)]">
        <div class=" flex flex-col gap-[clamp(0px,0.5vmin,9999px)]">
            <textInput :text="'帳號'" />
            <textInput :text="'舊密碼'" :showIcon="true" />
            <textInput :text="'新密碼'" :showIcon="true" />
            <textInput :text="'確認密碼'" :showIcon="true" />
        </div>
        <NuxtLink to="/login"
            class="flex  hover:font-bold items-center justify-center hover:bg-[#069736] w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] border border-black cursor-pointer bg-[#09ca49] rounded-[clamp(0px,1vmin,40px)] text-white text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl">註冊</span>
        </NuxtLink>
        <NuxtLink to="/login"
            class="absolute top-0 -left-10 md:-left-20 text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl text-[#3867DC] hover:font-bold ">返回</span>
        </NuxtLink>
    </div>
</template>