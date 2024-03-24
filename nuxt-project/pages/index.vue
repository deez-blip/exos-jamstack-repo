<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import type { Player, PlayersResponse } from '~/models/player.model';
import type { Competition, CompetitionsResponse } from '~/models/competition.model';

const { find } = useStrapi();
const selectedNationality = ref('');
const selectedCompetition = ref('');
const sortDirection = ref('ascending');
const page = ref(1);
const pageSize = ref(3);
const searchQuery = ref('');

// Réinitialiser la page à 1 chaque fois que l'ordre de tri change
watch(sortDirection, () => {
    page.value = 1;
});

// Réinitialiser la page à 1 chaque fois que la requête de recherche change
watch(searchQuery, () => {
    page.value = 1;
});

const playersQueryParams = computed(() => ({
    populate: '*',
    pagination: {
        page: page.value,
        pageSize: pageSize.value
    },
    sort: sortDirection.value === 'ascending' ? 'ranking:asc' : 'ranking:desc',
    filters: {
        competitions: {
            name: {
                $in: selectedCompetition.value !== '' ? selectedCompetition.value : []
            }
        },
        nationalite: {
            $in: selectedNationality.value !== '' ? selectedNationality.value : []
        },
        ...(searchQuery.value && {
            $or: [
                { first_name: { $containsi: searchQuery.value } },
                { last_name: { $containsi: searchQuery.value } }
            ]
        })
    }
}));

const { data: playersData, pending: playersPending, error: playersError } = useAsyncData(
    'players',
    () => find<PlayersResponse>('players', playersQueryParams.value),
    {
        watch: [page, pageSize, selectedCompetition, selectedNationality, sortDirection, searchQuery]
    }
);

const { data: competitionsData, pending: competitionsPending, error: competitionsError } = await useAsyncData('competitions', async () => {
    return await find<CompetitionsResponse>('competitions');
});

// Calculer le nombre de pages
const pageCount = computed(() => {
    const totalItems = playersData.value?.meta.pagination.total || 0;
    return Math.ceil(totalItems / pageSize.value);
});

// Computed property pour les joueurs filtrés et triés
const sortedFilteredPlayers = computed(() => {
    if (!playersData.value || !playersData.value.data) {
        return [];
    }

    let players = playersData.value.data.filter((player: Player) => {
        const matchesNationality = player.nationalite === selectedNationality.value || selectedNationality.value === '';
        const matchesCompetition = player.competitions.some(competition => competition.name === selectedCompetition.value) || selectedCompetition.value === '';
        const matchesSearchQuery = player.first_name.toLowerCase().includes(searchQuery.value.toLowerCase()) || player.last_name.toLowerCase().includes(searchQuery.value.toLowerCase());
        return matchesNationality && matchesCompetition && matchesSearchQuery;
    });

    if (sortDirection.value === 'ascending') {
        players.sort((a, b) => a.ranking - b.ranking);
    } else if (sortDirection.value === 'descending') {
        players.sort((a, b) => b.ranking - a.ranking);
    }

    return players;
});

</script>

<template>
    <div class="container">
        <h1>Liste des Joueurs</h1>
        <!-- Debbuging -->
        <!-- <pre>{{ playersData }}</pre> -->
        <!-- <pre>{{ competitionsData }}</pre> -->
        <div class="filters">
            <!-- Nationality Filter -->
            <div class="filter">
                <label for="country-select">Nationalité :</label>
                <select name="country" id="country-select" v-model="selectedNationality">
                    <option value="">--Please choose an option--</option>
                    <option value="France">France</option>
                    <option value="Chine">Chine</option>
                    <option value="Japon">Japon</option>
                </select>
            </div>

            <!-- Competition Filter -->
            <div class="filter">
                <label for="competitions-select">Compétitions :</label>
                <select name="competition" id="competitions-select" v-model="selectedCompetition">
                    <option value="">--Please choose an option--</option>
                    <option v-for="competition in competitionsData?.data" :key="competition.id"
                        :value="competition.name">{{ competition.name }}</option>
                </select>
            </div>

            <!-- Sort Direction Filter -->
            <div class="filter">
                <label for="sort-select">Trier par classement :</label>
                <select name="sort" id="sort-select" v-model="sortDirection">
                    <option value="ascending">Croissant</option>
                    <option value="descending">Décroissant</option>
                </select>
            </div>

            <div class="filter">
                <label for="search-input">Chercher un joueur :</label>
                <input type="text" id="search-input" v-model="searchQuery" placeholder="Entrez le nom du joueur">
            </div>
        </div>

        <section v-if="playersPending || competitionsPending">
            <span>Loading...</span>
        </section>

        <section v-else-if="playersError || competitionsError">
            <span>Error: {{ playersError || competitionsError }}</span>
        </section>

        <section v-else>
            <div class="player-list">
                <div v-for="player in sortedFilteredPlayers" :key="player.id" class="player-card">
                    <img :src="player.image.url" :alt="`Image de ${player.first_name} ${player.last_name}`"
                        class="player-image" />
                    <div class="player-info">
                        <h2 class="player-name">
                            <a :href="`/players/${player.slug}`">
                                {{ player.first_name }} {{ player.last_name }}
                            </a>
                        </h2>
                        <p class="player-ranking">Classement: #{{ player.ranking }}</p>
                    </div>
                </div>
            </div>
            <div class="pagination-container">
                <UPagination v-if="playersData?.meta" v-model="page" :page-count="pageCount"
                    :total="playersData?.meta.pagination.total" class="mx-auto mt-8" />
            </div>
        </section>
    </div>
</template>

<style scoped>
a {
    color: #006064;
    text-decoration: none;
    font-weight: bold;
}

h1 {
    color: black;
}

.container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    background-color: transparent;
    padding: 20px;
    background-color: #f0f0f0;
}

.filters {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 20px;
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.filters,
.pagination-container {
    display: flex;
    justify-content: space-between;
    gap: 20px;
    align-items: center;
}

/* Centre la pagination */
.pagination-container {
    justify-content: center;
    /* Centre les enfants de .pagination-container */
}

.filter label {
    display: block;
    margin-bottom: 5px;
    color: #333;
}

.filter select,
.filter input {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
}

.player-list {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.player-card {
    width: calc(33.333% - 20px);
    background-color: #ffffff;

    border: 1px solid #ddd;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
    transition: transform 0.2s ease-in-out;

}

.player-card:hover {
    transform: translateY(-5px);

}

.player-image {
    width: 100%;
    height: 200px;
    display: block;
    object-fit: contain;

}

.player-info {
    padding: 15px;
    text-align: center;
}

.player-name {
    font-size: 20px;
    margin: 0 0 10px 0;
}

.player-ranking {
    font-size: 16px;
    color: #00838f;

}
</style>