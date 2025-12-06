<script setup>
import timetable from '~/components/Timetable.vue'
</script>

<!-- 還沒做RWD，且還沒有api獲取課程 -->
<template>
    <div class="w-full h-screen pt-4 px-96 text-2xl text-black bg-[#d1fcdb]">
        <div class="w-full pb-4 text-3xl font-bold">
            <span>我的課表</span>
        </div>
        <timetable />
    </div>
</template>