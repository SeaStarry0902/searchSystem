<script setup>
definePageMeta({
    middleware: function (to, from) {
        const token = useCookie('access_token')
        if (!token.value) {
            return navigateTo('/login')
        }
    }
})
</script>
<template>
    <div class="text-2xl text-blue-500">
        <h1>Welcome to the indexPage</h1>
    </div>
</template>