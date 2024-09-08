<script lang="ts">
    import Pointer from "$lib/pointer.svelte";

    const pokemonImage1 = '/pokemon-pngs/charmander.png';
    const pokemonImage2 = '/pokemon-pngs/charmeleon.png';
    const pokemonImage3 = '/pokemon-pngs/charizard.png';

    let name = "Pokémon";

    type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

    let state: State = 'start'
    let pokemonLevel = 1;
    let currentPokemonImage = pokemonImage1;

    function resetGame() {
    state = 'start';
    pokemonLevel = 1;
    currentPokemonImage = pokemonImage1;
}

    function levelUpPokemon() {
        pokemonLevel++;
        if (pokemonLevel === 7) {
            currentPokemonImage = pokemonImage2;
        } else if (pokemonLevel === 14) {
            currentPokemonImage = pokemonImage3;
        }
    }
</script>



{#if state === 'start'}
    <h1>{name.toUpperCase()} Clicking Game</h1>
    <button on:click={() => state = 'playing'}>Play</button>
{/if}

{#if state === 'playing'}
    <h1>Level up your Pokémon!</h1>
    <div class = level-container>
    <p>Current Level: {pokemonLevel}</p>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
    <img src={currentPokemonImage} alt="Pokemon" on:click={levelUpPokemon} />
    </div>
    <button on:click={resetGame}>Back to Start</button>
{/if}

<style>

    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
        height: 100vh; /* Full height to center vertically */
    }

    .level-container {
        display: flex;
        flex-direction: column; /* Stack items vertically */
        align-items: center; /* Center items horizontally */
        margin: 20px 0; /* Add some space around the container */
    }

    img {
        width: 200px;
        height: 200px;
        cursor: pointer;
    }
    
    button {
        margin-top: 20px;
    }
</style>