<script setup>
definePageMeta({
    middleware: function (to, from) {
        const token = useCookie('access_token')
        if (!token.value) {
            return navigateTo('/unAuth')
        }
    }
})
import { ref, onMounted } from 'vue'
import textInput from '~/components/TextInput.vue';
onMounted(() => {
    getRole()
})
const token = useCookie('access_token')
const type = useCookie('token_type')
const role = ref('')
const baseUrl = 'https://representative-winni-chongouo-b8ca194b.koyeb.app/comments/search?page=1&page_size=20&comment_limit=5&'
const weekArray = ["一", "二", "三", "四", "五", "六", "日"]
const timeResult = []
const comments = ref([])
const inputComment = ref('')
const showChooseTime = ref(false)
const showResult = ref(false)
const showFilters = ref(true)
let params = []
let newUrl = ''
const filters = ref({
    keyword: '',
    required_type: '',
    semester: '',
    grade: '',
    teacher: '',
    catagory: '',
    department: ''
})
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
    showChooseTime.value = false
}
function buildUrl() {
    newUrl = baseUrl
    params = []
    for (const item in filters.value) {
        if (filters.value[item]) {
            params.push(`${item}=${filters.value[item]}`)
        }
    }
    if (timeResult.length != 0) {
        for (const item in timeResult) {
            params.push(`time_slots=${timeResult[item]}`)
        }
    }
    newUrl += `${params.join('&')}&page=1&page_size=100`
    console.log(newUrl)
}

async function getRole() {
    let temRole = ref('')
    try {
        temRole.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/me', {
            method: 'GET',
            headers: {
                'Authorization': type.value + ' ' + token.value
            }
        })
        role.value = temRole.value.role
    } catch (error) {

    }
}

async function getComments() {
    buildUrl()
    const res = await $fetch(newUrl, {
        method: 'GET',
        headers: {
            'Authorization': type.value + ' ' + token.value
        }
    })
    console.log('回傳', res)
    comments.value = res.items.map(item => ({
        id: item.id,
        name_zh: item.name_zh,
        comment_count: item.comment_count,
        comments: [...item.comments],
        showContent: false
    }))
    showResult.value = true
}

async function addComment(id) {
    try {
        await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/comments/${id}`,
            {
                method: 'POST',
                headers: {
                    Authorization: `${type.value} ${token.value}`,
                    'Content-Type': 'application/json'
                },
                body: {
                    content: inputComment.value
                }
            }
        )
        getComments()
        inputComment.value = ''
    } catch (error) {

    }
}
</script>
<template>
    <div class="w-full flex flex-col items-center pt-10 overflow-auto max-h-screen">

        <template v-if="showFilters">
            <span class="text-6xl">留言板</span>
            <div
                class="flex items-center justify-center gap-20 w-full h-auto pt-4 mx-auto text-2xl text-black bg-[#d1fcdb]">
                <div class="max-w-screen flex flex-col gap-5">
                    <div class="flex gap-5">
                        <textInput v-model="filters.keyword" :text="'課程名稱'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                        <textInput v-model="filters.required_type" :text="'課別'"
                            :customWidth="'w-16 md:w-40 2xl:w-70'" />
                        <textInput v-model="filters.semester" :text="'學期'" :customWidth="'w-16 md:w-40 2xl:w-70'"
                            :options="['1132', '1141']" />
                        <textInput v-model="filters.grade" :text="'年級'" :customWidth="'w-16 md:w-40 2xl:w-70'"
                            :options="[1, 2, 3, 4]" />
                    </div>
                    <div class="flex gap-5 items-end">

                        <textInput v-model="filters.teacher" :text="'教師'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                        <textInput v-model="filters.catagory" :text="'課程分類'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                        <textInput v-model="filters.department" :text="'系所'" :customWidth="'w-16 md:w-40 2xl:w-70'" />
                        <button class="bg-white w-16 md:w-40 2xl:w-70 h-[clamp(0px,6vmin,9999px)] border rounded-[clamp(0px,1vmin,40px)] cursor-pointer hover:bg-[#6ffc92] hover:font-bold 
                    text-xs md:text-xl 2xl:text-2xl" @click="showChooseTime = true">時間選擇</button>
                    </div>
                </div>
            </div>
            <button
                class="bg-[#23f157] w-96 h-16 mt-10 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl"
                @click="getComments()">
                <span>搜尋</span>
            </button>
        </template>
        <div class="flex flex-col flex-1 items-center w-[clamp(0px,80vw,9999px)] mt-10 overflow-auto" v-if="showResult">
            <div class=" w-full flex justify-end pr-4 mb-2">
                <button class="bg-[#6ffc92] w-30 h-10 border rounded cursor-pointer hover:bg-[#23f157] hover:font-bold"
                    @click="showFilters = !showFilters">{{ showFilters ? '隱藏條件' : '顯示條件' }}</button>
            </div>
            <div class="flex flex-col w-full items-center overflow-auto">
                <div class="border border-black h-px w-full "></div>
                <template v-for="course in comments" :key="course.id">
                    <div class="flex flex-col items-center w-full py-3 gap-2">
                        <div class="flex items-between justify-between w-full border-l-2 border-[#74EF93]  px-6 py-2">
                            <div class="flex gap-2 items-center">
                                <span class="text-3xl font-bold">{{ course.name_zh }}</span>
                                <div class="flex items-center gap-1">
                                    <NuxtImg src="comment_icon.svg" class="size-6" />
                                    <span class="text-xl text-gray-600">{{ course.comment_count }}</span>
                                </div>
                            </div>
                            <NuxtImg src="/arrow_icon.svg"
                                :class="course.showContent ? 'rotate-270 size-10 cursor-pointer' : 'size-10 cursor-pointer'"
                                @click="course.showContent = !course.showContent" />
                        </div>
                    </div>
                    <div class="border border-black h-px w-full"></div>
                    <div v-if="course.showContent" class="w-full flex flex-col items-center">
                        <template v-for="comment in course.comments" :key="comment.id">
                            <div v-if="course.showContent" class="w-full px-8 py-2 border-b ">
                                <span class="text-2xl">{{ comment.user_id }}: {{ comment.content }}</span>
                            </div>
                        </template>
                        <div class="flex w-full py-4 items-end gap-2">
                            <span class="text-2xl font-bold ">留言:</span>
                            <textarea v-model="inputComment"
                                class=" border w-96 bg-white rounded-2 px-2 py-2 resize-none " />
                            <button
                                class="bg-[#23f157] w-20 h-8 border rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-xl"
                                @click="addComment(course.id)">送出</button>
                        </div>
                    </div>
                </template>
            </div>
        </div>



        <!-- 時間選擇 -->
        <div v-if="showChooseTime" class=" fixed flex items-center justify-center inset-0 bg-black/50 z-40 ">
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
                        @click="showChooseTime = false"><span>取消</span></button>
                    <button class="text-2xl text-black hover:font-bold cursor-pointer"
                        @click="resetAll()"><span>清除</span></button>
                    <button class="text-2xl text-black hover:font-bold cursor-pointer"
                        @click="saveResult()"><span>確認</span></button>
                </div>
            </div>
        </div>
    </div>
</template>