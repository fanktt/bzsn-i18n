<template>
    <div class="flex justify-between">
        <div class="text-2xl">Keys</div>
        <div class="flex">
            <button class="btn btn-primary" @click="openCreateKeyModal">create</button>
        </div>
    </div>
    <div class="flex">
        <div class="form-control mr-2">
            <label class="cursor-pointer label">
                <span class="label-text mr-1">Show replaced</span>
                <input v-model="showReplaced" type="checkbox" class="checkbox checkbox-sm"/>
            </label>
        </div>
        <button class="btn btn-sm" @click="applySearchFilter">apply</button>
    </div>
    <div class="mt-2">
        <widget-table :headers="headers" :data="tableData" :page="page" :page-size="PAGE_SIZE">
            <template #action="{item}">
                <button class="btn btn-sm mr-1" @click="editKey(item)">Edit</button>
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
    <modal-translation-key
            :open-modal="modalKey"
            :mode="modalKeyMode"
            :key-to-edit="keyToEdit"
            @refresh="getKeysData"
            @close="modalKey = false"/>
    <modal-delete-key
            :open-modal="modalDelete"
            :key-to-delete="keyToDelete"
            @refresh="getKeysData"
            @close="closeDeleteModal"/>
</template>

<script setup>

const supabase = useSupabaseClient()

// UI
const page = ref(1)
const total = ref(0)
const headers = reactive([
    {title: 'Type', value: 'type_name'},
    {title: 'Key', value: 'key_name'},
    {title: '中文', value: 'translation'},
])
const modalKey = ref(false)
const modalKeyMode = ref('create')
const modalDelete = ref(false)
const showReplaced = ref(false)

// data
const tableData = reactive([])
const keyToDelete = reactive({id: null, type_name: null, key_name: null})
const keyToEdit = reactive({id: null, type_id: null, key_name: null})

const PAGE_SIZE = 10

watch(page, () => {
    getKeysData()
})


async function getKeysData() {
    const {data: mappingData} = await supabase
        .from('key_mapping')
        .select('old_key_id, new_key_id')
    const mapping = {}
    for (let i = 0; i < mappingData.length; i++) {
        const temp = mappingData[i]
        mapping[temp.old_key_id] = temp.new_key_id
    }


    let query = supabase
        .from('translations_expand')
        .select('id:key_id, key_name, type_id, type_name,  translation', {count: 'exact'})
        .eq('code', 'zh-TW')
        .order('key_id', {ascending: false})
        .range((page.value - 1) * PAGE_SIZE, page.value * PAGE_SIZE - 1)

    const findHeaderReplace = headers.find(item => item.value === 'isReplace') !== undefined
    if (showReplaced.value) {
        if (!findHeaderReplace) {
            headers.push(
                {title: 'Replaced?', value: 'isReplace'})
        }

    } else {
        query = query.filter('replacekey', 'is', 'null')
        if (findHeaderReplace) {
            headers.splice(headers.indexOf(findHeaderReplace), 1)
        }
    }

    const {count, data} = await query
    total.value = count
    if (data) {
        tableData.splice(0)
        for (let i = 0; i < data.length; i++) {
            const temp = data[i]
            if (mapping[temp.id]) {
                temp.isReplace = '✅'
            }
            tableData.push(temp)
        }
    }
}

async function applySearchFilter() {
    page.value = 1
    await getKeysData()
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

function editKey(item) {
    keyToEdit.id = item.id
    keyToEdit.type_id = item.type_id
    keyToEdit.key_name = item.key_name
    modalKeyMode.value = 'edit'
    modalKey.value = true
}

function openCreateKeyModal() {
    modalKeyMode.value = 'create'
    modalKey.value = true
}

function deleteKey(item) {
    keyToDelete.id = item.id
    keyToDelete.key_name = item.key_name
    keyToDelete.type_name = item.type_name
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