<template>
    <div>
        <h1 class="text-2xl">Export</h1>
        <div>
            <div class="form-control w-full max-w-xs mt-3">
                <select v-model="selectLang" class="select select-bordered">
                    <option v-for="lang in languages" :key="lang.id" :value="lang.code">{{ lang.code }}</option>
                </select>
            </div>
            <div class="form-control w-full max-w-xs mt-2">
                <label class="label cursor-pointer">
                    <span class="label-text">Export new keys only (replace by mapping)</span>
                    <input v-model="exportNewKeyOnly" type="checkbox" class="checkbox"/>
                </label>
            </div>
            <button class="btn mt-3" @click="exportJSON">
                export
            </button>
            <button class="btn mt-3 ml-3" @click="exportAllJSON">
                export all
            </button>
            <div class="divider"></div>
            <button class="btn mt-3" @click="exportMapping">
                export mapping
            </button>
        </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const languages = reactive([])
const selectLang = ref('zh-TW')
const exportNewKeyOnly = ref(true)

onMounted(async () => {
    const {data: languageData} = await supabase
        .from('languages')
        .select('id, code')
        .order('id', {ascending: true})
    if (languageData) {
        languages.push(...languageData)
    }
})

async function exportJSON() {
    // 取得所有key和translation
    const {data: translationData} = await supabase
        .from('translations_expand')
        .select('id, code, type_id, type_name, key_id, key_name, translation, replacekey', {count: 'exact'})
        .eq('code', selectLang.value)
        .order('type_id, key_name', {ascending: true})

    const translationJSON = {}
    for (let i = 0; i < translationData.length; i++) {
        const item = translationData[i]
        // 如果有replacekey且只匯出新key則跳過
        if (exportNewKeyOnly.value && item.replacekey !== null) {
            continue
        }
        // 如果有type則加入type object，key和translation加入type object
        if (item.type_id !== null) {
            // 判斷translationJSON是否已經有該type object
            const typeName = item.type_name.toLowerCase()
            if (translationJSON[typeName] === undefined) {
                translationJSON[typeName] = {}
            }
            translationJSON[typeName][item.key_name] = item.translation
        } else {
            translationJSON[item.key_name] = item.translation
        }
    }

    // 將translationData轉為JSON下載
    downloadJSON(translationJSON, `${selectLang.value}.json`)
}

async function exportAllJSON() {
    // for each language
    for (let i = 0; languages.length > i; i++) {
       selectLang.value = languages[i].code
        await exportJSON()
    }
}

function downloadJSON(jsonObject, fileName) {
    const json = JSON.stringify(jsonObject, null, 2)
    const blob = new Blob([json], {type: 'application/json'})
    const href = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = href
    link.download = fileName
    document.body.appendChild(link)
    link.click()
    setTimeout(() => {
        document.body.removeChild(link)
        URL.revokeObjectURL(href)
    }, 1000)
}

async function exportMapping() {
    const {data: mappingData} = await supabase
        .from('keys_replacement')
        .select('old_type_name, old_key_name, new_type_name, new_key_name')
    const mappingJSON = {}
    mappingData.forEach((item) => {
        if (item.old_type_name !== null) {
            mappingJSON[`${item.old_type_name}.${item.old_key_name}`] = `${item.new_type_name}.${item.new_key_name}`
        } else {
            mappingJSON[item.old_key_name] = `${item.new_type_name}.${item.new_key_name}`
        }
    })
    downloadJSON(mappingJSON, `replace_key.json`)
}

</script>

<style scoped>

</style>