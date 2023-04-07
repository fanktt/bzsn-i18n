<template>
    <div>
        <h1>Types</h1>
        <div class="my-3 border rounded text-xl">
            <ul class="list-disc m-3 px-3">
                <li v-for="t in types" :key="t.id" class="my-1 mx-1.5">
                    <div class="flex gap-0.5 items-baseline">
                        <div>
                            {{ t.type_name }}
                        </div>
                        <XCircleIcon class="w-4 h-4" @click="confirmDeleteType(t)"/>
                    </div>

                </li>
            </ul>
        </div>
        <div class="flex gap-3">
            <input v-model="newType" placeholder="New type" class="input input-bordered ax-w-xs"/>
            <button class="btn btn-primary" :class="{'loading': loadingCreate}" :disabled="loadingCreate"
                    @click="createType">create
            </button>
        </div>
        <div class="modal" :class="{ 'modal-open': modalDelete}">
            <div class="modal-box">
                <h3 class="font-bold text-lg">Are you sure?</h3>
                <p v-if="typeToDelete" class="py-4">Do you want to delete this type: {{ typeToDelete.type_name }}? It
                    cannot be recover later</p>
                <div class="modal-action">
                    <button class="btn mr-1" @click="cancelDeleteType">Cancel</button>
                    <button class="btn btn-secondary" @click="deleteType">Delete</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import {XCircleIcon} from '@heroicons/vue/24/solid'

const supabase = useSupabaseClient()

const types = reactive([])
const newType = ref('')
const loadingCreate = ref(false)
const modalDelete = ref(false)
const typeToDelete = ref(null)

let {data} = await supabase
    .from('key_types')
    .select('id, type_name')
    .order('id', {ascending: true})

if (data) {
    types.push(...data)
}

async function createType() {
    try {
        loadingCreate.value = true
        const newData = {
            type_name: newType.value,
            updated_at: new Date(),
        }

        let {data, error} = await supabase.from('key_types').insert(newData).select()
        if (error) throw error
        types.push(...data)
        newType.value = ''
    } catch (error) {
        alert(error.message)
    } finally {
        loadingCreate.value = false
    }
}

function confirmDeleteType(type) {
    modalDelete.value = true
    typeToDelete.value = type
}

function cancelDeleteType() {
    modalDelete.value = false
    typeToDelete.value = null
}

// 透過supabase的api刪除資料
function deleteType() {
    // 先檢查這個type沒有被translation_keys使用
    const {data} = supabase.from('translation_keys').select('id').eq('key_type_id', typeToDelete.value.id)
    if (data && data.length > 0) {
        alert('This type is used by translation keys, please delete them first')
        return
    }
    supabase.from('key_types').delete().eq('id', typeToDelete.value.id).then(({data, error}) => {
        if (error) {
            alert(error.message)
        } else {
            const index = types.findIndex((t) => t.id === typeToDelete.value.id)
            types.splice(index, 1)
            modalDelete.value = false
            typeToDelete.value = null
        }
    })

}

</script>

<style scoped>

</style>