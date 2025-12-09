<script setup>
import { ref } from 'vue'
import testInput from '~/components/TextInput.vue'
const courseArray = ['編號', '學期', '系所', '年級', '班組', '科目分類', '課程名稱', '教師', '上課人數', '學分數', '課別', '節次', '收藏'];
const weekArray = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'];
const weekResult = ref({
    星期一: false,
    星期二: false,
    星期三: false,
    星期四: false,
    星期五: false,
    星期六: false,
    星期日: false
})
const response = []
const courseCatagory = ref('')
const showChooceTime = ref(false)
</script>
<template>
    <div class="flex flex-col justify-center gap-10">
        <div
            class="flex items-center justify-center gap-20 w-full h-auto pt-4 mx-auto text-2xl text-black bg-[#d1fcdb]">
            <div class="max-w-screen flex flex-col gap-5">
                <div class="flex gap-5">
                    <testInput v-model="courseCatagory" :text="'課程分類'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput :text="'課程名稱'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput :text="'課別'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput :text="'教師'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                </div>
                <div class="flex gap-5 items-end">
                    <testInput :text="'學期'" :customWidth="'w-16 md:w-40 2xl:w-70'" :options="['1132', '1141']" />
                    <testInput :text="'系所'" :customWidth="'w-16 md:w-40 2xl:w-70'" :options="['選項1', '選項2']" />
                    <testInput :text="'年級'" :customWidth="'w-16 md:w-40 2xl:w-70'" :options="['1', '2', '3', '4']" />
                    <button class="bg-white w-16 md:w-40 2xl:w-70 h-[clamp(0px,6vmin,9999px)] border rounded-[clamp(0px,1vmin,40px)] cursor-pointer hover:bg-[#6ffc92] hover:font-bold 
                    text-xs md:text-xl 2xl:text-2xl" @click="showChooceTime = true">時間選擇</button>
                </div>
            </div>
        </div>
        <div class="flex items-center justify-center w-full">
            <button
                class="bg-[#23f157] w-96 h-16 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl">搜尋</button>
        </div>
    </div>
    <div class="mt-10 grid border h-auto  grid-cols-13 overflow-auto ">
        <div v-for="(item, index) in courseArray" :key="index"
            :class="index < 13 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
            class=" flex flex-col border border-[#146d2b] items-center justify-center">
            <div>{{ item }}</div>
        </div>
    </div>
    <!-- 時間選擇 -->
    <div v-if="showChooceTime" class=" fixed flex items-center justify-center inset-0 bg-black/50 z-40 ">
        <div
            class="flex flex-col items-center w-[clamp(0px,70vw,9999px)] h-[clamp(0px,80vh,9999px)] pt-10 gap-5 rounded-lg bg-[#d1fcdb] z-50 border">
            <div class=" cursor-pointer" @click="showChooceTime = false">
                <span class="text-3xl font-bold">時間表</span>
            </div>
            <div class="border w-full h-px"></div>
            <nav class="flex px-20 w-full justify-between overflow-auto">
                <div v-for="value in weekArray" class="text-2xl flex w-40">
                    <label class="flex items-center gap-2 cursor-pointer">
                        <input type="checkbox" class="size-6  cursor-pointer" v-model="weekResult[value]" />
                        <span>{{ value }}</span>
                    </label>
                </div>
            </nav>
            <div class="border w-full h-px"></div>
            <nav class="flex px-20 w-full justify-between overflow-auto">
                <div v-for="value in 7" class="text-2xl w-40 flex flex-col gap-4">
                    <div v-for="value in 14" class="flex flex-col">
                        <label class="flex items-center gap-2 cursor-pointer">
                            <input type="checkbox" class="size-6  cursor-pointer" v-model="weekResult[value]" />
                            <span>第{{ value }}節</span>
                        </label>
                    </div>
                </div>
            </nav>
            {{ weekResult }}
        </div>
    </div>
</template>