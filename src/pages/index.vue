<template>
    <div>
        <ClientOnly>
            <Sphere />
            <template #fallback>
                ... CARREGANDO
            </template>
        </ClientOnly>
    </div>
</template>

<script setup>

</script>

