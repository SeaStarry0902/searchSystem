<script setup>
import { ref, onMounted } from 'vue'
onMounted(() => {
    getRole()
})
const token = useCookie('access_token')
const type = useCookie('token_type')
const role = ref('')
const catagory = ['全部', '教務處', '課程異動', '系所消息', '活動']
// Available values : office, course_change, department, activity
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
</script>
<template>
    <div class="mt-10 w-full h-full flex justify-center ">
        <div class="flex w- underline gap-2 justify-start">
            <div v-for="key in catagory" class="">{{ key }}</div>
        </div>
    </div>
</template>