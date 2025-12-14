<script setup>
import { ref, onMounted } from 'vue'
definePageMeta({
    middleware: function (to, from) {
        const token = useCookie('access_token')
        if (!token.value) {
            return navigateTo('/unAuth')
        }
    }
})
onMounted(() => {
    getCredit()
})
// curl -X 'GET' \
//   'https://representative-winni-chongouo-b8ca194b.koyeb.app/students/me/credits/summary' \
//   -H 'accept: application/json' \
//   -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzZWEiLCJleHAiOjE3NjU2NDQwMzN9.LG4rzw5mB8Jd-HgLTBjf0IL0UqEaj33LLjGeJ9fcewY'
const token = useCookie('access_token')
const type = useCookie('token_type')
const showInfo = ref(false)
const userCredit = ref(null)
const select = ref(false)
const selectedProgram = ref('')
let userProgram = ref('')
let radius = 60
let circleLength = 2 * Math.PI * radius
// let circleOffset = circleLength * (100 - 10) / 100 
let circleOffset = ref('')
async function getCredit() {
    userCredit.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/students/me/credits/summary', {
        headers: {
            'Authorization': type.value + ' ' + token.value
        }
    })
    selectedProgram.value = userCredit.value.program.selected.name
    circleOffset.value = circleLength * (100 - userCredit.value.graduation.progress_percent) / 100
    console.log(userCredit.value)
    console.log(selectedProgram.value)
    showInfo.value = true
}
// curl -X 'PUT' \
//   'https://representative-winni-chongouo-b8ca194b.koyeb.app/students/me/program' \
//   -H 'accept: application/json' \
//   -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzZWEiLCJleHAiOjE3NjU2NDY3MjF9.niZ1YMe2xYAWs6knMqcuShq7eUzYajMTIGfuGkucb7M' \
//   -H 'Content-Type: application/json' \
//   -d '{
//   "program_code": "ai_application"
// }'
async function changeProgram() {
    try {
        console.log(selectedProgram.value)
        await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/students/me/program', {
            method: 'PUT',
            headers: {
                'Authorization': type.value + ' ' + token.value,
                'Content-Type': 'application/json',
                'accept': 'application/json'
            },
            body: {
                program_code: `${selectedProgram.value}`
            }
        })
    } catch (error) {
        console.log(error)
    }
    getCredit()
    alert('變更成功')
    select.value = false
}
</script>
<template>
    <div v-if="showInfo" class="flex flex-col w-[1200px] mx-auto gap-16 overflow-auto ">
        <div class="flex justify-between w-full">
            <div class="border bg-white w-80 h-64 rounded-lg flex items-center justify-center">
                <div class="flex flex-col items-center justify-center gap-6">
                    <span class="text-5xl">{{ userCredit.graduation.earned_total }}</span>
                    <span class="text-4xl font-bold">已修學分</span>
                </div>
            </div>
            <div class="relative border bg-white w-80 h-64 rounded-lg flex flex-col items-center justify-center">
                <div class="absolute top-1 right-2 text-[#3867DC] text-xl ">
                    <button v-if="!select" class="cursor-pointer hover:font-bold" @click="select = true">編輯</button>
                    <select v-model="selectedProgram" v-if="select" class="cursor-pointer" @change="changeProgram()">
                        <option value="ai_application">人工智慧應用學程</option>
                        <option value="big_data">大數據學程</option>
                        <option value="medical_it">醫療資訊學程</option>
                    </select>
                </div>
                <div class="flex flex-col items-center justify-center gap-6">
                    <span class="text-5xl">{{ userCredit.program.remaining }}</span>
                    <span class="text-4xl text-[#E19C08] font-bold">尚須學程學分</span>
                </div>
            </div>
            <div class="border bg-white w-80 h-64 rounded-lg flex items-center justify-center">
                <div class="flex flex-col items-center justify-center gap-6">
                    <span class="text-5xl">{{ userCredit.graduation.remaining_total }}</span>
                    <span class="text-4xl text-[#ED2323] font-bold">距離畢業還差</span>
                </div>
            </div>
        </div>
        <div class="flex justify-between w-full">
            <div
                class=" w-[650px] h-[347px] border rounded-lg bg-white flex flex-col items-center justify-between py-6">
                <div class="flex justify-center gap-8 text-4xl text-center">
                    <span class="w-26 ">類別</span>
                    <span class="w-36">所需學分</span>
                    <span class="w-20">已修</span>
                    <span class="w-20">未修</span>
                    <span>狀態</span>
                </div>
                <div class="border border-black w-full h-px"></div>
                <div class="flex gap-8 justify-center text-2xl text-center">
                    <span class="w-26 ">專業必修</span>
                    <span class="w-36">{{ userCredit.categories[0].required }}</span>
                    <span class="w-20">{{ userCredit.categories[0].earned }}</span>
                    <span class="w-20">{{ userCredit.categories[0].remaining }}</span>
                    <span>{{ userCredit.categories[0].remaining ? `進行中` : `完成` }}</span>
                </div>
                <div class="border border-black w-full h-px"></div>
                <div class="flex gap-8 justify-center text-2xl text-center">
                    <span class="w-26 ">專業選修</span>
                    <span class="w-36">{{ userCredit.categories[1].required }}</span>
                    <span class="w-20">{{ userCredit.categories[1].earned }}</span>
                    <span class="w-20">{{ userCredit.categories[1].remaining }}</span>
                    <span>{{ userCredit.categories[1].remaining ? `進行中` : `完成` }}</span>
                </div>
                <div class="border border-black w-full h-px"></div>
                <div class="flex gap-8 justify-center text-2xl text-center">
                    <span class="w-26 ">通識必修</span>
                    <span class="w-36">{{ userCredit.categories[2].required }}</span>
                    <span class="w-20">{{ userCredit.categories[2].earned }}</span>
                    <span class="w-20">{{ userCredit.categories[2].remaining }}</span>
                    <span>{{ userCredit.categories[2].remaining ? `進行中` : `完成` }}</span>
                </div>
            </div>
            <!-- <circle cx="50%" cy="50%" :r="radius" fill="transparent" stroke="#22c55e" stroke-width="10" stroke-linecap="round" :stroke-dasharray="circumference" :stroke-dashoffset="dashOffset" class="transition-all duration-500" /> -->
            <div class="w-[400px] h-[347px] border rounded-lg bg-white flex flex-col items-center justify-center gap-6">
                
                <span class="text-4xl font-bold">學業完成進度</span>
                <svg class=" -rotate-90 ">
                    <circle cx="50%" cy="50%" :r=radius fill="white" stroke="#EEEEEE" stroke-width="20" />
                    <circle cx="50%" cy="50%" :r=radius fill="white" stroke="blue" stroke-width="20"
                        :stroke-dasharray=circleLength :stroke-dashoffset=circleOffset />
                </svg>
                <span class="text-4xl">{{ userCredit.graduation.progress_percent }}%</span>
            </div>
        </div>
    </div>
</template>