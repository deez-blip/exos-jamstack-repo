<script setup lang="ts">
import type { PlayersResponse } from "~/models/player.model"

const { find } = useStrapi()
const { data, pending, error} = await useAsyncData('players', async() => {
    return await find<PlayersResponse>('players', {populate: '*'})
})

</script>

<template>
    <h1>Hello World</h1>
    <pre>{{ data }}</pre>
</template>

<style scoped></style>