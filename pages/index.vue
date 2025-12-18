<script setup>
definePageMeta({
    middleware: async function (to, from) {
        let role = ref('')
        const token = useCookie('access_token')
        const type = useCookie('token_type')
        if (!token.value) {
            return navigateTo('/login')
        }
        role.value = await $fetch('https://representative-winni-chongouo-b8ca194b.koyeb.app/auth/me', {
            method: 'GET',
            headers: {
                'Authorization': type.value + ' ' + token.value
            }
        })
        if (role.value.role === 'admin')
            return navigateTo('/admin/allUserInfo')
        else
            return navigateTo('/myTimetable')
    }
})
</script>
<template>
    <div class="text-2xl text-blue-500">
        <h1>Welcome to the indexPage</h1>
    </div>
</template>