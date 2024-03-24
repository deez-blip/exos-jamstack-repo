<script setup lang="ts">
import type { PlayerResponse } from "~/models/player.model"

const { findOne } = useStrapi4()
const route = useRoute()

const { data, pending, error} = await useAsyncData('player', async() => {
    return await findOne<PlayerResponse>(`players/${route.params.slug}`)
})
</script>

<template>
    <div class="player-profile">
        <h1 class="player-name">{{ data.data.first_name }} {{ data.data.last_name }}</h1>
        <div class="player-details">
            <img :src="data.data.image.url" :alt="`${data.data.first_name} ${data.data.last_name}`" class="player-image" />
            <div class="player-info">
                <p class="player-ranking">Ranking: #{{ data.data.ranking }}</p>
                <p class="player-nationality">Nationality: {{ data.data.nationalite }}</p>
                <div class="player-competitions">
                    <h2>Competitions</h2>
                    <ul>
                        <li v-for="competition in data.data.competitions" :key="competition.id">
                            {{ competition.name }}
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.player-profile {
    max-width: 960px;
    margin: 50px auto;
    padding: 40px;
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    text-align: center;
}

.container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    height: 100vh;
    background-color: #f0f0f0;
    padding-top: 100px;
}

.player-profile, .player-info {
    color: #333;
}

.player-image {
    width: 300px; /* Define a fixed width */
    height: 300px; /* Define a fixed height */
    border: 5px solid #ffffff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
    object-fit: contain; /* Cover will ensure the image covers the area, crop if necessary */
    margin-bottom: 20px; /* Added margin for spacing */
}

.player-name {
    font-size: 2.5rem;
    margin-bottom: 20px;
}

.player-ranking, .player-nationality {
    font-size: 1.5rem;
    margin-bottom: 10px;
}

.player-competitions h2 {
    font-size: 1.5rem;
    margin-bottom: 15px;
}

.player-competitions ul {
    list-style-type: none;
    padding: 0;
}

.player-competitions li {
    background-color: #e0e0e0;
    padding: 10px 15px;
    margin-bottom: 10px;
    border-radius: 5px;
}
</style>
