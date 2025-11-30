<script setup>
definePageMeta({
    middleware: function (to, from) {
        const auth = useCookie('user')
        if (!auth.value) {
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