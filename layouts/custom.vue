<script setup>
function definePageMeta(meta) {

}
</script>
<template>
    <div class="w-full h-screen bg-cover bg-center " style="background-image: url('/login_img/login_bg.svg')">
        <div class="h-full flex justify-center items-center">
            <main>
                <div
                    class="flex max-w-screen flex-col justify-start items-center pt-[clamp(0px,3vmin,9999px)] px-[clamp(0px,20vmin,9999px)] bg-[#F6F6F6] rounded-[14px] border-2 border-[#737272] h-[clamp(0px,75vmin,9999px)] w-[clamp(0px,110vmin,9999px)] ">
                    <a href="https://www.ntunhs.edu.tw/" class="w-full">
                        <NuxtImg src="/login_img/NTUNHS_icon.svg" title="https://www.ntunhs.edu.tw/" alt="NTUNHS_icon"
                            class="w-full" />
                    </a>
                    <slot />
                </div>
            </main>
        </div>
    </div>
</template>