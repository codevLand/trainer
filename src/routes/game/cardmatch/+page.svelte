<script lang="ts">
  import { emoji } from '../../../lib/emoji'

  type State = 'menu' | 'playing' | 'paused' | 'won' | 'lost'

  const game = {
    name: "Card Match"
  }

  let state: State = 'menu'                  // intial game state
  let grid = createGrid()                    // grid generator
  let player: string
  let maxMatches = grid.length / 2          // check if game is over
  let selected: number[] = []                // selected cards
  let matches: string[] = []                // matched cards
  let timerId: number | null = null
  let time = 5
  let score = 0

  function resetGame() {
    timerId && clearInterval(timerId)
    grid = createGrid()
    maxMatches = grid.length / 2
    selected = []
    matches = []
    timerId = null
    time = 100
    score = 0
  }

  function createGrid(size=20) {
    let cards = new Set<string>()            // initialize with unique cards
    let maxSize = size / 2                  // half because we duplicate the cards

    while (cards.size < maxSize) {
      cards.add(emoji[Math.floor(Math.random() * emoji.length)])          // emoji setter
    }

    return shuffle([...cards, ...cards])    // duplicate and shuffle cards
  }

  function shuffle<Items>(array: Items[]) {
    return array.sort(() => Math.random() - 0.5)  // randomize emoji position on board
  }

  function selectCard(cardIndex: number) { selected = selected.concat(cardIndex) }

  function matchCards() {
    // array destructuring can have any name for the values
    const [first, second] = selected

    if (grid[first] === grid[second]) {
      matches = matches.concat(grid[first])
      score += score + time
    }

    // clear selected
    setTimeout(() => selected = [], 300)
  }

  function gameWon() {
    state = 'won'
    resetGame()
  }

  function gameLost() {
    state = 'lost'
    resetGame()
  }

  function startGameTimer() {
    function countdown() { state !== 'paused' && (time -= 1) }
    timerId = setInterval(countdown, 1000)
  }


  $: selected.length === 2 && matchCards()
  $: maxMatches === matches.length && gameWon()
  $: if (state === 'playing') { !timerId && startGameTimer() } // game is paused
  $: time === 0 && gameLost()

</script>

{#if state === 'menu'}
  <h1>{game.name}</h1>
  <button on:click={() => state = 'playing'}> ⏵︎ Play </button>
{/if}

{#if state === 'playing'}
  <h3> Score: {score} </h3>
  <button on:click={() => state = 'paused'}> ⏸︎ Pause </button>
  <h1 class="timer" class:pulse={time <= 10}> {time} </h1>
  <div class="matches">
    {#each matches as match}
      <div>{match}</div>
    {/each}
  </div>
  <div class="cards">
    {#each grid as card, cardIndex}
      {@const isSelected = selected.includes(cardIndex)}
      {@const isSelectedOrMatch = selected.includes(cardIndex) || matches.includes(card)}
      {@const match = matches.includes(card)}

      <button
        class="card"
        class:selected={isSelected}
        class:flip={isSelectedOrMatch}
        disabled={isSelectedOrMatch}
        on:click={() => selectCard(cardIndex)}>
          <div class="back" class:match> {card} </div>
      </button>

    {/each}
  </div>
{/if}

{#if state === 'paused'}
  <h1>{game.name} is paused</h1>
  <button on:click={() => state = 'playing'}> ⏵︎ Continue </button>
{/if}

{#if state === 'lost'}
  <h1> You lost! 💩 </h1>
  <button on:click={() => state = 'playing'}> ⏵︎ Play again </button>
  <h1>Score: {score}</h1>
{/if}

{#if state === 'won'}
  <h1>You win! 🎉 </h1>
  <button on:click={() => state = 'playing'}> ⏵︎ Play again </button>
  <h1>Score: {score}</h1>
{/if}

<style>
  .matches {
    display: flex;
    gap: 1rem;
    margin-block: 2rem;
    font-size: 3rem;
  }

  .cards {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 0.4rem;
  }

  .card {
    height: 140px;
    width: 140px;
    font-size: 4rem;
    background-color: var(--bg-2);
    transition: rotate 0.3s ease-out;
    transform-style: preserve-3d;

    &.selected {
      border: 4px solid var(--border);
    }

    &.flip {
      rotate: y 180deg;
      pointer-events: none;
    }

    & .back {
      position: absolute;
      inset: 0;
      display: grid;
      place-content: center;
      backface-visibility: hidden;
      rotate: y 180deg;
    }

    & .match {
      transition: opacity 0.3s ease-out;
      opacity: 0.4;
    }
  }

  .timer {
    transition: color 0.3s ease;
  }

  .pulse {
    color: var(--pulse);
    animation: pulse 1s infinite ease;
  }

  @keyframes pulse {
    to {
      scale: 1.4;
    }
  }

</style>