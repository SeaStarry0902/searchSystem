<script setup>
import { ref } from 'vue'
import testInput from '~/components/TextInput.vue'
const token = useCookie('access_token')
const type = useCookie('token_type')
const courseArray = ['編號', '學期', '系所', '年級', '班組', '科目分類', '課程名稱', '教師', '上課人數', '學分數', '課別', '節次', '收藏']
const weekArray = ["一", "二", "三", "四", "五", "六", "日"]
const result = []
const apiResponse = ref(null)
const keyword = ref('')
const semester = ref('')
const required_type = ref('')
const grade = ref(null)
const teacher = ref('')
const catagory = ref('')
const department = ref('')
const baseURL = `${token.value? 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses?' : 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses/public?'}`
let newURL = ''
const filters = {
    keyword,
    semester,
    required_type,
    grade,
    teacher,
    catagory,
    department
}
function buildUrl() {
    newURL = baseURL
    const params = []
    for (const item in filters) {
        if (filters[item].value) {
            params.push(`${item}=${filters[item].value}`)
        }
    }
    if (result.length != 0) {
        for (const item in result) {
            params.push(`time_slots=${result[item]}`)
        }
    }
    newURL += `${params.join('&')}&page=1&page_size=20`
    console.log(newURL)
}
const showChooceTime = ref(false)
let temDay = 1
let temSec = 1
const weekResult = ref(
    Object.fromEntries(
        weekArray.map(day => [
            day,
            Object.fromEntries(
                Array.from({ length: 14 }, (_, i) => [i + 1, false])
            )
        ])
    )
);

function toggleDay(day) {
    const checked = Object.values(weekResult.value[day]).every(v => v === true);
    const newValue = !checked;

    for (let i = 1; i <= 14; i++) {
        weekResult.value[day][i] = newValue;
    }
}
function resetAll() {
    for (const day in weekResult.value) {
        for (const sec in weekResult.value[day]) {
            weekResult.value[day][sec] = false
        }
    }
}
function saveResult() {
    result.length = 0
    temDay = 1
    for (const day in weekResult.value) {
        temSec = 1
        for (const sec in weekResult.value[day]) {
            if (weekResult.value[day][sec] === true) {
                result.push(`${temDay}-${temSec}`)
            }
            temSec++
        }
        temDay++
    }
    console.log(result)
    showChooceTime.value = false
}
async function courseSearch() {
    buildUrl()
    apiResponse.value =await $fetch(newURL, {
        headers: {
            'Authorization': type.value + ' ' + token.value,
            'Content-Type': 'application/json'
        }
    })
    console.log('回傳結果:',apiResponse.value)
}
</script>
<template>
    <!-- 篩選條件 -->
    <div class="flex flex-col justify-center gap-10">
        <div
            class="flex items-center justify-center gap-20 w-full h-auto pt-4 mx-auto text-2xl text-black bg-[#d1fcdb]">
            <div class="max-w-screen flex flex-col gap-5">
                <div class="flex gap-5">
                    <testInput v-model="keyword" :text="'課程名稱'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput v-model="required_type" :text="'課別'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput v-model="semester" :text="'學期'" :customWidth="'w-16 md:w-40 2xl:w-70'"
                        :options="['1132', '1141']" />
                    <testInput v-model="grade" :text="'年級'" :customWidth="'w-16 md:w-40 2xl:w-70'"
                        :options="[1, 2, 3, 4]" />
                </div>
                <div class="flex gap-5 items-end">

                    <testInput v-model="teacher" :text="'教師'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput v-model="catagory" :text="'課程分類'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <testInput v-model="department" :text="'系所'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                    <button class="bg-white w-16 md:w-40 2xl:w-70 h-[clamp(0px,6vmin,9999px)] border rounded-[clamp(0px,1vmin,40px)] cursor-pointer hover:bg-[#6ffc92] hover:font-bold 
                    text-xs md:text-xl 2xl:text-2xl" @click="showChooceTime = true">時間選擇</button>
                </div>
            </div>
        </div>
        <div class="flex items-center justify-center w-full">
            <button
                class="bg-[#23f157] w-96 h-16 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl"
                @click="courseSearch()">搜尋</button>
        </div>
    </div>
    <!-- 搜尋結果表格 -->
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
            class="flex flex-col items-center w-[clamp(0px,70vw,9999px)] h-[clamp(0px,80vh,9999px)] px-20 pt-10 gap-5 rounded-lg bg-[#d1fcdb] z-50 border ">
            <div class="">
                <span class="text-3xl font-bold">時間表</span>
            </div>
            <div class="flex flex-col w-full gap-5 overflow-auto ">
                <nav class="flex  w-full justify-center md:gap-5 2xl:gap-8 ">
                    <div v-for="day in weekArray" :key="day" class="md:text-base 2xl:text-xl w-44 flex">
                        <label class="flex items-center gap-2 cursor-pointer">
                            <input type="checkbox" class="md:size-4 2xl:size-6 cursor-pointer"
                                :checked="Object.values(weekResult[day]).every(v => v)" @change="toggleDay(day)">
                            <span>星期{{ day }}</span>
                        </label>
                    </div>
                </nav>
                <div class="border w-full h-px"></div>
                <nav class="flex  w-full justify-center md:gap-5 2xl:gap-8 ">
                    <div v-for="day in weekArray" :key="day" class="md:text-base 2xl:text-xl w-44 flex flex-col gap-4">
                        <div v-for="sec in 14" :key="sec" class="flex flex-col">
                            <label class="flex items-center gap-2 cursor-pointer">
                                <input type="checkbox" class="md:size-4 2xl:size-6 cursor-pointer"
                                    v-model="weekResult[day][sec]">
                                <span>第{{ sec }}節</span>
                            </label>
                        </div>
                    </div>
                </nav>
            </div>
            <div class="w-full py-8 flex justify-end gap-2">
                <button class="text-2xl text-black hover:font-bold cursor-pointer"
                    @click="showChooceTime = false"><span>取消</span></button>
                <button class="text-2xl text-black hover:font-bold cursor-pointer"
                    @click="resetAll()"><span>清除</span></button>
                <button class="text-2xl text-black hover:font-bold cursor-pointer"
                    @click="saveResult()"><span>確認</span></button>
            </div>
            <!-- <nav class="flex px-20 w-full justify-between overflow-auto">
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
            </nav> -->
        </div>
    </div>
</template>