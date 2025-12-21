<script setup>
import textInput from '~/components/TextInput.vue';
definePageMeta({
    middleware: async () => {
        const token = useCookie('access_token')
        const type = useCookie('token_type')

        try {
            const role = await $fetch(
                'https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/me',
                {
                    headers: {
                        Authorization: `${type.value} ${token.value}`
                    }
                }
            )

            if (role.role !== 'admin') {
                return navigateTo('/unAuth')
            }
        } catch {
            return navigateTo('/login')
        }
    }
})
import { ref } from 'vue'
import testInput from '~/components/TextInput.vue'
let params = []
let newURL = ''
let temDay = 1
let temSec = 1
const token = useCookie('access_token')
const type = useCookie('token_type')
const fileInput = ref(null)
let courseArray = ['編號', '學期', '系所', '年級', '課程名稱', '教師', '上課人數', '學分', '課別', '地點', '星期', '節次', '操作']
const weekArray = ["一", "二", "三", "四", "五", "六", "日"]
const result = []
let createCourse = ref(false)
let courses = ref()
let deleteCheck = ref(false)
let selectedCourseId = ref(null)
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
const createFilter = ref({
    id: '',
    name_zh: '',
    name_en: '',
    semester: '',
    grade: null,
    credit: null,
    class_group: '',
    group_code: '',
    required_type: '',
    category: '',
    limit_min: null,
    limit_max: null,
    department_name: '',
    teacher_name: '',
    chinese_summary: '',
    english_summary: '',
    raw_remark: '',
    times: [
        {
            weekday: null,
            classroom: '',
            start_section: null,
            end_section: null
        }
    ]
})
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
    showChooceTime.value = false
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
    if (!apiResponse.value || apiResponse.value.items.length === 0) {
        alert('查無資料，請更改搜尋條件後再試一次。')
        showTable.value = false
        return
    }
    else {
        courses.value = apiResponse.value.items.map(item => ({
            id: item.id || '',
            semester: item.semester || '',
            department_name: item.department_name || '',
            grade: item.grade || '',
            name_zh: item.name_zh || '',
            teacher_name: item.teacher_name || '',
            limit_max: item.limit_max || '',
            credit: item.credit || '',
            required_type: item.required_type || '',
            location: item.times[0].classroom || '',
            weekday: item.times[0].weekday || '',
            start_section: item.times[0].start_section || '',
            end_section: item.times[0].end_section || '',
            editing: false,
            _backup: null
        }))
        showTable.value = true
    }
    console.log('課程列表：', courses.value)
}
function startEdit(course) {
    course._backup = { ...course }
    course.editing = true
}

function cancelEdit(course) {
    Object.assign(course, course._backup)
    course.editing = false
}
async function saveCourse(course) {
    console.log('儲存課程：', course)
    await $fetch(
        `https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/courses/${course.id}`,
        {
            method: 'PUT',
            headers: {
                Authorization: `${type.value} ${token.value}`,
                'Content-Type': 'application/json'
            },
            body: {
                name_zh: course.name_zh || '',
                semester: course.semester || '',
                department_name: course.department_name,
                teacher_name: course.teacher_name,
                grade: course.grade || 0,
                credit: course.credit || 0,
                required_type: course.required_type || '',
                limit_max: course.limit_max || 0,
                times: [
                    {
                        weekday: course.weekday,
                        start_section: course.start_section,
                        end_section: course.end_section,
                        classroom: course.location || ''
                    }
                ]
            }
        }
    )
    course.editing = false
}
async function deleteCourse() {
    await $fetch(
        `https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/courses/${selectedCourseId.value}`,
        {
            method: 'DELETE',
            headers: {
                Authorization: `${type.value} ${token.value}`
            }
        }
    )

    deleteCheck.value = false
    selectedCourseId.value = null
    courseSearch()
}
async function createNewCourse() {
    console.log('建立課程：', createFilter.value)
    await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/courses`,
        {
            method: 'POST',
            headers: {
                Authorization: `${type.value} ${token.value}`,
                'Content-Type': 'application/json'
            },
            body: createFilter.value
        }
    )
    createCourse.value = false
}

async function importCourse(event) {
    const file = event.target.files[0]
    const allowed = 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
    const formData = new FormData()
    formData.append('file', file, file.name)
    if (!allowed.includes(file.type)) {
        alert('只能上傳 .xlsx 格式')
        return
    }
    try {
        await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/import`,
            {
                method: 'POST',
                headers: {
                    Authorization: `${type.value} ${token.value}`
                },
                body: formData
            }       
        )
        alert('匯入成功!')
        courseSearch()
    } catch (error) {

    }
}
</script>
<template>
    <!-- 篩選條件 -->
    <div v-if="showFilters" class="flex flex-col items-center pt-10 justify-center ">
        <span class="text-6xl">課程管理</span>
        <div
            class="flex items-center justify-center gap-20 pt-4 w-full h-auto mx-auto text-2xl text-black bg-[#d1fcdb]">
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
        <div class="flex items-center justify-center w-full pt-10 gap-8">
            <button
                class="bg-[#23f157] w-96 h-16 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl"
                @click="courseSearch()">
                <span>搜尋</span>
            </button>
        </div>
    </div>
    <!-- 搜尋結果表格 -->
    <div v-if="showTable" class="mt-10 flex flex-col items-center overflow-auto w-full gap-5">
        <div class="flex gap-2 items-end justify-end w-[1520px]">
            <button class="bg-[#6ffc92] w-30 h-10 border rounded cursor-pointer hover:bg-[#23f157] hover:font-bold"
                @click="showFilters = !showFilters">{{ showFilters ? '隱藏條件' : '顯示條件' }}</button>
            <input type="file" ref="fileInput" accept=".xlsx" @change="importCourse"
                style="display: none" />
            <button class="bg-[#6ffc92] w-30 h-10 border rounded cursor-pointer hover:bg-[#23f157] hover:font-bold"
                @click="fileInput.click()">匯入</button>
        </div>
        <div
            class=" inline-grid h-auto grid-cols-[40px_80px_140px_80px_320px_80px_80px_60px_180px_100px_60px_140px_160px]  ">
            <div v-for="item in courseArray"
                class="h-10 bg-[#74EF93] font-bold flex flex-col border border-[#146d2b] items-center justify-center">
                {{ item }}
            </div>
            <template v-for="(course, idx) in courses" :key="course.id">
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    {{ idx + 1 }}
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.semester }}</span>
                    <select v-else v-model="course.semester" class="border w-full px-1 bg-white">
                        <option value="1132">1132</option>
                        <option value="1141">1141</option>
                    </select>
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.department_name }}</span>
                    <input v-else v-model="course.department_name" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.grade }}</span>
                    <select v-else v-model="course.grade" class="border w-full px-1 bg-white">
                        <option v-for="n in 4" :key="n" :value="n">{{ n }}</option>
                    </select>
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.name_zh }}</span>
                    <input v-else v-model="course.name_zh" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.teacher_name }}</span>
                    <input v-else v-model="course.teacher_name" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.limit_max }}</span>
                    <input v-else v-model="course.limit_max" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.credit }}</span>
                    <!-- <input v-else v-model="course.credit" class="border w-full px-1 bg-white" /> -->
                    <select v-else v-model="course.credit" class="border w-full px-1 bg-white">
                        <option value="0">0</option>
                        <option v-for="n in 3" :key="n" :value="n">{{ n }}</option>
                    </select>
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.required_type }}</span>
                    <input v-else v-model="course.required_type" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.location }}</span>
                    <input v-else v-model="course.location" class="border w-full px-1 bg-white" />
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.weekday }}</span>
                    <!-- <input v-else v-model="course.weekday" class="border w-full px-1 bg-white" /> -->
                    <select v-else v-model="course.weekday" class="border w-full px-1 bg-white">
                        <option v-for="n in 7" :key="n" :value="n">{{ n }}</option>
                    </select>
                </div>
                <div class="h-16 bg-[#EEEEEE] flex flex-col border border-[#146d2b] items-center justify-center px-1">
                    <span v-if="!course.editing">{{ course.start_section === course.end_section ? course.start_section :
                        course.start_section + '~' + course.end_section }}</span>
                    <div v-else class="flex">
                        <select v-model="course.start_section" class="border px-1 bg-white mx-1">
                            <option v-for="n in 14" :key="n" :value="n">{{ n }}</option>
                        </select>
                        ~
                        <select v-model="course.end_section" class="border px-1 bg-white mx-1">
                            <option v-for="n in 14" :key="n" :value="n">{{ n }}</option>
                        </select>
                    </div>
                </div>
                <div class="h-16 bg-[#EEEEEE] gap-2 flex border border-black items-center justify-center text-white">
                    <span v-if="!course.editing"
                        class="bg-[#1878d8] px-4 py-2  rounded cursor-pointer hover:bg-[#0061c2] hover:text-black"
                        @click="startEdit(course)">
                        編輯
                    </span>
                    <span v-if="course.editing"
                        class="bg-green-600 px-4 py-2 rounded cursor-pointer hover:bg-green-700 hover:text-black"
                        @click="saveCourse(course)">
                        儲存
                    </span>
                    <span v-if="course.editing"
                        class="bg-gray-500 px-4 py-2 rounded cursor-pointer hover:bg-gray-600 hover:text-black"
                        @click="cancelEdit(course)">
                        取消
                    </span>
                    <span v-if="!course.editing"
                        class="bg-amber-700 px-4 py-2 rounded cursor-pointer hover:bg-amber-800 hover:text-black"
                        @click="deleteCheck = true; selectedCourseId = course.id">
                        刪除
                    </span>
                </div>
            </template>
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
    <!-- 刪除確認 -->
    <div v-if="deleteCheck" class="fixed inset-0 bg-black/50 flex items-center justify-center z-40">
        <div class="bg-[#23f157] px-20 py-10 rounded-lg border-2 border-black text-center">
            <p class="text-2xl text-red-600 font-bold">是否確定刪除此課程？</p>
            <div class="flex gap-10 mt-10 justify-center">
                <button
                    class="bg-red-600 border border-black text-white px-10 py-4 rounded hover:bg-red-800 cursor-pointer"
                    @click="deleteCourse()">
                    確認
                </button>
                <button class="bg-white border px-10 py-4 rounded hover:bg-gray-200 cursor-pointer"
                    @click="deleteCheck = false">
                    取消
                </button>
            </div>
        </div>
    </div>

    <div class="absolute right-4 bottom-4 size-20 rounded-full bg-white hover:bg-gray-200 border cursor-pointer z-60"
        @click="createCourse = !createCourse">
        <div class="flex items-center justify-center w-full h-full">
            <NuxtImg src="edit_icon.svg" class="size-12" />
        </div>
    </div>
    <div v-if="createCourse" class="fixed inset-0 bg-black/50 z-40 flex items-center justify-center">
        <div
            class="flex flex-col items-center justify-start w-[clamp(0px,70vw,9999px)] h-[clamp(0px,80vh,9999px)] mx-auto bg-[#d1fcdb] rounded-lg z-50 border gap-6 py-6 overflow-auto">
            <span class="text-4xl font-bold">新增課程</span>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.id" :text="'courseId'" />
                <textInput v-model="createFilter.semester" :text="'semester'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.name_zh" :text="'name_zh'" />
                <textInput v-model="createFilter.name_en" :text="'name_en'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.grade" :text="'grade'" />
                <textInput v-model="createFilter.credit" :text="'credit'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.class_group" :text="'class_group'" />
                <textInput v-model="createFilter.group_code" :text="'group_code'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.required_type" :text="'required_type'" />
                <textInput v-model="createFilter.category" :text="'category'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.limit_min" :text="'limit_min'" />
                <textInput v-model="createFilter.limit_max" :text="'limit_max'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.department_name" :text="'department_name'" />
                <textInput v-model="createFilter.teacher_name" :text="'teacher_name'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.chinese_summary" :text="'chinese_summary'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.english_summary" :text="'english_summary'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.raw_remark" :text="'raw_remark'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.times[0].weekday" :text="'weekday'" />
                <textInput v-model="createFilter.times[0].classroom" :text="'classroom'" />
            </div>
            <div class="flex w-full justify-center gap-6">
                <textInput v-model="createFilter.times[0].start_section" :text="'start_section'" />
                <textInput v-model="createFilter.times[0].end_section" :text="'end_section'" />
            </div>
            <div class="flex w-full justify-end gap-6">
                <button class="bg-white border px-10 py-4 rounded hover:bg-gray-200 cursor-pointer"
                    @click="createCourse = false">
                    取消
                </button>
                <button
                    class="bg-green-600 border border-black text-white px-10 py-4 rounded hover:bg-green-800 cursor-pointer"
                    @click="createNewCourse()">
                    確認
                </button>
            </div>
        </div>
    </div>
</template>