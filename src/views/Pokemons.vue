<script setup>
import { ref, onMounted, computed } from "vue";
import { usePokemonsStore } from "../store/pokemonsStore.js";
import PokeModal from "../components/PokeModal.vue";
import Filters from "../components/Filters.vue";
import FavButton from "../components/FavButton.vue";
import { capitalizeWord } from "../utils/utils.js";

const pokemonsStore = usePokemonsStore();

const pokemonsList = ref([]);
const searchInput = ref("");

const displayModalDetails = ref(false);

onMounted(() => {
  pokemonsList.value = pokemonsStore.pokemons;
});

const filteredPokemons = computed(() => {
  if (!searchInput.value) return pokemonsList.value;
  return pokemonsList.value.filter((pokemon) =>
    pokemon.name.toLowerCase().includes(searchInput.value.toLowerCase())
  );
});

const getPokemonID = (url) => {
  const urlParts = url.split("/");
  return urlParts[urlParts.length - 2];
};

const pokeDetails = computed (() => {
  const { name, height, weight, types, sprites } = pokemonsStore.pokemonDetails;
  const { front_default } = sprites.other["official-artwork"];
  return { name, height, weight, types, image: front_default };
});

const openPokemonDetails = async (pokemonName) => {
  await pokemonsStore.getPokemonDetails(pokemonName);
  displayModalDetails.value = true;
};
</script>

<template>
  <div>
    <div class="pokemon-list">
      <input
        class="pokemon-list__search-input"
        type="text"
        placeholder="Search"
        name="search"
        v-model="searchInput"
      />
      <ul>
        <li
          v-for="pokemon in filteredPokemons"
          :key="getPokemonID(pokemon.url)"
          @click="openPokemonDetails(pokemon.name)"
        >
          <div class="pokemon-list__pokemon">
            <p>{{ capitalizeWord(pokemon.name) }}</p>
            <FavButton
              :pokemon="pokemon"
            />
          </div>
        </li>
      </ul>
      <!-- <Filters
        @filter-all="pokemonsList = filteredPokemons"
        @filter-favorites="pokemonsList = filteredPokemons.filter(pokemon => favoritePokemons[pokemon.name] )"
      /> -->
    </div>
    <PokeModal
      v-if="displayModalDetails"
      :pokeDetails="pokeDetails"
      @close="displayModalDetails = false"
    />
  </div>
</template>

<style scoped lang="scss">
.pokemon-list {
  background-color: #E8E8E8;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;

  &__search-input {
    width: 570px;
    height: 50px;
    border-radius: 5px;
    border: none;
    padding: 0 20px;
    margin: 2rem 0;
    color: #BFBFBF;
  }

  ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
    li {
      cursor: zoom-in;
      transition: background-color 0.3s ease;

      &:active {
        transform: scale(0.98);
      }
    }
  }

  &__pokemon {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    width: 570px;
    height: 60px;
    background-color: #FFFFFF;
    border-radius: 5px;
    padding: 0 20px;
    margin-bottom: 0.5rem;
    &:hover {
      background-color: #BFBFBF;
    }
  }
}
</style>