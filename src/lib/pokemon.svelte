<script>
    export let pokemon;
    export let onLevelUp;
  </script>
  
  <div class="pokemon">
    <img src={pokemon.image} alt={pokemon.name} />
    <p>{pokemon.name} - Level: {pokemon.level}</p>
    <button on:click={onLevelUp}>Level Up</button>
  </div>
  
  <style>
    .pokemon {
      border: 1px solid #ccc;
      padding: 10px;
      margin: 10px 0;
    }
    img {
      width: 100px;
      height: 100px;
    }
  </style>