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

<section class="">
	<div class="board flex flex-col gap-2 h-auto border-slate-800 border-solid border-2 bg-blue-400">
		{#each board as row, x}
			<div class="row w-full h-full flex justify-center gap-2">
				{#each row as cell, y}
					<button
						aria-label="cell"
						class="cell bg-blue-800 rounded-md"
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

	.cell {
		height: clamp(30px, 8vw, 65px);
		width: clamp(30px, 8vw, 65px);
	}
</style>
