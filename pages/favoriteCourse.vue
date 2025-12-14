<script setup>
definePageMeta({
    middleware: function (to, from) {
        const token = useCookie('access_token')
        if (!token.value) {
            return navigateTo('/unAuth')
        }
    }
})
const token = useCookie('access_token')
const type = useCookie('token_type')
const apiResponse = ref(null)
let courseArray = ['編號', '學期', '系所', '年級', '課程名稱', '教師', '上課人數', '學分', '課別', '地點', '星期', '節次', '收藏']
let favoriteArray = ref([])
let courseIdArray = ref([])
const showTable = ref(false)
async function getFavorite() {
    apiResponse.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/favorites', {
        headers: {
            'Authorization': type.value + ' ' + token.value
        }
    })
    console.log(apiResponse.value)
    console.log(apiResponse.value.items.length)
}
await getFavorite()
function showResult() {
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
showResult()
// https://representative-winni-chongouo-b8ca194b.koyeb.app/favorites/221400026013A0
async function deleteFavorite(idx) {
    favoriteArray.value[idx] = false
    await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/favorites/${courseIdArray.value[idx]}`, {
        method: 'DELETE',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': type.value + ' ' + token.value
        }
    })
    alert('刪除成功!')
    window.location.reload()
}
</script>
<template>
    <div class="w-full  flex flex-col items-center pt-4 mx-auto overflow-auto ">
        <div class="w-full flex justify-center pb-10 text-4xl font-bold">
            <span>收藏課程</span>
        </div>
        <div v-if="showTable" class="mt-10 flex flex-col items-center w-full gap-5">
            <div
                class=" inline-grid h-auto grid-cols-[60px_100px_180px_40px_400px_100px_80px_40px_200px_100px_40px_100px_60px]  ">
                <div v-for="(item, index) in courseArray" :key="index"
                    :class="index < 13 ? 'h-10 bg-[#74EF93] font-bold ' : 'h-16 bg-[#EEEEEE]'"
                    class=" flex flex-col border border-[#146d2b] items-center justify-center">
                    <div :class="index > 13 && index % 13 === 12 ? 'hidden' : ''">{{ item }}</div>
                    <svg @click="deleteFavorite(Math.floor((index - 13) / 13))"
                        :class="index > 13 && index % 13 === 12 ? `${favoriteArray[Math.floor((index - 13) / 13)] ? 'text-[#ffff00] hover:text-white cursor-pointer' : 'text-white  '}` : 'hidden'"
                        xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24"
                        fill="currentColor" stroke="black" stroke-width="1" stroke-linecap="round"
                        stroke-linejoin="round">
                        <title>取消收藏</title>
                        <polygon
                            points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2">
                        </polygon>
                    </svg>
                </div>
            </div>
        </div>
    </div>

</template>