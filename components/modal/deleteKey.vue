<template>
    <div class="modal" :class="{ 'modal-open': openModal}">
        <div class="modal-box">
            <h3 class="font-bold text-lg">Are you sure?</h3>
            <p class="mb-2">Do you want to delete key <span
                    class="bg-gray-300 rounded p-0.5">{{ `${keyToDelete.type_name}.${keyToDelete.key_name}` }}</span> and related translation? It can not
                be recover. </p>
            <p v-for="t in translations" :key="t.id">{{ t.languages.code }}: <span
                    class="bg-gray-300 rounded p-0.5">{{ t.translation }}</span></p>
            <div class="modal-action">
                <button class="btn mr-1" @click="$emit('close')">Cancel</button>
                <button class="btn btn-error" :class="{'loading': loadingDelete}" :disabled="loadingDelete"
                        @click="deleteKey">Delete
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const props = defineProps({
    openModal: {
        type: Boolean,
        default: false
    },
    keyToDelete: {
        type: Object,
        default: () => {
            return {
                id: 0,
                key_name: '',
                type_name: '',
            }
        }
    }
})
const emit = defineEmits(['close', 'refresh'])

const translations = reactive([])
const loadingDelete = ref(false)

const keyNameWithType = computed(() => {
    if(props.keyToDelete.type_name) {
        return `${props.keyToDelete.type_name}.${props.keyToDelete.key_name}`
    }
    return props.keyToDelete.key_name
})

watch(() => props.openModal, async (value) => {
    if (!value || props.keyToDelete.id === 0) {
        return
    }
    // 取得key的所有translation
    const {data} = await supabase
        .from('translations')
        .select('id, languages(code), translation)')
        .eq('translation_key_id', props.keyToDelete.id)
    if (data) {
        translations.splice(0)
        translations.push(...data)
    }
})

async function deleteKey() {
    try {
        // 刪除key_mapping，old_key_id和new_key_id
        const {error: error3} = await supabase
            .from('key_mapping')
            .delete()
            .or(`old_key_id.eq.${props.keyToDelete.id},new_key_id.eq.${props.keyToDelete.id}`,)
        if (error3) throw error3

        // 刪除translations
        const {error} = await supabase
            .from('translations')
            .delete()
            .eq('translation_key_id', props.keyToDelete.id)
        if (error) throw error

        // 刪除translation_key
        const {error: error2} = await supabase
            .from('translation_keys')
            .delete()
            .eq('id', props.keyToDelete.id)
        if (error2) throw error2

        emit('refresh')
        emit('close')
    } catch (error) {
        alert(error.message)
    } finally {
        loadingDelete.value = false
    }

}
</script>
