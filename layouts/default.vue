<script setup>
import sidebarLink from '~/components/sidebarLink.vue';
import TextInput from '~/components/TextInput.vue';
const sidebarOpen = ref(false)
const isLogin = ref(false)
const isShowUserInfo = ref(false)
const userInfo = ref(null)

const token = useCookie('access_token', {
    path: '/'
})
const type = useCookie('token_type', {
    path: '/'
})
if (token.value) {
    isLogin.value = true
    async function getUserInfo() {
        userInfo.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/me', {
            method: 'GET',
            headers: {
                'Authorization': type.value + ' ' + token.value
            }
        }).catch((error) => {
            console.log(error)
        })
        console.log("使用者資訊：", userInfo.value)
        console.log("使用者：", userInfo.username)
    }
    await getUserInfo()
}
else {
    isLogin.value = false
}
const logout = () => {
    token.value = null
    type.value = null
    navigateTo('/login')
}
</script>
<template>
    <div class="flex flex-col max-w-screen h-screen relative bg-[#d1fcdb]">
        <header>
            <!-- 遮罩 -->
            <div v-if="sidebarOpen" class="fixed inset-0 bg-black/50 z-40 cursor-pointer" @click="sidebarOpen = false">
            </div>

            <!-- sidebar -->
            <div v-if="sidebarOpen"
                class="absolute bg-[#ffffff] pt-10 flex flex-col border-x border-b items-start justify-start h-screen w-60 md:w-72 shadow-[0_0px_14px_rgba(0,0,0,0.25)] z-50">

                <sidebarLink @click="sidebarOpen = false" :text="'課表'" :link="'/myTimetable'"
                    :image-url="'/component_img/sidebar_img/timetable_icon.svg'" />
                <sidebarLink @click="sidebarOpen = false" :text="'首頁'" :link="'/'" />
                <sidebarLink @click="sidebarOpen = false" :text="'查詢'" :link="'/courseSearch'"
                    :image-url="'/component_img/sidebar_img/search_icon.svg'" />
                <sidebarLink @click="sidebarOpen = false" :text="'測試用'" :link="'/unAuth'" />
            </div>

            <!-- header本體 -->
            <div
                class="flex items-center justify-between w-full h-20 md:h-32 px-6 md:px-10 2xl:px-16 gap-4 md:gap-7 2xl:gap-16 bg-[#74EF93] border-y border-[#146d2b] ">
                <svg class="hover:text-[#3867DC] size-8 md:size-10 2xl:size-12  cursor-pointer"
                    @click="sidebarOpen = !sidebarOpen" viewBox="0 0 41 31" fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <line y1="2.5" x2="41" y2="2.5" stroke="currentColor" stroke-width="5" />
                    <line y1="15.5" x2="41" y2="15.5" stroke="currentColor" stroke-width="5" />
                    <line y1="28.5" x2="41" y2="28.5" stroke="currentColor" stroke-width="5" />
                </svg>
                <span class="flex-1 text-xl md:text-4xl text-black font-bold tracking-widest">國北護課程查詢系統</span>
                <div class="flex gap-1 md:gap-4 hover:text-[#3867DC] items-center cursor-pointer" @click="logout()">
                    <NuxtImg src="/header_img/logout_icon.svg" alt="logout_icon" class="size-7 md:size-9" />
                    <!-- click後跳回login並清空cookie，這樣不論是否login都能起效? -->
                    <span class="text-xl md:text-2xl font-bold">{{ isLogin ? '登出' : '登入' }}</span>
                </div>
            </div>

            <!-- 底下顯示user -->
            <div v-if="!isShowUserInfo && userInfo"
                class="max-w-screen flex justify-end pr-5 md:pr-10 pt-4 bg-[#d1fcdb]">
                <span class="text-black text-xl md:text-2xl">歡迎</span><span
                    class="text-[#3867DC] text-xl md:text-2xl hover:underline hover:font-medium cursor-pointer hover:text-[#0031ad]"
                    @click="isShowUserInfo = !isShowUserInfo">{{ userInfo.username }}</span>
            </div>
        </header>
        <slot v-if="!isShowUserInfo" class="flex-1" />
        <div v-if="isShowUserInfo" class="pt-20 w-full flex items-center justify-center ">
            <div
                class="bg-[#F6F6F6] relative w-[clamp(0px,120vmin,9999px)] h-[clamp(0px,75vmin,9999px)] flex justify-center pt-20 mx-auto border-2 border-black">
                <div class="w-full flex flex-col gap-20 items-center">
                    <span class="text-4xl tracking-widest font-bold">個人帳號管理</span>
                    <nav class="w-full flex flex-col items-center ">
                        <div class="flex-col flex gap-16 ">
                            <span class="text-4xl tracking-widest">名字:<span class=" ">Sea{{ }}</span></span>
                            <span class="text-4xl tracking-widest">學號:<span class=" ">122214136{{ }}</span></span>
                            <span class="text-4xl tracking-widest">系所:<span class=" ">資訊管理系{{ }}</span></span>
                            <span class="text-4xl tracking-normal">Email:<span class=" pl-1">zzznn.chen@gmail.com{{ }}</span></span>
                        </div>
                    </nav>
                </div>
                <div class="absolute h-full flex flex-col items-center gap-20 top-10 left-16">
                    <div class="">
                        <NuxtImg src="/component_img/eye_icon.svg" alt="eyes_icon"
                            class="border rounded-full size-20 md:size-52" />
                    </div>
                    <button
                        class="bg-[#d1fcdb] border w-40 h-16 text-2xl rounded-b-sm cursor-pointer hover:font-bold hover:bg-[#74EF93]"
                        @click="isShowUserInfo = false">上傳頭像</button>
                </div>

                <button
                    class="absolute bottom-8 right-8 text-2xl w-40 h-16 border rounded-sm bg-[#23f157] cursor-pointer hover:font-bold hover:bg-[#18a83c]">儲存</button>
            </div>
        </div>
    </div>
</template>