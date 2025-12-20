<script setup>
import textInput from '~/components/TextInput.vue'
const username = ref('')
const password = ref('')
const rememberMe = ref(false)
async function login() {
    console.log("記住帳號：", rememberMe.value)
    if (!username.value || !password.value) {
        alert("請輸入完整資訊")
        return
    }
    const res = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/login', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: new URLSearchParams({
            grant_type: 'password',
            username: username.value,
            password: password.value,
            scope: '',
            client_id: 'string',
            client_secret: '********'
        })
    }).catch((error) => {
        alert("登入失敗，請檢查帳號密碼是否正確")
    })
    console.log(res)
    const token = useCookie('access_token', {
        path: '/',
        maxAge: rememberMe.value ? 60 * 60 * 24 * 7 : undefined
    })
    const type = useCookie('token_type', {
        path: '/',
        maxAge: rememberMe.value ? 60 * 60 * 24 * 7 : undefined// 7天
    })

    token.value = res.access_token
    type.value = res.token_type
    navigateTo('/')
}

definePageMeta({
    layout: 'custom'
})
</script>
<template>
    <div class="flex flex-col justify-center items-center mt-[clamp(0px,5vmin,9999px)] gap-[clamp(0px,2.5vmin,9999px)]">
        <div class="flex flex-col gap-[clamp(0px,3vmin,9999px)]">
            <textInput v-model="username" id="username" :text="'學號'" />
            <textInput v-model="password" id="password" :text="'密碼'" :showIcon="true" />
        </div>
        <div class="flex items-center justify-center">
            <label class="flex items-center justify-center gap-1 md:gap-2 cursor-pointer">
                <input type="checkbox" v-model="rememberMe" class="size-3 md:size-5 cursor-pointer" />
                <span class="text-xs md:text-xl">記住帳號</span>
            </label>
        </div>
        <button @click="login()"
            class=" hover:font-bold hover:bg-[#069736] w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] border border-black cursor-pointer bg-[#09ca49] rounded-[clamp(0px,1vmin,40px)] text-white text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl">登入</span>
        </button>
        <NuxtLink to="/courseSearch"
            class="md:pt-1 2xl:pt-3 text-[#3867DC] text-xs md:text-xl 2xl:text-3xl hover:font-bold">
            訪客登入</NuxtLink>
        <!-- <nav class="flex pt-10 items-center justify-center gap-[clamp(0px,4vmin,9999px)]">
            <NuxtLink to="/login/signup" class="text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">註冊
            </NuxtLink>
            <NuxtLink to="/myTimetable"
                class="text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">訪客登入</NuxtLink>
        </nav> -->
        <!-- <NuxtLink to="/myTimetable"
            class="md:pt-2 2xl:pt-4 text-[#3867DC] text-xs md:text-xl 2xl:text-2xl  hover:font-bold">
            訪客登入</NuxtLink> -->
    </div>
</template>