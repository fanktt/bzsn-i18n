<template>
    <div>
        <h1>Keys</h1>
        <div class="mt-2">
            <widget-table :headers="headers" :data="tableData"/>
        </div>
    </div>
</template>

<script setup>
const headers = [
    {title: 'Name', value: 'name'},
    {title: 'Job', value: 'job'},
    {
        title: 'FavoriteColor',
        value: 'favoriteColor'
    }
    ]
const tableData = [
    {
        id: 1,
        name: 'Cy Ganderton',
        job: 'Quality Control Specialist',
        favoriteColor: 'Blue'
    },
    {
        id: 2,
        name: 'Hart Hagerty',
        job: 'Desktop Support Technician',
        favoriteColor: 'Purple'
    },
    {
        id: 3,
        name: 'Brice Swyre',
        job: 'Tax Accountant',
        favoriteColor: 'Red'
    }
]
</script>

<style scoped>

</style>