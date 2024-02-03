<script lang="ts">
	import { word } from './word'

	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

	let state: State = 'start'
	let size = 12
	let grid = createGrid()
	let maxMatches = grid.length / 2
	let selected: number[] = []
	let matches: string[] = [' ']
	let timerId: number | null = null
	let time = 90

	function createGrid() {
		// only want unique cards
		let cards = new Set<string>()
		// half because we duplicate the cards
		let maxSize = size / 2

		while (cards.size < maxSize) {
			// pick random word
			const randomIndex = Math.floor(Math.random() * word.length)
			cards.add(word[randomIndex])
		}

		// duplicate and shuffle cards
		return shuffle([...cards, ...cards])
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5)
	}

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1)
		}
		timerId = setInterval(countdown, 1000)
	}

	function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex)
	}

	function matchCards() {
		// array destructuring can have any name for the values
		const [first, second] = selected

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first])
		}

		// clear selected
		setTimeout(() => (selected = []), 300)
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused'
					break
				case 'paused':
					state = 'playing'
					break
			}
		}
	}

	function resetGame() {
		timerId && clearInterval(timerId)
		grid = createGrid()
		maxMatches = grid.length / 2
		selected = []
		matches = []
		timerId = null
		time = 90
	}

	function gameWon() {
		state = 'won'
	}

	function gameLost() {
		state = 'lost'
		resetGame()
	}

	$: if (state === 'playing') {
		//	in case you pause the game
		!timerId && startGameTimer()
	}

	$: selected.length === 2 && matchCards()
	$: maxMatches === matches.length-1 && gameWon()
	$: time === 0 && gameLost()
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'start'}
	<h1>Learn Trovi</h1>
	<button on:click={() => (state = 'playing')}>Play</button>
	<section>
		<p>Know more at <br> <strong><a style=" color: inherit; text-decoration: none;" href="https://t.me/boidcommunity/34/430">Boid Community: The Ark</a></strong></p>
		<a href="https://t.me/boidcommunity/34/430"><img src="https://blog.boid.com/content/images/2023/03/logo-boid_500x500px_neg.png" alt="boid"></a>
	</section>
	
{/if}

{#if state === 'paused'}
	<h1>Game paused</h1>
{/if}

{#if state === 'playing'}
	<h1 class="timer" class:pulse={time <= 10}>
		{time}
	</h1>

	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
	</div>

	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatch =
				selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}

			<button
				class="card"
				class:selected={isSelected}
				class:flip={isSelectedOrMatch}
				disabled={isSelectedOrMatch}
				on:click={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>
					{card}
				</div>
			</button>
		{/each}
	</div>
{/if}

{#if state === 'lost'}
	<h1>You lost!</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
{/if}

{#if state === 'won'}
	<h1>You win!</h1>
	<button on:click={() => { state = 'playing'; resetGame(); }}>Play again</button>
	<div class="matches">
		{#each matches as card}
			<div>{card}  </div>
		{/each}
	</div>
	<div class="matches trovi">
		{#each matches as card}
			<div>{card}  </div>
		{/each}
	</div>
{/if}

<style>

@font-face {
    font-family: 'Trovi';
    src: url('/TroviDialect-Regular.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
  }

	.cards {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 70px;
		width: 200px;
		font-family: 'Trovi', sans-serif;
		font-size: 1.5rem;
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

	.matches {
		gap: 0.5rem;
		margin-block: 2rem;
		margin-bottom: 0.5rem;
		display: flex;
		align-items: center;
		justify-content:center;
    	gap: 1rem;
    	font-size: 2rem;
	}

	.trovi{
		font-family: 'Trovi', sans-serif;
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
