<template>
    <div>
        <NuxtLayout/>
    </div>
</template>

<script setup>
const store = useGlobalStore()

onMounted(() => {
    // fonction pour vérifié si l'utilisateur était connecté à sa dernière visite
    const token = localStorage.getItem("Token de Connection")
    store.setToken(JSON.parse(token))
})
</script>