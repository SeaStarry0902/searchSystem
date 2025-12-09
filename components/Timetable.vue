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
// const monCourse = apiResult.value.flatMap(course => course.times).filter(t => t.weekday === 1)
const monCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 1);
const tueCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 2);
const wedCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 3);
const thuCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 4);
const friCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 5);
const satCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 6);
const sunCourse = apiResult.value.flatMap(course => course.times.map(t => ({ ...course, ...t }))).filter(c => c.weekday === 7);
// const tueCourse = apiResult.value.filter(course => course.weekday === 2);
// const wedCourse = apiResult.value.filter(course => course.weekday === 3);
// const thuCourse = apiResult.value.filter(course => course.weekday === 4);
// const friCourse = apiResult.value.filter(course => course.weekday === 5);
// const satCourse = apiResult.value.filter(course => course.weekday === 6);
// const sunCourse = apiResult.value.filter(course => course.weekday === 7);
console.log("星期一課程：", monCourse);
console.log("星期二課程：", tueCourse);





const weekArray = ['', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'];
const apiCourseNameArray = []
const apiCourseLocationArray = ['', '', '', '', '', '', '', '']
// 最多12個字
const totalCells = 112
var period = 0
for (let i = 0; i < totalCells - weekArray.length; i++) {
    let week = i % 8;
    switch (week) {
        case 0:
            period++;
            apiCourseNameArray.push(`第${period}節`);
            apiCourseLocationArray.push('');
            continue;
        case 1:
            const monCoursesThisPeriod = monCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (monCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(monCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(monCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 2:
            const tueCoursesThisPeriod = tueCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (tueCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(tueCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(tueCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 3:
            const wedCoursesThisPeriod = wedCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (wedCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(wedCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(wedCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 4:
            const thuCoursesThisPeriod = thuCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (thuCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(thuCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(thuCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 5:
            const friCoursesThisPeriod = friCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (friCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(friCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(friCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 6:
            const satCoursesThisPeriod = satCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (satCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(satCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(satCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
        case 7:
            const sunCoursesThisPeriod = sunCourse.filter(course => course.start_section <= period && course.end_section >= period);
            if (sunCoursesThisPeriod.length > 0) {
                apiCourseNameArray.push(sunCoursesThisPeriod[0]?.name_zh || '');
                apiCourseLocationArray.push(sunCoursesThisPeriod[0]?.classroom || '');
            } else {
                apiCourseNameArray.push('');
                apiCourseLocationArray.push('');
            }
            continue;
    }

}

const result = [...weekArray, ...apiCourseNameArray];
</script>
<template>
    <div
        :class="[customHeight ? customHeight : 'h-[clamp(0px,70vh,9999px)]', ' w-[clamp(0px,70vw,9999px)] grid border-2 border-[#146d2b] grid-cols-8 overflow-auto']">
        <!-- <div :class="[customHeight ? customHeight : 'h-[700px]','grid border-2 border-[#146d2b] grid-cols-8 overflow-auto ']"
        class="grid border-2 border-[#146d2b] grid-cols-8 overflow-auto "> -->
        <div v-for="(item, index) in result" :key="index"
            :class="index < 8 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
            class=" flex flex-col border border-[#146d2b] items-center justify-center">
            <div class="text-xl text-center leading-5">{{ item }}</div>
            <!-- <span :class="index < 8 ? 'hidden' : item && index % 8 != 0 ? 'text-xs' : 'hidden'">B123</span> -->
            <span class="text-base text-[#3867DC] font-bold">{{ apiCourseLocationArray[index] }}</span>
        </div>
    </div>
</template>