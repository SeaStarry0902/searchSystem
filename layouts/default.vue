<script setup>
import sidebarLink from '~/components/sidebarLink.vue';
const sidebarOpen = ref(false)
const isLogin = ref(false)
console.log(sidebarOpen.value)
</script>
<template>
    <div class="max-w-screen max-h-screen relative">
        <header>
            <!-- 遮罩 -->
            <div v-if="sidebarOpen" class="fixed inset-0 bg-black/50 z-40 cursor-pointer" @click="sidebarOpen = false">
            </div>

            <!-- sidebar -->
            <div v-if="sidebarOpen"
                class="absolute bg-[#ffffff] pt-10 flex flex-col border-x border-b items-start justify-start h-screen w-60 md:w-72 shadow-[0_0px_14px_rgba(0,0,0,0.25)] z-50">
                <!--hover顏色不錯 <NuxtLink to="/" class="w-full px-6 py-4 border-b hover:bg-[#E2E2E2]">首頁</NuxtLink> -->
                <sidebarLink :text="'查詢'" :link="'/event'" :image-url="'/component_img/sidebar_img/timetable_icon.svg'" />
                <sidebarLink :text="'首頁'" :link="'/'" />
            </div>

            <!-- header本體 -->
            <div
                class="flex items-center justify-between w-full h-20 md:h-32 px-6 md:px-10 2xl:px-16 gap-4 md:gap-7 2xl:gap-16 bg-[#74EF93] border-y border-[#146d2b] ">
                <svg class="hover:text-[#3867DC] size-8 md:size-10 2xl:size-12  cursor-pointer"
                    @click="sidebarOpen = !sidebarOpen" viewBox="0 0 41 31" fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <line y1="2.5" x2="41" y2="2.5" stroke="currentColor" stroke-width="5" />
                    <line y1="15.5" x2="41" y2="15.5" stroke="currentColor" stroke-width="5" />
                    <line y1="28.5" x2="41" y2="28.5" stroke="currentColor" stroke-width="5" />
                </svg>
                <span class="flex-1 text-xl md:text-4xl text-black font-bold tracking-widest">國北護課程查詢系統</span>
                <div class="flex gap-1 md:gap-4 hover:text-[#3867DC] items-center cursor-pointer">
                    <NuxtImg src="/header_img/logout_icon.svg" alt="logout_icon" class="size-7 md:size-9" />
                    <!-- click後跳回login並清空cookie，這樣不論是否login都能起效? -->
                    <span class="text-xl md:text-2xl font-bold">{{ isLogin ? '登出' : '登入' }}</span>
                </div>
            </div>

            <!-- 底下顯示user -->
            <div class="max-w-screen flex justify-end pr-5 md:pr-10 pt-4 bg-[#d1fcdb]">
                <span class="text-black text-xl md:text-2xl">歡迎</span><span
                    class="text-[#3867DC] text-xl md:text-2xl hover:underline hover:font-medium cursor-pointer hover:text-[#0031ad]">使用者</span>
            </div>
        </header>
        <slot />
    </div>
</template>