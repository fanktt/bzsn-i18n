<template>
    <div>
        <h1>Keys</h1>
        <div class="mt-2">
            <widget-table :headers="headers" :data="tableData" :page="page" :page-size="pageSize"/>
            <div class="btn-group">
                <button class="btn" @click="prevPage">«</button>
                <button class="btn">Page {{ page }}</button>
                <button class="btn" @click="nextPage">»</button>
            </div>
        </div>
    </div>
</template>

<script setup>

const supabase = useSupabaseClient()


const page = ref(1)
const total = ref(0)
const headers = [
    {title: 'Type', value: 'type_name'},
    {title: 'Key', value: 'key_name'},
    {title: 'Updated', value: 'updated_at'},
]
const tableData = reactive([])

const pageSize = 10

watch(page, () => {
    getKeysData()
})

async function getKeysData() {

    tableData.splice(0)
    const {count, data} = await supabase
        .from('translation_keys')
        .select('id, key_types( type_name ), key_name, updated_at', {count: 'exact'})
        .order('id', {ascending: true})
        .range((page.value - 1) * pageSize, page.value * pageSize - 1)
    total.value = count
    if (data) {
        for (let i = 0; i < data.length; i++) {
            const temp = data[i]
            if (temp.key_types) {
                temp.type_name = temp.key_types.type_name
            }
            tableData.push(temp)
        }
    }
}

function prevPage() {
    if (page.value > 1) {
        page.value--
    }
}

function nextPage() {
    if (page.value < Math.ceil(total.value / pageSize)) {
        page.value++
    }
}


getKeysData()
</script>

<style scoped>

</style>