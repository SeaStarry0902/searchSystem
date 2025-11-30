<script setup> 
import textInput from '~/components/TextInput.vue'


definePageMeta({
  layout: 'custom'
})
</script>
<template>
    <div class="flex flex-col gap-[clamp(0px,3vmin,9999px)] mt-[clamp(0px,5vmin,9999px)]">
        <textInput :text='"名稱"'  />
        <textInput :text="'fwqewe'":showIcon="true" />
        <textInput :text="'密碼'" :showIcon="true" />
</div>
</template>