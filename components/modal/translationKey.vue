<template>
    <div class="modal" :class="{ 'modal-open': openModal}">
        <div class="modal-box">
            <h3 class="font-bold text-lg">Translation Key</h3>
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
                <div v-for="lang in languages" class="form-control w-full max-w-xs" :key="lang.id">
                    <label class="label">
                        <span class="label-text">{{ lang.code }}</span>
                    </label>
                    <input v-model="lang.translation" type="text" :placeholder="`Translation: ${lang.code}`"
                           class="input input-bordered"/>
                </div>
            </form>
            <div class="modal-action">
                <button class="btn mr-1" @click="$emit('close')">Cancel</button>
                <button class="btn btn-primary" :class="{'loading': loadingCreate}" :disabled="loadingCreate"
                        @click="createKey">Create
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()

defineProps({openModal: {type: Boolean, default: false}})
const emit = defineEmits(['close'])

const types = reactive([])
const languages = reactive([])
const keyType = ref(0)
const translationKey = ref('')
const loadingCreate = ref(false)

onMounted(async () => {
    const {data: typeData} = await supabase
        .from('key_types')
        .select('id, type_name')
        .order('id', {ascending: true})
    if (typeData) {
        types.push(...typeData)
    }
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
        const { error: translationError} = await supabase
            .from('translations')
            .insert(newTranslations)
        if (translationError) {
            throw translationError
        }
        emit('close')
    } catch (error) {
        alert(error.message)
    } finally {
        loadingCreate.value = false
    }
}

</script>