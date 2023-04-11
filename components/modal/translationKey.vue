<template>
    <div class="modal" :class="{ 'modal-open': openModal}">
        <div class="modal-box">
            <h3 class="font-bold text-lg">Translation Key: {{ mode.toUpperCase() + (copyToNew ? ' to Copy' : '') }}</h3>
            <form action="">
                <div class="form-control w-full max-w-xs">
                    <label class="label">
                        <span class="label-text">Key type:</span>
                    </label>
                    <select v-model="keyType" class="select select-bordered">
                        <option v-for="t in types" :key="t.id" :value="t.id">{{ t.type_name }}</option>
                    </select>
                </div>
                <div class="form-control w-full max-w-xs">
                    <label class="label">
                        <span class="label-text">Key:</span>
                    </label>
                    <input v-model="translationKey" type="text" placeholder="Translation Key"
                           class="input input-bordered"/>
                </div>
                <div v-if="mode === 'edit'" class="form-control w-full max-w-xs mt-2">
                    <label v-if="canBeCopy" class="label cursor-pointer">
                        <span class="label-text">Create new key with this data</span>
                        <input v-model="copyToNew" type="checkbox" class="checkbox"/>
                    </label>
                    <label v-else>(Mapping exist)</label>
                </div>
                <div class="divider">Translations:</div>
                <div v-for="lang in languages" class="form-control w-full max-w-xs" :key="lang.id">
                    <label class="label">
                        <span class="label-text">{{ lang.code }}</span>
                    </label>
                    <input v-model="lang.translation" type="text" :placeholder="`Translation: ${lang.code}`"
                           class="input input-bordered"/>
                </div>
            </form>
            <div class="modal-action">
                <button class="btn mr-1" @click="closeModal">Cancel</button>
                <button v-if="mode === 'create' || copyToNew" class="btn btn-primary"
                        :class="{'loading': loadingCreate}"
                        :disabled="loadingCreate"
                        @click="createKey">Create
                </button>
                <button v-if="mode === 'edit' && copyToNew === false" class="btn btn-primary"
                        :class="{'loading': loadingSaveEdit}"
                        :disabled="loadingSaveEdit"
                        @click="saveKey">Save
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const props = defineProps({
    openModal: {type: Boolean, default: false},
    mode: {type: String, default: 'create'},
    keyToEdit: {
        type: Object, default: () => {
            return {
                id: 0,
                key_name: '',
                type_id: 0
            }
        }
    }
})
const emit = defineEmits(['close'])

const types = reactive([])
const languages = reactive([])
const keyType = ref(0)
const translationKey = ref('')
const loadingCreate = ref(false)
const loadingSaveEdit = ref(false)
const copyToNew = ref(false)
const canBeCopy = ref(true)

watch(() => props.openModal, async (value) => {
    if (!value) {
        return
    }
    const {data: typeData} = await supabase
        .from('key_types')
        .select('id, type_name')
        .order('id', {ascending: true})
    if (typeData) {
        types.push(...typeData)
    }
    if (props.mode === 'create') {
        const {data: languageData} = await supabase
            .from('languages')
            .select('id, code')
            .order('id', {ascending: true})
        if (languageData) {
            languageData.forEach((language) => {
                language.translation = ''
            })
            languages.push(...languageData)
        }
    } else if (props.mode === 'edit') {
        keyType.value = props.keyToEdit.type_id
        translationKey.value = props.keyToEdit.key_name
        const {data: languageData} = await supabase
            .from('translations')
            .select('id, languages(id, code), translation')
            .eq('translation_key_id', props.keyToEdit.id)
            .order('id', {ascending: true})
        if (languageData) {
            languageData.forEach((language) => {
                language.id = language.languages.id
                language.code = language.languages.code
            })
            languages.push(...languageData)
        }

        // 用id 搜尋 key_mapping，沒有被複製過才可以複製
        const {count} = await supabase
            .from('key_mapping')
            .select('id', {count: 'exact', head: true})
            .or(`old_key_id.eq.${props.keyToEdit.id},new_key_id.eq.${props.keyToEdit.id}`,)
        canBeCopy.value = count === 0
    }
})

function validateInput() {
    if (keyType.value === 0) {
        return false
    }
    if (translationKey.value === '') {
        return false
    }
    for (let i = 0; i < languages.length; i++) {
        if (languages[i].translation === '') {
            return false
        }
    }
    return true
}

function closeModal() {
    // 清空資料
    keyType.value = 0
    translationKey.value = ''
    copyToNew.value = false
    canBeCopy.value = false
    types.splice(0)
    languages.splice(0)
    emit('close')
}

async function createKey() {
    if (!validateInput()) {
        alert('Please fill in all fields')
        return
    }

    try {
        loadingCreate.value = true
        const newKeyData = {
            key_type_id: keyType.value,
            key_name: translationKey.value,
            updated_at: new Date(),
        }
        const {data, error} = await supabase.from('translation_keys').insert(newKeyData).select()
        if (error) {
            throw error
        }
        const newKey = data[0]
        const newTranslations = []
        for (let i = 0; i < languages.length; i++) {
            const language = languages[i]
            newTranslations.push({
                translation_key_id: newKey.id,
                language_id: language.id,
                translation: language.translation,
                updated_at: new Date(),
            })
        }
        const {error: translationError} = await supabase
            .from('translations')
            .insert(newTranslations)
        if (translationError) {
            throw translationError
        }
        // 如果copyToNew則新增關聯資料
        if (copyToNew.value && props.keyToEdit.id) {
            const newMapping = {
                old_key_id: props.keyToEdit.id,
                new_key_id: newKey.id,
            }
            const {error: mappingError} = await supabase
                .from('key_mapping')
                .insert(newMapping)

            if (mappingError) {
                throw mappingError
            }
        }

        emit('refresh')
        closeModal()
    } catch (error) {
        alert(error.message)
    } finally {
        loadingCreate.value = false
    }
}

async function saveKey() {
    if (!validateInput()) {
        alert('Please fill in all fields')
        return
    }
    try {
        loadingSaveEdit.value = true
        console.log(languages)
        // update translation key
        const updateKeyData = {
            key_type_id: keyType.value,
            key_name: translationKey.value,
            updated_at: new Date(),
        }
        const {error: keyError} = await supabase
            .from('translation_keys')
            .update(updateKeyData)
            .eq('id', props.keyToEdit.id)
        if (keyError) {
            throw keyError
        }
        // update translations
        const updateTranslations = []
        for (let i = 0; i < languages.length; i++) {
            const language = languages[i]
            updateTranslations.push({
                id: language.id,
                translation_key_id: props.keyToEdit.id,
                language_id: language.id,
                translation: language.translation,
                updated_at: new Date(),
            })
        }
        const {error: translationError} = await supabase
            .from('translations')
            .upsert(updateTranslations)
        if (translationError) {
            throw translationError
        }
        emit('refresh')
        closeModal()
    } catch (error) {
        alert(error.message)
    } finally {
        loadingSaveEdit.value = false
    }
}

</script>