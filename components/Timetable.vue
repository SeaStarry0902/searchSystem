<script setup>
import timetable from '~/components/Timetable.vue'
const props = defineProps({
    customHeight: {
        type: String,
        default: ''
    }
})
const weekArray = ['', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'];
const apiResponse = ['國文', '', '英文', '', '數學', '', '', '', '', '體育', '', '', '', '', '', '', '', '', '歷史'];
const totalCells = 112
const result = [...weekArray];
var period = 1
var apiIndex = 0
for (let i = 0; i < totalCells; i++) {
    if (i % 8 === 0) {
        result.push(`第${period}節`);
        period++;
        continue;
    }
    else {
        result.push(apiResponse[apiIndex]);
        apiIndex++;
    }

}
</script>
<template>
    <div :class="customHeight ? customHeight : 'h-[700px]'" class="grid border-2  border-[#146d2b] grid-cols-8 overflow-auto ">
        <div v-for="(item, index) in result" :key="index"
            :class="index < 8 ? 'h-10 bg-[#EEEEEE] font-bold ' : 'h-16 bg-white'"
            class=" flex flex-col border border-[#146d2b] items-center justify-center">
            <div>{{ item }}</div>
            <span :class="index < 8 ? 'hidden' : item && index % 8 != 0 ? 'text-xs' : 'hidden'">B123</span>
        </div>
    </div>
</template>