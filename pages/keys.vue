<template>
    <div class="flex justify-between">
        <div class="text-2xl">Keys</div>
        <div>
            <button class="btn btn-primary" @click="modalCreate = true">create</button>
        </div>
    </div>
    <div class="mt-2">
        <widget-table :headers="headers" :data="tableData" :page="page" :page-size="PAGE_SIZE">
            <template #action="{item}">
                <button class="btn btn-sm mr-1" @click="editKey(item.id)">Edit</button>
                <button class="btn btn-sm btn-error" @click="deleteKey(item)">Delete</button>
            </template>
        </widget-table>
        <div class="mt-2 flex justify-between">
            <div>Total: {{ total }}</div>
            <div class="btn-group">
                <button class="btn" @click="prevPage">«</button>
                <button class="btn">Page {{ page }}</button>
                <button class="btn" @click="nextPage">»</button>
            </div>
        </div>
    </div>
    <modal-translation-key :open-modal="modalCreate" @close="modalCreate = false"/>
    <modal-delete-key
        :open-modal="modalDelete"
        :key-to-delete="keyToDelete"
        @refresh="getKeysData"
        @close="closeDeleteModal"/>
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
const modalCreate = ref(false)
const modalDelete = ref(false)
const keyToDelete = reactive({id: null, key_name: null})

const PAGE_SIZE = 10

watch(page, () => {
    getKeysData()
})

async function getKeysData() {

    const {count, data} = await supabase
        .from('translation_keys')
        .select('id, key_types( type_name ), key_name, updated_at', {count: 'exact'})
        .order('id', {ascending: false})
        .range((page.value - 1) * PAGE_SIZE, page.value * PAGE_SIZE - 1)
    total.value = count
    if (data) {
        tableData.splice(0)
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
    if (page.value < Math.ceil(total.value / PAGE_SIZE)) {
        page.value++
    }
}

function editKey(itemId) {
    console.log(itemId)
}

function deleteKey(item) {
    keyToDelete.id = item.id
    keyToDelete.key_name = item.key_name
    modalDelete.value = true
}

function closeDeleteModal() {
    keyToDelete.value = {id: 0, key_name: ''}
    modalDelete.value = false
}

getKeysData()
</script>

<style scoped>

</style>