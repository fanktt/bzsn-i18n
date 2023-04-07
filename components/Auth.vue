<template>
    <div>
        <p class="description py-1">Sign in via magic link with your email below</p>
        <div class="py-1">
            <input v-model="email" class="input input-bordered w-full max-w-xs" type="email" placeholder="Your email"/>
        </div>
        <div class="py-1">
            <button class="btn" :disabled="loading" @click="handleLogin">
                {{ loading ? 'Loading' : 'Send magic link' }}
            </button>
        </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()
const loading = ref(false)
const email = ref("")
const handleLogin = async () => {
    try {
        loading.value = true
        const {error} = await supabase.auth.signInWithOtp({email: email.value})
        if (error) throw error
        alert("Check your email for the login link!")
    } catch (error) {
        alert(error.message)
    } finally {
        loading.value = false
    }
}
</script>