<template>
    <div>
        <div class="border border-solid bg-gray-100 text-gray-300 w-8 py-1 pl-2 pr-1 flex items-center">
            <button @click="toggleList" class="cursor-pointer w-6 h-6 text-gray-600 outline-none focus:outline-none">
                <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" fill="none" viewBox="0 0 24 24"
                    stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
                    class="feather feather-chevron-up w-4 h-4">
                    <polyline v-if="showList" points="18 15 12 9 6 15"></polyline>
                    <polyline v-if="!showList" points="6 9 12 15 18 9"></polyline>
                </svg>
            </button>
            <div class="w-8 h-8 flex justify-center items-center">
                <polyline v-if="showList" points="18 15 12 9 6 15"></polyline>
                <polyline v-if="!showList" points="6 9 12 15 18 9"></polyline>
            </div>
        </div>
        <div v-if="showList">
            <VuePerfectScrollbar class="scroll-area" v-once :settings="settings">
                <div v-for="(option, index) in list" :key="index"
                    class="cursor-pointer w-full border-gray-100 rounded-t border-b hover:bg-sky-300"
                    @click="handleClick(option)">
                    <div
                        class="flex w-full items-center p-2 pl-2 border-transparent border-l-2 relative hover:border-teal-100">
                        <div class="w-6 flex flex-col items-center">
                            <div
                                class="flex relative w-5 h-5 justify-center items-center m-1 mr-2 w-4 h-4 mt-1 rounded-full ">
                                <img class="rounded-full" :alt="option.ticker" :src="option.image">
                            </div>
                        </div>
                        <div class="w-full items-center flex">
                            <div class="mx-2 -mt-1">{{ option.name }}</div>
                        </div>
                    </div>
                </div>
            </VuePerfectScrollbar>
        </div>
    </div>
</template>

<script>

    import VuePerfectScrollbar from 'vue-perfect-scrollbar/index.vue';

    export default {
        data() {
            return {
                showList: false,
                settings: {
                    maxScrollbarLength: 60
                }
            }
        },
        props: {
            list: Array,
            value: Object,
        },
        components: {
            VuePerfectScrollbar,
        },
        name: 'SelectForm',
        methods: {
            handleClick(option) {
                this.showList = false;
                this.$emit('change-option', option);
            },
            toggleList() {
                this.showList = !this.showList;
            },
        }
    }
</script>

<style scoped>
    .scroll-area {
        position: absolute;
        width: 400px;
        height: 300px;
        background-color: #F6F7F8;
    }

    * {
        font-family: 'Vollkorn', serif;
    }
</style>