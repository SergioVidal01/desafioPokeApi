<template>
  <div id="app">
    <h1>¿Quién es ese <span class="pokemon">Pokémon</span>?</h1>
    <button v-if="!gameStarted" @click="startGame">Empezar</button>
    <div v-if="gameStarted">
      <div v-if="pokemon">
        <PokemonCard :pokemon="pokemon" @correctGuess="incrementGuessedCount" />
        <p>Pokémon adivinados: <span class="count">{{ guessedCount }}</span></p>
      </div>
      <button @click="loadNewPokemon">Cambiar Pokémon</button>
      <button @click="resetCounter">Reiniciar Juego</button>
      <audio ref="audio" src="../public/whos-that-pokemon_.mp3"></audio>
    </div>
  </div>
</template>



<script>
import axios from 'axios';
import PokemonCard from './components/PokemonCard.vue';

export default {
  components: {
    PokemonCard
  },
  data() {
    return {
      pokemon: null,
      guessedPokemon: new Set(),
      guessedCount: 0,
      gameStarted: false,
      audioPlayed: false
    };
  },
  methods: {
    async loadNewPokemon() {
      try {
        const offset = Math.floor(Math.random() * 1025);
        const response = await axios.get(`https://pokeapi.co/api/v2/pokemon?limit=1&offset=${offset}`);
        const pokemonDetails = await axios.get(response.data.results[0].url);

        this.pokemon = {
          name: response.data.results[0].name,
          image: pokemonDetails.data.sprites.front_default,
          nameGuessed: this.guessedPokemon.has(response.data.results[0].name)
        };

        // Reproducir sonido si ya hubo interacción del usuario
        if (this.audioPlayed) {
          this.$refs.audio.play().catch(error => {
            console.error("Error al reproducir el sonido:", error);
          });
        }
      } catch (error) {
        console.error("Error al cargar los datos de Pokémon:", error);
      }
    },
    incrementGuessedCount(pokemonName) {
      if (!this.guessedPokemon.has(pokemonName)) {
        this.guessedCount++;
        this.guessedPokemon.add(pokemonName);
      }
    },
    resetCounter() {
      this.guessedCount = 0;
      this.guessedPokemon.clear();
      this.loadNewPokemon();
    },
    startGame() {
      this.gameStarted = true;
      this.$nextTick(() => {
        this.loadNewPokemon();
        this.$refs.audio.play().then(() => {
          this.audioPlayed = true;
        }).catch(error => {
          console.error("Error al reproducir el sonido al iniciar el juego:", error);
        });
      });
    }
  }
};
</script>



<style>
@import url('https://fonts.cdnfonts.com/css/pokemon-solid');

h1 {
  margin-top: -20px;
}

#app {
  text-align: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

button {
  margin: 8px;
  padding: 8px 16px;
  background-color: black;
  box-shadow: 10px 5px 5px gray;
  color: white;
  font-size: 16px;
  border-radius: 10px;
}

.pokemon {
  font-size: 36px;
  color: yellow;
  -webkit-text-stroke: 1px blue;
  font-family: 'Pokemon Solid', sans-serif;
  letter-spacing: 4px;
}

.count {
  color: darkkhaki;
  font-weight: bold;
  font-size: 20px;
  margin-top: 10px;
}
</style>

