<script lang="ts">
    import { onDestroy } from 'svelte'; // Import onDestroy for cleanup
    import { pokemonList } from "./Pokemons"
    import { onMount } from 'svelte';
    


    // Define a new interface for leaderboard entries
    interface LeaderboardEntry {
        time: number;
        moves: number;
        player: string;
    }

    // Update the leaderboard variable to use the new interface
    let leaderboard: LeaderboardEntry[] = [];

    // Load leaderboard from localStorage on component mount
    onMount(() => {

        // resetLeaderboard(); // Entfernen Sie diese Zeile nach der ersten Ausführung
        const storedLeaderboard = localStorage.getItem('pokemonMatchLeaderboard');
        if (storedLeaderboard) {
            // Parse the stored leaderboard and filter out any invalid entries
            leaderboard = JSON.parse(storedLeaderboard).filter((entry: LeaderboardEntry) => 
                entry && typeof entry.time === 'number' && 
                typeof entry.moves === 'number' && 
                typeof entry.player === 'string'
            );
        }
    });

    const pairColors = [
    '#FFB3BA', // Light Pink
    '#FFDFBA', // Light Peach
    '#FFBAF3', // Light Purple
    '#B3E5FF', // Light Sky Blues
    '#BAFFC9', // Light Mint Green
    '#FFEBAA', // Light Yellow
    '#FFABAB', // Light Coral
    '#FFC3A0', // Light Salmon
    '#D5AAFF', // Light Lavender
    '#FF677D'  // Light Raspberry
];
    
    
    
    type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

    let state: State = 'start'
    // Select a random Pokémon from the list
    const randomPokemon = pokemonList[Math.floor(Math.random() * pokemonList.length)];


    function shuffleArray(array: any[]) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    function selectAndShufflePokemon() {
        // Select 10 random Pokémon
        const selectedPokemon = shuffleArray([...pokemonList]).slice(0, 10);
        // Double the array to create pairs
        const pairedPokemon = [...selectedPokemon, ...selectedPokemon];
        // Shuffle the pairs
        return shuffleArray(pairedPokemon);
    }


    let gamePokemon = selectAndShufflePokemon();
    let matchedPairs: number[] = [];
    let attempts: number = 0;
    let gameStartTime: number | null = null;
    let elapsedSeconds: number = 0; // Initialize elapsed seconds
    let timerInterval: any;

    let flippedCards: boolean[] = new Array(gamePokemon.length).fill(false);
    let flippedIndices: number[] = [];

function flipCard(index: number) {
    if (flippedIndices.length < 2 && !flippedCards[index] && !matchedPairs.includes(index)) {
        flippedCards[index] = true;
        flippedIndices.push(index);
        flippedCards = [...flippedCards]; // trigger reactivity
        
        if (flippedIndices.length === 2) {
            attempts++;
            checkForMatch(); // Wait 1 second before checking for a match
        }
    }
}


function checkForMatch() {
    const [index1, index2] = flippedIndices;
    if (gamePokemon[index1].id === gamePokemon[index2].id) {
        matchedPairs = [...matchedPairs, index1, index2];
        checkGameCompletion();
        flippedIndices = []; // Reset flipped indices immediately for matched pairs
    } else {
        // For non-matching pairs, use setTimeout
        setTimeout(() => {
            flippedCards[index1] = false;
            flippedCards[index2] = false;
            flippedCards = [...flippedCards]; // trigger reactivity
            flippedIndices = []; // Reset flipped indices
        }, 850);
    }
}



    function startGame() {
        state = 'playing';
        gameStartTime = Date.now(); // Set the start time
        elapsedSeconds = 0; // Reset elapsed seconds
        // Start the timer
    timerInterval = setInterval(() => {
        if (gameStartTime !== null) {
            elapsedSeconds = Math.floor((Date.now() - gameStartTime) / 1000);
        } else {
            clearInterval(timerInterval);
        }
    }, 1000);
    }

    function checkGameCompletion() {
        if (matchedPairs.length === gamePokemon.length) {
            state = 'won'; // Change state to 'won'
            clearInterval(timerInterval); // Stop the timer
         // Create a new leaderboard entry
            const newEntry: LeaderboardEntry = {
                time: elapsedSeconds,
                moves: attempts,
                player: "Anonym"
            };
            
            // Update leaderboard
            leaderboard = [...leaderboard, newEntry]
                .sort((a, b) => a.moves - b.moves)
                .slice(0, 10);
            localStorage.setItem('pokemonMatchLeaderboard', JSON.stringify(leaderboard));
        }
    }

    // if I would like to reset the leader uncomment this function in the onmount function above
    function resetLeaderboard() {
    localStorage.removeItem('pokemonMatchLeaderboard');
    leaderboard = [];
    console.log('Bestenliste wurde zurückgesetzt');
}

</script>



{#if state === 'start'}
    <h1>Pokemon Matching Game</h1>
    <button on:click={startGame}>Play</button>
{/if}

{#if state === 'playing'}
<h2>Match the cards</h2>
<div class="stats">
    <h3>Game Stats</h3>
    <p>Time: {elapsedSeconds} s</p>
    <p>Pairs Matched: {matchedPairs.length / 2} / {gamePokemon.length / 2}</p>
    <p>Attempts: {attempts}</p>
</div>
<div class="game-container">
    <div class="grid">
        {#each gamePokemon as pokemon, index}
        <div class="card" 
     on:click={() => flipCard(index)} 
     style="background-color: {matchedPairs.includes(index) ? pairColors[pokemon.id % pairColors.length] : '#f0f0f0'}">
    <div class="card-content">
        {#if flippedCards[index] || matchedPairs.includes(index)}
            <img src={pokemon.imageUrl} alt={pokemon.name} />
        {:else}
            <img src="https://img.pokemondb.net/sprites/items/poke-ball.png" alt="Pokeball" />
        {/if}
    </div>
</div>
        {/each}
    </div>
</div>
{/if}

{#if state === 'won'}
    <h1>Congratulations!</h1>
    <div class="game-stats">
    <p>You matched all the Pokémon pairs!</p>
    <p>Du hast {attempts} Versuche gebraucht</p>
    <p>Es hat {elapsedSeconds} Sekunden gedauert</p>
    </div>
    <!-- Add this section to display the leaderboard as a table -->
    <h2>Top 10 Bestenliste</h2>
    {#if leaderboard.length > 0}
    <div class="leaderboard-container">
        <table>
            <thead>
                <tr>
                    <th>Rang</th>
                    <th>Züge</th>
                    <th>Zeit (Sekunden)</th>
                    <th>Spieler</th>
                </tr>
            </thead>
            <tbody>
                {#each leaderboard as entry, index}
                    <tr>
                        <td>{index + 1}</td>
                        <td>{entry.moves}</td>
                        <td>{entry.time}</td>
                        <td>{entry.player}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
    {:else}
        <p>Noch keine Einträge in der Bestenliste.</p>
    {/if}

    <button on:click={() => { 
        state = 'start'; 
        // Reset other variables if needed
        matchedPairs = [];
        attempts = 0;
        gamePokemon = selectAndShufflePokemon();
        flippedCards = new Array(gamePokemon.length).fill(false);
        elapsedSeconds = 0; // Reset elapsed time
        gameStartTime = null; // Reset game start time
    }}>Play Again</button>
{/if}


<style>
     h2 {
        text-align: center;
        margin-bottom: 20px;
    }

    .game-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 20px;
    }


    .grid {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 10px;
        max-width: 800px;
        margin: 0 auto;
    }

    .card {
        aspect-ratio: 1; /* This ensures the card is always square */
        border: 1px solid #ccc;
        border-radius: 10px;
        cursor: pointer;
        background-color: #f0f0f0;
        /* Set a fixed size for the cards */
        width: 120px; /* Adjust this value as needed */
        height: 120px; /* This should be the same as the width */
        /*transition: background-color 0.3s ease;*/
    }

    
    .card-content {
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 10px;
        box-sizing: border-box;
    }


    .card img {
        max-width: 100%;
        max-height: 100%;
        object-fit: contain;
    }
    .card.matched {
        background-color: #e0ffe0;  /* light green background for matched cards */
        cursor: default;  /* change cursor to indicate card is no longer clickable */
    }

    .stats {
        width: 200px; /* Fixed width for the stats box */
        padding: 20px;
        margin-left: 20px; /* Space between grid and stats */
        border-radius: 10px;
        /*color: #333;  Dark text for better readability */
        font-family: Arial, sans-serif; /* Use a clear font */
    }

    .leaderboard-container {
        max-height: 400px;
        overflow-y: auto;
        margin-top: 20px;
    }

    .game-stats {
        margin-bottom: 4rem; /* Fügt Abstand unter den Statistiken hinzu */
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 20px;
    }

    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }


</style>