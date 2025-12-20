<script setup>
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
import textInput from '~/components/TextInput.vue'
const name = ref('')
const student_no = ref('')
const department = ref('')
const email = ref('')
const filters = {
    name,
    student_no,
    department,
    email
}
const token = useCookie('access_token')
const type = useCookie('token_type')

const users = ref([])
const deleteCheck = ref(false)
const selectedUserId = ref(null)
const baseUrl = 'https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/users?page=1&page_size=60'

async function buildUrl() {
    let url = baseUrl
    const params = []

    for (const key in filters) {
        if (filters[key].value) {
            params.push(`${key}=${encodeURIComponent(filters[key].value)}`)
        }
    }

    if (params.length) {
        url += '&' + params.join('&')
    }
    return url
}

async function getAllUserInfo() {
    const url = await buildUrl()

    const res = await $fetch(url, {
        headers: {
            Authorization: `${type.value} ${token.value}`
        }
    })
    console.log('回傳結果:', res)
    users.value = res.items.map(user => ({
        id: user.id,
        full_name: user.full_name || '',
        student_no: user.student_no || '',
        department: user.department_name || '',
        email: user.email || '',
        last_login_at: user.last_login_at ? new Date(user.last_login_at).toLocaleString('zh-TW', { hour12: false }) : '',
        editing: false,
        _backup: null
    }))
    console.log('使用者列表：', users.value)
}

function startEdit(user) {
    user._backup = { ...user }
    user.editing = true
}

function cancelEdit(user) {
    Object.assign(user, user._backup)
    user.editing = false
}

async function saveUser(user) {
    await $fetch(
        `https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/users/${user.id}`,
        {
            method: 'PUT',
            headers: {
                Authorization: `${type.value} ${token.value}`
            },
            body: {
                full_name: user.full_name,
                student_no: user.student_no,
                department_name: user.department,
                email: user.email
            }
        }
    )

    user.editing = false
}

async function deleteUser() {
    await $fetch(
        `https://representative-winni-chongouo-b8ca194b.koyeb.app/admin/users/${selectedUserId.value}`,
        {
            method: 'DELETE',
            headers: {
                Authorization: `${type.value} ${token.value}`
            }
        }
    )

    deleteCheck.value = false
    selectedUserId.value = null
    getAllUserInfo()
}
</script>
<template>
    <div class="flex flex-col h-screen w-full items-center px-10 gap-10 overflow-auto pt-10">
        <span class="text-6xl">使用者管理</span>
        <div class="flex gap-10">
            <textInput v-model="name" text="姓名" />
            <textInput v-model="student_no" text="學號" />
            <textInput v-model="department" text="系所" />
            <textInput v-model="email" text="Email" />
        </div>

        <button class="bg-[#23f157] w-96 h-16 border rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl"
            @click="getAllUserInfo">
            <span>搜尋</span>
        </button>

        <div class="flex w-full justify-center overflow-auto">
            <div class="inline-grid grid-cols-6 w-full text-center">

                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">姓名</span>
                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">學號</span>
                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">系所</span>
                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">Email</span>
                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">最後登入</span>
                <span class="bg-[#23f157] border h-20 flex items-center justify-center text-2xl font-bold">操作</span>

                <template v-for="user in users" :key="user.id">
                    <div class="border h-20 flex items-center justify-center px-2 bg-white">
                        <span v-if="!user.editing">{{ user.full_name }}</span>
                        <input v-else v-model="user.full_name" class="border px-2 py-1 w-full " />
                    </div>
                    <div class="border h-20 flex items-center justify-center px-2 bg-white">
                        <span v-if="!user.editing">{{ user.student_no }}</span>
                        <input v-else v-model="user.student_no" class="border px-2 py-1 w-full" />
                    </div>
                    <div class="border h-20 flex items-center justify-center px-2 bg-white">
                        <span v-if="!user.editing">{{ user.department }}</span>
                        <input v-else v-model="user.department" class="border px-2 py-1 w-full" />
                    </div>
                    <div class="border h-20 flex items-center justify-center px-2 bg-white">
                        <span v-if="!user.editing">{{ user.email }}</span>
                        <input v-else v-model="user.email" class="border px-2 py-1 w-full" />
                    </div>
                    <div class="border h-20 flex items-center justify-center bg-white">
                        {{ user.last_login_at }}
                    </div>
                    <div class="border border-black h-20 flex gap-3 items-center justify-center text-white bg-white">
                        <span v-if="!user.editing"
                            class="bg-[#1878d8] px-6 py-2 rounded cursor-pointer hover:bg-[#0061c2]"
                            @click="startEdit(user)">
                            編輯
                        </span>
                        <span v-if="user.editing"
                            class="bg-green-600 px-6 py-2 rounded cursor-pointer hover:bg-green-700"
                            @click="saveUser(user)">
                            儲存
                        </span>
                        <span v-if="user.editing" class="bg-gray-500 px-6 py-2 rounded cursor-pointer hover:bg-gray-600"
                            @click="cancelEdit(user)">
                            取消
                        </span>
                        <span v-if="!user.editing"
                            class="bg-amber-700 px-6 py-2 rounded cursor-pointer hover:bg-amber-800"
                            @click="deleteCheck = true; selectedUserId = user.id">
                            刪除
                        </span>
                    </div>
                </template>

            </div>
        </div>
        <div v-if="deleteCheck" class="fixed inset-0 bg-black/50 flex items-center justify-center z-40">
            <div class="bg-[#23f157] px-20 py-10 rounded-lg border-2 border-black text-center">
                <p class="text-2xl text-red-600 font-bold">是否確定刪除此使用者？</p>
                <div class="flex gap-10 mt-10 justify-center">
                    <button
                        class="bg-red-600 border border-black text-white px-10 py-4 rounded hover:bg-red-800 cursor-pointer"
                        @click="deleteUser()">
                        確認
                    </button>
                    <button class="bg-white border px-10 py-4 rounded hover:bg-gray-200 cursor-pointer"
                        @click="deleteCheck = false">
                        取消
                    </button>
                </div>
            </div>
        </div>

    </div>
</template>