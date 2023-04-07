<template>
    <div class="overflow-x-auto">
        <table class="table w-full" data="" headers="">
            <!-- head -->
            <thead>
            <tr>
                <th></th>
                <th v-for="h in headers" :key="h.title">{{ h.title }}</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(d, rowIndex) in data" :key="d.id">
                <th>{{ (page - 1) * pageSize  + rowIndex + 1 }}</th>
                <td v-for="(h, columnIndex) in headers" :key="`${rowIndex}${columnIndex}`">{{ d[h.value] }}</td>
            </tr>
            </tbody>
        </table>
    </div>
</template>

<script setup>
defineProps({
    headers: {
        type: Array,
        required: true,
        default: () => []
    },
    data: {
        type: Array,
        required: true,
        default: () => []
    },
    page: {
        type: Number,
        required: true,
        default: 1
    },
    pageSize: {
        type: Number,
        required: true,
        default: 10
    }
})
</script>
