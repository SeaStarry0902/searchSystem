<script setup>
import timetable from '~/components/Timetable.vue'
const props = defineProps({
    customHeight: {
        type: String,
        default: ''
    }
})
const token = useCookie('access_token')
const type = useCookie('token_type')
const apiResult = ref(null)
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
// 目前可以取得該星期的課有哪些，接著在底下的switch裡再根據period去篩選時間，像是period=1，就去找有沒有end_section>=period>=start_section，
// 有的話就push進apiCourseNameArray，並且把地點也push進apiCourseLocationArray，沒有的話就只push('')進Name那個，
// 最後Name會塞進result，每次判斷是課程格(index%8!=0)且有課程(item)，就從Location取一個出來，就不會出現對不上的情況 
await getCourse()
// const monCourse = apiResult.value.filter(course => course.times.weekday === 1);
const monCourse = apiResult.value.flatMap(course => course.times).filter(t => t.weekday === 1)
const tueCourse = apiResult.value.filter(course => course.weekday === 2);
const wedCourse = apiResult.value.filter(course => course.weekday === 3);
const thuCourse = apiResult.value.filter(course => course.weekday === 4);
const friCourse = apiResult.value.filter(course => course.weekday === 5);
const satCourse = apiResult.value.filter(course => course.weekday === 6);
const sunCourse = apiResult.value.filter(course => course.weekday === 7);
console.log("星期一課程：", monCourse);
console.log("星期二課程：", tueCourse);





const weekArray = ['', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'];
const apiCourseNameArray = []
const apiCourseLocationArray = []
// 最多12個字
const totalCells = 112
const result = [...weekArray];
var period = 1
for (let i = 0; i < totalCells - weekArray.length; i++) {
    let week = i % 8;
    switch (week) {
        case 0:
            result.push(`第${period}節`);
            period++;
            continue;
        case 1:
        case 2:
        case 3:
        case 4:
        case 5:
        case 6:
        case 7:
    }

}
</script>
<template>
    <div :class="customHeight ? customHeight : 'h-[700px]'"
        class="grid border-2  border-[#146d2b] grid-cols-8 overflow-auto ">
        <div v-for="(item, index) in result" :key="index"
            :class="index < 8 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
            class=" flex flex-col border border-[#146d2b] items-center justify-center">
            <div class="text-xl text-center leading-5">{{ item }}</div>
            <span :class="index < 8 ? 'hidden' : item && index % 8 != 0 ? 'text-xs' : 'hidden'">B123</span>
        </div>
    </div>
</template>