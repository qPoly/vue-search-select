<script setup>
import { computed, ref } from 'vue';

const props = defineProps(['modelValue', 'rows', 'title', 'subtitle', 'searchfields']);
const emit = defineEmits(['update:modelValue']);

const inputRef = ref(null);
const searchQuery = ref('');
const open = ref(false);

const title = typeof props.title === 'function'
    ? props.title
    : (row) => row[props.title];

const subtitle = typeof props.subtitle === 'function'
    ? props.subtitle
    : (row) => row[props.subtitle];

const filteredRows = computed(() =>
    searchQuery.value === ''
        ? props.rows
        : props.rows.filter(row => {
            for (let searchfield of props.searchfields) {
                if (row[searchfield] && row[searchfield].toLowerCase().includes(searchQuery.value.toLowerCase())) {
                    return true;
                }
            }
        })
);

const onSelect = (row) => {
    emit('update:modelValue', row);
    open.value = false;
    searchQuery.value = '';
};

const toggle = () => {
    open.value = !open.value;

    if (props.searchfields && open.value) {
        setTimeout(() => {
            inputRef.value.focus();
        }, 200);
    }
};
</script>

<template>
    <div class="w-full relative border border-gray-700 bg-gray-900 text-gray-300 rounded-md shadow-sm cursor-default select-none" :class="{ 'rounded-b-none border-indigo-500 outline-indigo-500 outline-1 outline': open }">
        <div class="h-full p-3 flex justify-between items-center" @click="toggle">
            <div class="leading-none">
                <template v-if="modelValue">
                    {{ title(modelValue) }}
                    <div class="text-xs text-gray-400 ">
                        {{ subtitle(modelValue) }}
                    </div>
                </template>
            </div>
            <svg class="ms-2 -me-0.5 h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5"></path>
            </svg>
        </div>

        <!-- Full Screen Overlay -->
        <div v-show="open" class="fixed inset-0 z-40" @click="open = false" />

        <Transition enter-active-class="transition ease-out duration-200" enter-from-class="transform opacity-0 scale-95" enter-to-class="transform opacity-100 scale-100" leave-active-class="transition ease-in duration-75" leave-from-class="transform opacity-100 scale-100" leave-to-class="transform opacity-0 scale-95">
            <div v-if="open" class="absolute top-10 left-[-2px] right-[-2px] z-50 border-indigo-500 border-2 border-t-0 bg-gray-900 text-gray-300 rounded-b-md shadow-sm overflow-hidden">
                <input v-if="searchfields" type="search" ref="inputRef" v-model="searchQuery" class="w-full border-gray-700 bg-gray-900 text-gray-300 border-0 border-t focus:ring-0 focus:border-gray-700" />

                <ul class="max-h-96 overflow-y-auto border-gray-700 border-t">
                    <li @click="onSelect(row)" v-for="row in filteredRows" class="py-1.5 pl-3 leading-4 cursor-pointer select-none hover:bg-gray-800">
                        {{ title(row) }}
                        <div class="text-xs text-gray-400">
                            {{ subtitle(row) }}
                        </div>
                    </li>
                    <li v-if="!filteredRows.length" class="py-1.5 pl-3 text-gray-400 italic">No results...</li>
                </ul>
            </div>
        </Transition>
    </div>
</template>
