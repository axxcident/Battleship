<script lang="ts">
	import { onMount } from "svelte";

	//board size
	const BOARD_SIZE = 10;

	// Initialize empty board
	let board = Array(BOARD_SIZE)
		.fill(null)
		.map(() => Array(BOARD_SIZE).fill('empty'));
	console.log(board);

	// Cell states: 'empty | 'hit' | 'miss' | 'ship'
	export let isPlacementPhase = true;
	export let currentShip = {size: 3, orientation: 'horizontal'}; // example ship

	let hoveredCell: {x:number, y:number} | null = null;

	function handleCellHover(x:number, y:number) {
		hoveredCell = {x, y};
	}

	function handleCellClick(x:number, y:number) {
		if (isPlacementPhase) {
			// place ship logic
			placeShip(x,y);
		} else {
			// fire shot logic
			fireShot(x,y);
		}
	}

	function placeShip(x: number, y: number) {
		if (!isValidPlacement(x, y)) return;

		const { size, orientation } = currentShip;
		for (let i = 0; i < size; i++) {
			const newX = orientation === 'horizontal' ? x : x + i;
			const newY = orientation === 'horizontal' ? y + i : y;
			board[newX][newY] = 'ship';
		}
		board = [...board];
	}

	function isValidPlacement(x:number, y:number) {
		const { size, orientation } = currentShip;

		//check if ship goes out of bounds
		if (orientation === 'horizontal' && y + size > BOARD_SIZE) return false;
		if (orientation === 'vertical' && x + size > BOARD_SIZE) return false;

		// check if ship overlaps with another ship
		for (let i = 0; i < size; i++) {
			const newX = orientation === 'horizontal' ? x : x + i;
			const newY = orientation === 'horizontal' ? y + i : y;
			if (board[newX][newY] === 'ship') return false;
		}

		return true;
	}

	function fireShot(x:number, y:number) {
		if (board[x][y] === 'ship') {
			board[x][y] = 'hit';
		} else if (board[x][y] === 'empty') {
			board[x][y] = 'miss';
		}
		board = [...board];
	}

</script>

<section class="flex justify-around text-2xl w-[70%] mx-auto">
	<h4 class="">Ships</h4>
	<h4 class="">Radar</h4>
</section>
<section class="boards-container">
	<div class="board player">
		{#each board as row, x}
		  <div class="row">
			{#each row as cell, y}
			  <button
				aria-label="cell"
				class="cell"
				class:ship={cell === 'ship'}
				class:hit={cell === 'hit'}
				class:miss={cell === 'miss'}
				on:mouseenter={() => handleCellHover(x, y)}
				on:mouseleave={() => hoveredCell = null}
				on:click={() => handleCellClick(x, y)}
			  ></button>
			{/each}
		  </div>
		{/each}
	</div>
	<div class="board radar">
	  {#each board as row, x}
		<div class="row">
		  {#each row as cell, y}
			<button
			  aria-label="cell"
			  class="cell"
			  class:ship={cell === 'ship'}
			  class:hit={cell === 'hit'}
			  class:miss={cell === 'miss'}
			  on:mouseenter={() => handleCellHover(x, y)}
			  on:mouseleave={() => hoveredCell = null}
			  on:click={() => handleCellClick(x, y)}
			></button>
		  {/each}
		</div>
	  {/each}
	</div>
</section>

<style>

	.boards-container {
		display: flex;
		flex-direction: row;
		gap: 1.5rem;
		height: 100%;
		width: 70%;
		padding: 1rem;
		box-sizing: border-box;
		margin: 0 auto;
	}

	.board {
		flex: 1;
		aspect-ratio: 1;
		height: 100%;
		max-height: min(100%, 100vw);
	}

	.radar {
		background-color: #1b4d3e;
	}

	.player {
		background-color: #0066cc;
	}

	.row {
		display: grid;
		grid-template-columns: repeat(10, 1fr);
	}

	.cell {
		aspect-ratio: 1;
		border: 1px solid hsla(0, 0%, 100%, 0.2);
		background: none;
		position: relative;
	}

	.cell::before {
		content: '+';
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		color: rgba(255, 255, 255, 0.7);
		font-size: 0.8rem;
	}

	.cell.ship {
		background-color: white;
	}

	.cell.hit {
		background-color: red;
	}

	.cell.miss {
		background-color: gray;
	}
</style>
