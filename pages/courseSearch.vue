<script setup>
import { ref } from 'vue'
import testInput from '~/components/TextInput.vue'
const courseArray = ['編號', '學期', '系所', '年級', '班組', '科目分類', '課程名稱', '教師', '上課人數', '學分數', '課別', '節次', '收藏'];
const response = [];
const resultArray = [...courseArray, ...response];
for(let i=0;i < resultArray.length;i++){
    response.push(resultArray[i].courseName);
}


const ciso =ref('');
const courseCatagory = ref('');
</script>
<template>
    <div class="flex flex-col justify-center">
        <div class="flex items-center justify-center gap-20 w-full h-auto pt-4 px-96 text-2xl text-black bg-[#d1fcdb]">
            <div class="flex flex-col gap-5">
                <div class="flex gap-5">
                    <testInput v-model="courseCatagory" :text="'課程分類'" :customWidth="'w-72'" />
                    <testInput :text="'課程名稱'" :customWidth="'w-72'" />
                    <testInput :text="'課別'" :customWidth="'w-72'" />
                    <testInput :text="'教師'" :customWidth="'w-72'" />
                </div>
                <div class="flex gap-5">
                    <testInput :text="'學期'" :customWidth="'w-72'" :options="['選項1', '選項2']" />
                    <testInput :text="'學制'" :customWidth="'w-72'" :options="['選項1', '選項2']" />
                    <testInput :text="'系所'" :customWidth="'w-72'" :options="['選項1', '選項2']" />
                    <testInput :text="'年級'" :customWidth="'w-72'" :options="['選項1', '選項2']" />
                </div>
            </div>
            <div class="pt-10">
                <button
                    class="bg-white w-60 h-20 border rounded cursor-pointer hover:bg-[#6ffc92] hover:font-bold text-2xl">時間選擇</button>
            </div>
        </div>
        <div class="mt-10 flex items-center justify-center w-full">
            <button
                class="bg-[#23f157] w-96 h-16 border-2 rounded cursor-pointer hover:bg-[#18a83c] hover:font-bold text-3xl" @click="console.log(ciso)">搜尋</button>
        </div>
    </div>
    <div class="mt-10 grid border h-auto  grid-cols-13 overflow-auto ">
        <div v-for="(item, index) in courseArray" :key="index"
            :class="index < 13 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
            class=" flex flex-col border border-[#146d2b] items-center justify-center">
            <div>{{ item }}</div>
        </div>
    </div>
</template>