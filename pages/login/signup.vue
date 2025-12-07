<script setup>
import textInput from '~/components/TextInput.vue'
definePageMeta({
    layout: 'custom'
})
const firstPassword = ref('')
const secordPassword = ref('')
const account = ref('')
async function signup() {
    if (!account.value || !firstPassword.value || !secordPassword.value) {
        alert("請輸入完整資訊")
        return
    }
    if (firstPassword.value !== secordPassword.value) {
        alert("密碼與確認密碼不相符")
        return
    }
    const res = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/register', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body:({
            username: account.value,
            password: firstPassword.value
        })
    }).catch((error) => {
        alert(error)
        
    })
    console.log(res)
    navigateTo('/login')
}
</script>
<template>
    <div
        class="relative w-full flex flex-col justify-center items-center mt-[clamp(0px,5vmin,9999px)] gap-[clamp(0px,6vmin,9999px)]">
        <div class=" flex flex-col gap-[clamp(0px,3vmin,9999px)]">
            <textInput v-model="account" :text="'帳號'" />
            <textInput v-model="firstPassword" :text="'密碼'" :showIcon="true" />
            <textInput v-model="secordPassword" :text="'確認密碼'" :showIcon="true" />
        </div>
        <button @click="signup()"
            class="flex hover:font-bold items-center justify-center hover:bg-[#069736] w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] border border-black cursor-pointer bg-[#09ca49] rounded-[clamp(0px,1vmin,40px)] text-white text-xs md:text-xl 2xl:text-2xl"><span
                class="text-xs md:text-xl 2xl:text-2xl">註冊</span></button>
        <!-- <NuxtLink to="/login"
            class="flex hover:font-bold items-center justify-center hover:bg-[#069736] w-[clamp(0px,40vmin,9999px)] h-[clamp(0px,6vmin,9999px)] border border-black cursor-pointer bg-[#09ca49] rounded-[clamp(0px,1vmin,40px)] text-white text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl">註冊</span>
        </NuxtLink> -->
        <NuxtLink to="/login" class="absolute top-0 -left-10 md:-left-20 text-xs md:text-xl 2xl:text-2xl">
            <span class="text-xs md:text-xl 2xl:text-2xl text-[#3867DC] hover:font-bold ">返回</span>
        </NuxtLink>
    </div>
</template>