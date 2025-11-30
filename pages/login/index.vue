<script setup>
import textInput from '~/components/TextInput.vue'


definePageMeta({
    layout: 'custom'
})
</script>
<template>
    <div class="flex flex-col justify-center items-center mt-[clamp(0px,5vmin,9999px)] gap-[clamp(0px,3vmin,9999px)]">
        <div class="flex flex-col gap-[clamp(0px,3vmin,9999px)]">
            <textInput :text="'學號'" />
            <textInput :text="'密碼'" :showIcon="true" />
        </div>
        <div class="flex items-center justify-center">
            <label class="flex items-center justify-center gap-1 md:gap-2 cursor-pointer">
                <input type="checkbox" class="size-3 md:size-5 cursor-pointer" />
                <span class="text-xs md:text-xl">記住帳號</span>
            </label>
        </div>
        <button
            class=" hover:font-bold hover:bg-[#069736] w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] border border-black cursor-pointer bg-[#09ca49] rounded-[clamp(0px,1vmin,40px)] text-white text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl ">登入</span>
        </button>
        <nav class="flex items-center justify-center gap-[clamp(0px,4vmin,9999px)]">
            <NuxtLink to="/login/signup" class="text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">註冊</NuxtLink>
            <NuxtLink to="/login/forgot-password" class="text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">忘記密碼</NuxtLink>
        </nav>
        <NuxtLink to="#" class="md:pt-2 2xl:pt-4 text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">訪客登入</NuxtLink>
    </div>
</template>