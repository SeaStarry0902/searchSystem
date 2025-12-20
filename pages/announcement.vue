<script setup>
import { ref, onMounted } from 'vue'
onMounted(() => {
    getRole()
    getAnnouncement()
})
const token = useCookie('access_token')
const type = useCookie('token_type')
const role = ref('')
const deleteCheck = ref(false)

const createAnnouncement = ref(false)
const category = ['全部', '教務處', '課程異動', '系所', '活動']
const announcements = ref([])
const filteredAnnouncements = ref([])
let selectedAnnouncement = ref(null)
const nameTranslate = {
    office: '教務處',
    course_change: '課程異動',
    department: '系所',
    activity: '活動'
}
const createFilter = ref({
    title: "",
    content: "",
    category: "",
    is_pinned: false,
    is_active: true
})
const showContent = ref(false)
// Available values : office, course_change, department, activity
function changeCategory(key) {
    let temKey = nameTranslate[key] || key
    filteredAnnouncements.value = announcements.value.filter(announcement => {
        if (temKey === '全部') {
            return true
        }
        return nameTranslate[announcement.category] === temKey
    })
    console.log("篩選後公告：", filteredAnnouncements.value)
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
        console.log("無法取得使用者角色")
    }

}
async function getAnnouncement() {
    try {
        const announcement = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/announcements?page=1&page_size=30&include_inactive=false', {})
        console.log('回傳', announcement)
        announcements.value = announcement.items.map((item => ({
            id: item.id,
            title: item.title,
            content: item.content,
            category: item.category,
            created_at: item.created_at,
            showContent: false
        })))
        filteredAnnouncements.value = announcements.value
        console.log("公告資料：", announcements.value)
    } catch (error) {
        console.log("無法取得公告資料")
    }
}
async function createNewAnnouncement() {
    console.log('建立公告：', createFilter.value)
    await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/announcements`,
        {
            method: 'POST',
            headers: {
                Authorization: `${type.value} ${token.value}`,
                'Content-Type': 'application/json'
            },
            body: createFilter.value
        }
    )
    createAnnouncement.value = false
    getAnnouncement()
}
async function deleteAnnouncement() {
    await $fetch(`https://representative-winni-chongouo-b8ca194b.koyeb.app/announcements/${selectedAnnouncement.value}`,
        {
            method: 'DELETE',
            headers: {
                Authorization: `${type.value} ${token.value}`
            }
        }
    )
    deleteCheck.value = false
    getAnnouncement()
}
</script>
<template>
    <div class="flex flex-col pt-10 items-center w-full h-screen overflow-auto">
        <div class=" w-[clamp(0px,70vw,9999px)] max-h-full flex flex-col items-center ">
            <span class="text-6xl">公告</span>
            <div class="mt-10 flex w-full justify-start">
                <div class="flex gap-4 text-4xl justify-start">
                    <button v-for="key in category" class=" hover:bg-green-400 rounded-t-lg cursor-pointer"
                        @click="changeCategory(key)">
                        {{ key }}</button>
                </div>
            </div>
            <div class="border w-full h-px"></div>

            <div class="pt-5 w-full flex-1 flex flex-col items-center overflow-auto">
                <template v-for="announcement in filteredAnnouncements" :key="announcement.id">
                    <div class="flex w-full items-center justify-start px-1 py-2 " ">
                        <div class=" flex border-l-2 border-[#74EF93] w-full items-center justify-between px-4">
                        <div class="pl-4 flex flex-col">
                            <div class="flex items-center gap-1">
                                <span class="text-4xl font-bold">{{ announcement.title }}</span>
                                <NuxtImg src="delete_icon.svg" v-if="role === 'admin'" class="size-6 cursor-pointer"
                                    @click="deleteCheck = true; selectedAnnouncement = announcement.id" />
                                <!-- <button v-if="role === 'admin'" @click.stop="deleteAnnouncement(announcement.id)">刪除</button> -->
                            </div>

                            <span class="text-2xl text-gray-600 mt-2">{{ new
                                Date(announcement.created_at).toLocaleDateString('zh-TW') }}</span>
                        </div>
                        <NuxtImg src="/arrow_icon.svg"
                            :class="announcement.showContent ? 'rotate-270 size-10 cursor-pointer' : 'size-10 cursor-pointer'"
                            @click="announcement.showContent = !announcement.showContent" />
                    </div>
            </div>
            <div class="border w-full h-px my-2 "></div>
            <div v-if="announcement.showContent" class="w-full px-8 py-2">
                <span class="text-2xl">{{ announcement.content }}</span>
            </div>
</template>
</div>
</div>

<div v-if="deleteCheck" class="fixed inset-0 bg-black/50 flex items-center justify-center z-40">
    <div class="bg-[#23f157] px-20 py-10 rounded-lg border-2 border-black text-center">
        <p class="text-2xl text-red-600 font-bold">是否確定刪除此公告？</p>
        <div class="flex gap-10 mt-10 justify-center">
            <button class="bg-red-600 border border-black text-white px-10 py-4 rounded hover:bg-red-800 cursor-pointer"
                @click="deleteAnnouncement()">
                確認
            </button>
            <button class="bg-white border px-10 py-4 rounded hover:bg-gray-200 cursor-pointer"
                @click="deleteCheck = false">
                取消
            </button>
        </div>
    </div>
</div>

<div v-if="role === 'admin'"
    class="absolute right-4 bottom-4 size-20 rounded-full bg-white hover:bg-gray-200 border cursor-pointer z-60"
    @click="createAnnouncement = !createAnnouncement">
    <div class="flex items-center justify-center w-full h-full">
        <NuxtImg src="edit_icon.svg" class="size-12" />
    </div>
</div>
<div v-if="createAnnouncement" class="fixed inset-0 bg-black/50 z-40 flex items-center justify-center">
    <div
        class="flex flex-col items-center justify-start w-[clamp(0px,70vw,9999px)] h-[clamp(0px,80vh,9999px)] mx-auto bg-[#d1fcdb] rounded-lg z-50 border gap-6 py-6 overflow-auto">
        <span class="text-4xl font-bold">新增公告</span>
        <div class="flex w-full justify-center gap-6">
            <textInput v-model="createFilter.title" :text="'標題'" />
            <textInput v-model="createFilter.category" :text="'類別'"
                :options="['office', 'course_change', 'department', 'activity']" />
        </div>
        <div class="flex w-full justify-center gap-6">
            <textInput v-model="createFilter.content" :text="'內容'" />
        </div>
        <div class="flex flex-1 w-full justify-end items-end gap-6 pr-2">
            <button class="bg-white border h-20 px-10 py-4 rounded hover:bg-gray-200 cursor-pointer text-center"
                @click="createAnnouncement = false">
                取消
            </button>
            <button
                class="bg-green-600 border h-20  border-black text-white px-10 py-4 rounded hover:bg-green-800 text-center cursor-pointer"
                @click="createNewAnnouncement()">
                確認
            </button>
        </div>
    </div>
</div>
</div>

</template>