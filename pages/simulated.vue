<script setup>
import timetable from '~/components/Timetable.vue'
import { ref } from 'vue'
const token = useCookie('access_token')
const type = useCookie('token_type')
const courseArray = ['', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日']
const weekArray = ["一", "二", "三", "四", "五", "六", "日"]
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
let params = []
let newURL = ''
let apiResponse = ref(null)
const baseURL = `${token.value ? 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses?' : 'https://representative-winni-chongouo-b8ca194b.koyeb.app/courses/public?'}`
const keyword = ref('')
const semester = ref('')
const required_type = ref('')
const grade = ref('')
const teacher = ref('')
const catagory = ref('')
const department = ref('')
const weekMap = {
    1: '一',
    2: '二',
    3: '三',
    4: '四',
    5: '五',
    6: '六',
    7: '日'
}
const filters = {
    keyword,
    semester,
    required_type,
    grade,
    teacher,
    catagory,
    department
}
const timeResult = []
const apiResult = ref(null)
let courseList = ref({
  items: []
})
const showResult = ref(true)
const selectCourse = ref(false)
const showChooceTime = ref(false)
let result = ref([...courseArray])
let simulatedCourse = ref([])
let apiCourseLocationArray = ref(['', '', '', '', '', '', '', ''])
let simulatedLocation = ref([])
const totalCells = 120
let period = 0
for (let i = 0; i < totalCells - courseArray.length; i++) {
    let week = i % 8
    if (week === 0) {
        period++
        result.value.push(`第${period}節`)
    }
    else
        result.value.push('')
}
simulatedCourse.value = [...result.value]
simulatedLocation.value = [...apiCourseLocationArray.value]
function buildUrl() {
    newURL = baseURL
    params = []
    for (const item in filters) {
        if (filters[item].value) {
            params.push(`${item}=${filters[item].value}`)
        }
    }
    if (timeResult.length != 0) {
        for (const item in timeResult) {
            params.push(`time_slots=${timeResult[item]}`)
        }
    }
    newURL += `${params.join('&')}&page=1&page_size=100`
    console.log('這是查詢用的' + newURL)
}
async function getFavorite() {
    courseList.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/favorites', {
        headers: {
            'Authorization': type.value + ' ' + token.value
        }
    })
    console.log('收藏課程', courseList.value)
}
async function getCourse() {
    apiResult.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/students/me/timetable?semester=1141&status=completed', {
        metgod: 'GET',
        headers: {
            'Authorization': type.value + ' ' + token.value,
            'Content-Type': 'application/json'
        }
    }).catch((error) => {
        console.log(error)
    })
    console.log("課表資料：", apiResult.value)
}
if (token.value) {
    await getCourse()
    await getFavorite()
    const monCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 1);
    const tueCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 2);
    const wedCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 3);
    const thuCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 4);
    const friCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 5);
    const satCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 6);
    const sunCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 7);
    const apiCourseNameArray = []
    apiCourseLocationArray.value = ['', '', '', '', '', '', '', '']
    period = 0
    for (let i = 0; i < totalCells - courseArray.length; i++) {
        let week = i % 8;
        switch (week) {
            case 0:
                period++;
                apiCourseNameArray.push(`第${period}節`);
                apiCourseLocationArray.value.push('');
                continue;
            case 1:
                const monCoursesThisPeriod = monCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (monCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(monCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(monCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 2:
                const tueCoursesThisPeriod = tueCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (tueCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(tueCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(tueCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 3:
                const wedCoursesThisPeriod = wedCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (wedCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(wedCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(wedCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 4:
                const thuCoursesThisPeriod = thuCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (thuCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(thuCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(thuCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 5:
                const friCoursesThisPeriod = friCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (friCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(friCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(friCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 6:
                const satCoursesThisPeriod = satCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (satCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(satCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(satCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
            case 7:
                const sunCoursesThisPeriod = sunCourse.filter(course => course.start_section <= period && course.end_section >= period);
                if (sunCoursesThisPeriod.length > 0) {
                    apiCourseNameArray.push(sunCoursesThisPeriod[0]?.name_zh || '');
                    apiCourseLocationArray.value.push(sunCoursesThisPeriod[0]?.classroom || '');
                } else {
                    apiCourseNameArray.push('');
                    apiCourseLocationArray.value.push('');
                }
                continue;
        }

    }
    result.value = [...courseArray, ...apiCourseNameArray]
    simulatedCourse.value = [...result.value]
    simulatedLocation.value = [...apiCourseLocationArray.value]
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
    timeResult.length = 0
    let temDay = 1
    let temSec = 1
    for (const day in weekResult.value) {
        temSec = 1
        for (const sec in weekResult.value[day]) {
            if (weekResult.value[day][sec] === true) {
                timeResult.push(`${temDay}-${temSec}`)
            }
            temSec++
        }
        temDay++
    }
    console.log(timeResult)
    showChooceTime.value = false
}
async function courseSearch() {
    buildUrl()
    if (semester.value === '') {
        alert('請選擇學期')
        return
    }
    apiResponse.value = await $fetch(newURL, {
        headers: {
            'Authorization': type.value + ' ' + token.value,
            'Content-Type': 'application/json'
        }
    })
    courseList.value = apiResponse.value
    console.log('回傳結果:', courseList.value)
}
function addCourse(course) {
    let day = course.times[0].weekday
    let sec = course.times[0].start_section
    let duration = course.times[0].end_section - course.times[0].start_section
    for (let i = 0; i <= duration; i++) {
        let curIndex = day + (sec + i) * 8
        if (simulatedCourse.value[curIndex] !== '') {
            alert('課程衝堂，無法加入')
            return
        }
    }
    for (let i = 0; i <= duration; i++) {
        let curIndex = day + (sec + i) * 8
        simulatedCourse.value[curIndex] = course.name_zh
        simulatedLocation.value[curIndex] = course.times[0].classroom
    }
    console.log('加入課程', course)
    console.log('目前模擬課表', simulatedCourse.value)
    console.log('地點陣列', simulatedLocation.value)
}
function removeAllCourse() {
    simulatedCourse.value = [...result.value]
    simulatedLocation.value = [...apiCourseLocationArray.value]
}
function changeTimetable() {
    result.value = [...simulatedCourse.value]
    apiCourseLocationArray.value = [...simulatedLocation.value]
    console.log('確認模擬課表')
    showResult.value = true
}
</script>
<template>
    <div v-if="showResult"
        class="w-full h-screen flex flex-col items-center pt-4 mx-auto text-2xl text-black bg-[#d1fcdb]">
        <div class="w-[clamp(0px,70vw,9999px)] flex justify-start items-end pb-10 gap-2">
            <span class="text-4xl font-bold">預選課功能</span>
            <button class="cursor-pointer hover:font-bold text-[#3867DC] text-xl"
                @click="showResult = false; selectCourse = true">點我進入</button>
        </div>
        <div
            class="h-[clamp(0px,70vh,9999px)] w-[clamp(0px,70vw,9999px)] grid border-2 border-[#146d2b] grid-cols-8 overflow-auto">
            <div v-for="(item, index) in result" :key="index"
                :class="index < 8 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
                class=" flex flex-col border border-[#146d2b] items-center justify-center">
                <div class="text-xl text-center leading-5">{{ item }}</div>
                <span class="text-base text-[#3867DC] font-bold">{{ apiCourseLocationArray[index] }}</span>
            </div>
        </div>
    </div>


    <div v-if="!showResult && selectCourse" class="w-full flex items-center justify-center pt-10 ">
        <div
            class="bg-white w-[1000px] h-[clamp(0px,75vh,9999px)] flex flex-col items-center justify-start rounded-lg border overflow-auto">
            <div class="flex flex-col w-full px-10 py-2 gap-4 overflow-auto">
                <div class="flex  justify-between text-xl">
                    <input v-model="keyword" class="border-b border-black outline-none w-24" type="text"
                        placeholder="課程名稱" />
                    <input v-model="required_type" class="border-b border-black outline-none w-24" type="text"
                        placeholder="課別" />
                    <select v-model="semester" class="border-b border-black outline-none w-24">
                        <option value="" hidden>學期</option>
                        <option value="1132">1132</option>
                        <option value="1141">1141</option>
                    </select>
                    <select v-model="grade" class="border-b border-black outline-none w-24">
                        <option value="" hidden>年級</option>
                        <option value="1">1</option>
                        <option value="2">2</option>
                        <option value="3">3</option>
                        <option value="4">4</option>
                    </select>
                    <input v-model="teacher" class="border-b border-black outline-none w-24" type="text"
                        placeholder="教師" />
                    <input v-model="catagory" class="border-b border-black outline-none w-24" type="text"
                        placeholder="課程分類" />
                    <input v-model="department" class="border-b border-black outline-none w-24" type="text"
                        placeholder="系所" />
                    <button class="border w-30 rounded-lg text-xl cursor-pointer hover:font-bold hover:bg-[#6ffc92]"
                        @click="showChooceTime = true">時間選擇</button>
                </div>
                <div class="flex w-full justify-center ">
                    <button
                        class="bg-[#23f157] w-96 h-10 border rounded-lg cursor-pointer hover:bg-[#18a83c] hover:font-bold text-xl"
                        @click="courseSearch()">查詢
                    </button>
                </div>
            </div>

            <div class="border border-black w-full h-px "></div>

            <div class="flex flex-1 w-full overflow-auto ">
                <div class=" w-[800px] border-r grid  grid-cols-8 overflow-auto">
                    <div v-for="(item, index) in simulatedCourse" :key="index"
                        :class="index < 8 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
                        class=" flex flex-col border border-[#146d2b] items-center justify-center">
                        <div class="text-xl text-center leading-5">{{ item }}</div>
                        <span class="text-base text-[#3867DC] font-bold">{{ simulatedLocation[index] }}</span>
                    </div>
                </div>


                <div class="flex flex-col flex-1">
                    <div class=" overflow-auto">
                        <div v-for="(item, idx) in courseList.items" class="h-24 w-full flex border-b border-r">
                            <div class="flex-col flex justify-center items-center gap-1 w-28 ">
                                <div class="flex items-center gap-1">
                                    <span class="text-base font-bold">{{ item.name_zh }}</span>
                                    <span class="text-xs text-[#9C9696]">{{ item.times[0].classroom }}</span>
                                </div>
                                <div class="flex">
                                    <span>星期{{ weekMap[item.times[0].weekday] }}{{ item.times[0].start_section ===
                                        item.times[0].end_section ? `第${item.times[0].start_section}` :
                                        `${item.times[0].start_section}~${item.times[0].end_section}` }}節</span>
                                </div>
                            </div>
                            <div class="flex flex-1 items-center justify-center">
                                <button class="text-[#3867DC] cursor-pointer hover:font-bold"
                                    @click="addCourse(item)">加入</button>
                            </div>
                        </div>
                    </div>

                    <div class="flex flex-1 items-end justify-between">
                        <button class="w-full border-t  hover:bg-[#23f157] hover:font-bold cursor-pointer"
                            @click="showResult = true">取消</button>
                        <button class="w-full border-t border-x hover:bg-[#23f157] hover:font-bold cursor-pointer"
                            @click="removeAllCourse()">清除</button>
                        <button class="w-full border-t  hover:bg-[#23f157] hover:font-bold cursor-pointer"
                            @click="changeTimetable()">確認</button>
                    </div>

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
                        <div v-for="day in weekArray" :key="day"
                            class="md:text-base 2xl:text-xl w-44 flex flex-col gap-4">
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
    </div>
</template>