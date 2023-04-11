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
        .order('id', {ascending: false})

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
    const json = JSON.stringify(translationJSON)
    const blob = new Blob([json], {type: 'application/json'})
    const href = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = href
    link.download = `${selectLang.value}.json`
    document.body.appendChild(link)
    link.click()
    setTimeout(() => {
        document.body.removeChild(link)
        URL.revokeObjectURL(href)
    }, 1000)

}

</script>

<style scoped>

</style>