<template>
    <div v-if="user" class="h-full flex">
        <!-- Static sidebar for desktop -->
        <div class="inset-y-0 z-50 flex w-72 h-full">
            <!-- Sidebar component, swap this element with another sidebar if you like -->
            <div class="h-full flex grow flex-col gap-y-5 overflow-y-auto border-r border-gray-200 bg-white px-6 pb-4">
                <div class="flex h-16 shrink-0 items-center">
                    BZSN i18n
                </div>
                <nav class="flex flex-1 flex-col">
                    <ul v-if="user" role="list" class="-mx-2 space-y-1 mb-6">
                        <li class="flex">
                            <div class="mr-1">
                                hi, {{ user.email }}
                            </div>
                            <button @click="signOut">
                                <ArrowRightOnRectangleIcon class="w-6 h-6"/>
                            </button>
                        </li>
                    </ul>
                    <ul role="list" class="-mx-2 space-y-1">
                        <li v-for="item in navigations" :key="item.name">
                            <a :href="item.href"
                               :class="[ routeMatched(item.href) ? 'bg-gray-50 text-indigo-600' : 'text-gray-700 hover:text-indigo-600 hover:bg-gray-50', 'group flex gap-x-3 rounded-md p-2 text-sm leading-6 font-semibold']"
                            >
                                <component :is="item.icon" class=""
                                           :class="[ routeMatched(item.href) ? 'text-indigo-600': 'text-gray-400 group-hover:text-indigo-600', 'h-6 w-6 shrink-0']"
                                />
                                {{ item.name }}
                            </a>
                        </li>
                    </ul>
                </nav>
            </div>
        </div>

        <main class="py-10 px-4 w-full">
            <NuxtPage/>
        </main>
    </div>
    <div v-else class="h-full">
        <div class="grid h-screen place-items-center mx-auto">
            <Auth/>
        </div>
    </div>
</template>

<script setup>
import {
    ArrowDownTrayIcon,
    ArrowRightOnRectangleIcon,
    HomeIcon,
    KeyIcon,
    RectangleGroupIcon,
} from '@heroicons/vue/24/outline'

const user = useSupabaseUser()
const supabase = useSupabaseClient()
const route = useRoute()
const currentPath = ref(route.path)

const navigations = [
    {name: 'Home', href: '/', icon: HomeIcon},
    {name: 'Types', href: '/types', icon: RectangleGroupIcon},
    {name: 'Keys', href: '/keys', icon: KeyIcon},
    {name: 'Export', href: '/export', icon: ArrowDownTrayIcon},
]

function routeMatched(href) {
    return currentPath.value === href
}

async function signOut() {
    try {
        let {error} = await supabase.auth.signOut()
        if (error) throw error
        user.value = null
    } catch (error) {
        alert(error.message)
    }
}
</script>

<style>
html, body, div#__nuxt {
    height: 100%;
}
</style>