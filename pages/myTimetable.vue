<script setup>
import timetable from '~/components/Timetable.vue'
definePageMeta({
    middleware: function (to, from) {
        const token = useCookie('access_token')
        if (!token.value) {
            return navigateTo('/unAuth')
        }
    }
})
</script>

<!-- 還沒做RWD，且還沒有api獲取課程 -->
<template>
    <div class="w-full h-screen flex flex-col items-center pt-4 mx-auto text-2xl text-black bg-[#d1fcdb]">
        <div class="w-full flex justify-center pb-10 text-4xl font-bold">
            <span>我的課表</span>
        </div>
        <timetable />
    </div>
</template>