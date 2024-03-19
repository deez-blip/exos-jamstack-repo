<script setup lang="ts">
import type { PlayersResponse } from "~/models/player.model"

const { find } = useStrapi()
const { data, pending, error} = await useAsyncData('players', async() => {
    return await find<PlayersResponse>('players', {populate: '*'})
})

</script>

<template>
    <h1>Hello World</h1>
    <template v-if="pending">
        <span>Loading...</span>
    </template>
    <template v-else>
        <pre>{{ data }}</pre>
        <a v-for="player in data?.data" :href="`/players/${player.slug}`">{{ player.first_name }} {{ player.last_name }}</a>
    </template>
</template>

<style scoped></style>