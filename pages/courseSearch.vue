<script setup>
import { ref } from 'vue'
import testInput from '~/components/TextInput.vue'
let params = []
let newURL = ''
let temDay = 1
let temSec = 1
const token = useCookie('access_token')
const type = useCookie('token_type')
let courseArray = ['編號', '學期', '系所', '年級', '課程名稱', '教師', '上課人數', '學分', '課別', '地點', '星期', '節次', '收藏']
const weekArray = ["一", "二", "三", "四", "五", "六", "日"]
const result = []
let favoriteArray = ref([])
let courseIdArray = ref([])
const apiResponse = ref(null)
const keyword = ref('')
const semester = ref('')
const required_type = ref('')
const grade = ref(null)
const teacher = ref('')
const catagory = ref('')
const department = ref('')
const showTable = ref(false)
const showFilters = ref(true)
const excelBlob = ref(null)
const showChooceTime = ref(false)
const baseExcelURL = 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses/export'
const baseURL = `${token.value ? 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses?' : 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses/public?'}`
const weekResult = ref(
    Object.fromEntries(
        weekArray.map(day => [
            day,
            Object.fromEntries(
                Array.from({ length: 14 }, (_, i) => [i + 1, false])
            )
        ])
    )
)
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
    params = []
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
    newURL += `${params.join('&')}&page=1&page_size=100`
    console.log('這是查詢用的' + newURL)
}
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
function showResult() {
    console.log("showResult進入")
    showTable.value = false
    let idx = 1
    let lastId = ''
    courseArray = ['編號', '學期', '系所', '年級', '課程名稱', '教師', '上課人數', '學分', '課別', '地點', '星期', '節次', '收藏']
    favoriteArray.value = []
    courseIdArray.value = []
    for (let key = 0; key < apiResponse.value.items.length; key++) {
        if (apiResponse.value.items[key].id === lastId)
            continue
        const temData = apiResponse.value.items[key]
        courseArray.push(idx)
        courseArray.push(temData.semester)
        courseArray.push(temData.department_name)
        courseArray.push(temData.grade)
        courseArray.push(temData.name_zh)
        courseArray.push(temData.teacher_name)
        courseArray.push(temData.limit_max)
        courseArray.push(temData.credit)
        courseArray.push(temData.required_type)
        courseArray.push(temData.times[0].classroom)
        courseArray.push(temData.times[0].weekday)
        courseArray.push(`${temData.times[0].start_section === temData.times[0].end_section ? temData.times[0].start_section : temData.times[0].start_section + '~' + temData.times[0].end_section}節`)
        courseArray.push(temData.is_favorite)
        favoriteArray.value.push(temData.is_favorite)
        courseIdArray.value.push(temData.id)
        idx++
        lastId = temData.id
    }
    console.log("showResult離開")
    console.log(favoriteArray.value)
    showTable.value = true
}
async function downloadExcel() {
    newURL = baseExcelURL
    if (params.length !== 0)
        newURL += `?${params.join('&')}`
    console.log('這是excel的' + newURL)
    excelBlob.value = await $fetch(newURL, {
        responseType: 'blob',
        headers: {
            'Content-Type': 'application/json'
        }
    })
    const url = URL.createObjectURL(excelBlob.value)
    const a = document.createElement('a')
    a.href = url
    a.download = '課程資料.xlsx'
    a.click()
    URL.revokeObjectURL(url)
}
async function courseSearch() {
    buildUrl()
    apiResponse.value = await $fetch(newURL, {
        headers: {
            'Authorization': type.value + ' ' + token.value,
            'Content-Type': 'application/json'
        }
    })
    console.log('回傳結果:', apiResponse.value)
    showFilters.value = false
    showResult()
}
async function addFavorite(idx) {
    if (!token.value)
        return
    favoriteArray.value[idx] = true
    await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/favorites/${courseIdArray.value[idx]}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': type.value + ' ' + token.value
        }
    })
    alert('收藏成功!')
}
</script>
<template>
    <!-- 篩選條件 -->
    <div v-if="showFilters" class="flex flex-col items-center justify-center pt-10 ">
        <span class="text-6xl">課程查詢</span>
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
        <div class="flex items-center justify-center w-full pt-10">
            <button
                class="bg-[#23f157] w-96 h-16 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl"
                @click="courseSearch()">搜尋</button>
        </div>
    </div>
    <!-- 搜尋結果表格 -->
    <div v-if="showTable" class="relative mt-10 flex flex-col items-center overflow-auto w-full gap-5">
        <div
            class="flex gap-2 items-center justify-end w-[calc(60px+100px+180px+40px+400px+100px+80px+40px+200px+100px+40px+100px+60px)]">
            <button class="bg-[#6ffc92] w-30 h-10 border rounded cursor-pointer hover:bg-[#23f157] hover:font-bold"
                @click="showFilters = !showFilters">{{ showFilters ? '隱藏條件' : '顯示條件' }}</button>
            <button class="bg-[#6ffc92] w-30 h-10 border rounded cursor-pointer hover:bg-[#23f157] hover:font-bold"
                @click="downloadExcel()">匯出</button>
        </div>
        <div
            class=" inline-grid h-auto grid-cols-[60px_100px_180px_40px_400px_100px_80px_40px_200px_100px_40px_100px_60px]  ">
            <div v-for="(item, index) in courseArray" :key="index"
                :class="index < 13 ? 'h-10 bg-[#74EF93] font-bold ' : 'h-16 bg-[#EEEEEE]'"
                class=" flex flex-col border border-[#146d2b] items-center justify-center">
                <div :class="index > 13 && index % 13 === 12 ? 'hidden' : ''">{{ item }}</div>
                <svg @click="addFavorite(Math.floor((index - 13) / 13))"
                    :class="index > 13 && index % 13 === 12 ? `${favoriteArray[Math.floor((index - 13) / 13)] ? 'text-[#ffff00]' : 'text-white hover:text-[#ffff00] cursor-pointer'}` : 'hidden'"
                    xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24" fill="currentColor"
                    stroke="black" stroke-width="1" stroke-linecap="round" stroke-linejoin="round">
                    <title>收藏課程</title>
                    <polygon
                        points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2">
                    </polygon>
                </svg>
            </div>
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
        </div>
    </div>
</template>