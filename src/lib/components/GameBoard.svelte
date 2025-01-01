<script lang="ts">
	import ShipYard from "./ShipYard.svelte";
	import { onMount } from "svelte";

	//board size
	const BOARD_SIZE = 10;
	let cellSize: number;
    let boardElement: HTMLElement;

	let previewCells: {x: number, y: number, isValid: boolean}[] = [];
	let placedShips = new Set<number>();

	// Cell states: 'empty | 'hit' | 'miss' | 'ship'
	export let isPlacementPhase = true;
	export let currentShip = {size: 3, orientation: 'horizontal'}; // example ship

	let hoveredCell: {x:number, y:number} | null = null;
	let draggedShip: any = null;

	let grabOffset: {x: number, y: number} | null = null;

	// Calculate cell size when component mounts and on window resize
	function updateCellSize() {
        if (boardElement) {
            const boardRect = boardElement.getBoundingClientRect();
            cellSize = boardRect.width / BOARD_SIZE;
        }
    }

	// Initialize empty board
	let board = Array(BOARD_SIZE)
		.fill(null)
		.map(() => Array(BOARD_SIZE).fill('empty'));

	function handleDragStart(event: CustomEvent) {
		const ship = event.detail.ship;
		if (placedShips.has(ship.id)) return;

		draggedShip = event.detail.ship;
		grabOffset = event.detail.grabOffset;
	}

	function handleDragEnd() {
		if (hoveredCell && isValidPlacement(hoveredCell.x, hoveredCell.y)) {
			placeShip(hoveredCell.x, hoveredCell.y);
		}
		previewCells = [];
	}

	function handleCellHover(x:number, y:number) {
		if (!grabOffset) return;
		hoveredCell = {x, y};

		if (draggedShip && hoveredCell) {
			const size = draggedShip.size;
			const { orientation } = currentShip;

			// Calculate preview based on grab offset
			const offsetCells = orientation === 'horizontal'
				? Math.floor(grabOffset?.x / cellSize)
				: Math.floor(grabOffset?.y / cellSize);

			const startX = orientation === 'horizontal' ? x : x - offsetCells;
			const startY = orientation === 'horizontal' ? y - offsetCells : y;

			// Generate preview cells
			previewCells = [];

			// First check if the entire placement would be valid
			let isEntirePlacementValid = true;

			// Check bounds
			if (orientation === 'horizontal') {
				if (startY < 0 || startY + size > BOARD_SIZE || startX < 0 || startX >= BOARD_SIZE) {
					isEntirePlacementValid = false;
				}
			} else {
				if (startX < 0 || startX + size > BOARD_SIZE || startY < 0 || startY >= BOARD_SIZE) {
					isEntirePlacementValid = false;
				}
			}

			for (let i = 0; i < size; i++) {
				const newX = orientation === 'horizontal' ? startX : startX + i;
				const newY = orientation === 'horizontal' ? startY + i : startY;

				// Individual cell validity
				let isValid = true;

				// Check bounds and overlap
				if (newX < 0 || newX >= BOARD_SIZE ||
					newY < 0 || newY >= BOARD_SIZE ||
					board[newX]?.[newY] === 'ship' ||
					!isEntirePlacementValid) {
					isValid = false;
				}

				previewCells.push({ x: newX, y: newY, isValid });
			}
		} else {
			previewCells = [];
    	}
	}

	// Clear preview when leaving board or ending drag
	function handleCellLeave() {
		hoveredCell = null;
		previewCells = [];
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
		if (!isValidPlacement(x, y) || !draggedShip || !grabOffset || placedShips.has(draggedShip.id)) return;

		const size = draggedShip.size;
		const { orientation } = currentShip;

		// Calculate the board coordinates from visual position
		const boardX = Math.floor(x);
		const boardY = Math.floor(y);

		// Calculate offsets based on where the ship was grabbed
		const offsetCells = orientation === 'horizontal'
			? Math.floor(grabOffset?.x / cellSize)
			: Math.floor(grabOffset?.y / cellSize);

		// Adjust starting position based on grab offset
		const startX = orientation === 'horizontal' ? boardX : boardX - offsetCells;
		const startY = orientation === 'horizontal' ? boardY - offsetCells : boardY;

		for (let i = 0; i < size; i++) {
			const newX = orientation === 'horizontal' ? startX : startX + i;
			const newY = orientation === 'horizontal' ? startY + i : startY;

			if (newX >= 0 && newX < BOARD_SIZE && newY >= 0 && newY < BOARD_SIZE) {
				board[newX][newY] = 'ship';
			}
		}
		placedShips.add(draggedShip.id);
		board = [...board];
		draggedShip = null;
		hoveredCell = null;
		previewCells = [];
	}

	function isValidPlacement(x:number, y:number) {
		if (!draggedShip || !grabOffset) return false;

		const size = draggedShip.size;
		const { orientation } = currentShip;

		const offsetCells = orientation === 'horizontal'
			? Math.floor(grabOffset?.x / cellSize)
			: Math.floor(grabOffset?.y / cellSize);

		const startX = orientation === 'horizontal' ? x : x - offsetCells;
		const startY = orientation === 'horizontal' ? y - offsetCells : y;

    // Check bounds
		if (orientation === 'horizontal') {
			if (startY < 0 || startY + size > BOARD_SIZE) {
				console.log('out of bounds horizontal');
				return false
			};
			if (startX < 0 || startX >= BOARD_SIZE) {
				console.log('out of bounds vertical');
				return false
			};
		} else {
			if (startX < 0 || startX + size > BOARD_SIZE) {
				console.log('out of bounds vertical');
				return false
			};
			if (startY < 0 || startY >= BOARD_SIZE) {
				console.log('out of bounds horizontal');
				return false
			};
		}

		// Check overlap
		for (let i = 0; i < size; i++) {
			const newX = orientation === 'horizontal' ? startX : startX + i;
			const newY = orientation === 'horizontal' ? startY + i : startY;
			if (newX >= BOARD_SIZE || newY >= BOARD_SIZE) {
				console.log('out of bounds');
				return false
			};
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

	onMount(() => {
        updateCellSize();
        window.addEventListener('resize', updateCellSize);
		console.log('cellSize', cellSize);
        return () => window.removeEventListener('resize', updateCellSize);
    });
</script>

<section class="flex justify-around text-2xl w-[70%] mx-auto">
	<h4 class="">Ships</h4>
	<h4 class="">Radar</h4>
</section>
<section class="boards-container">
	<div class="board player" bind:this={boardElement}>
		{#each board as row, x}
		  <div class="row">
			{#each row as cell, y}
			  <button
				aria-label="cell"
				class="cell"
				class:ship={cell === 'ship'}
				class:hit={cell === 'hit'}
				class:miss={cell === 'miss'}
				class:preview={previewCells.some(p => p.x === x && p.y === y)}
				class:preview-invalid={previewCells.some(p => p.x === x && p.y === y && !p.isValid)}
				on:mouseenter={() => handleCellHover(x, y)}
				on:mouseleave={handleCellLeave}
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

{#if isPlacementPhase}
	<ShipYard
		{currentShip}
		cellSize={cellSize}
		{placedShips}
		on:dragstart={handleDragStart}
		on:dragend={handleDragEnd}
	/>
{/if}

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
		max-height: min(100%, 100vh);
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

	.cell.preview {
		background-color: rgba(255, 255, 255, 0.3);
	}

	.cell.preview-invalid {
		background-color: rgba(255, 0, 0, 0.3);
	}

	/* Optional: add transition for smooth preview */
	.cell {
		transition: background-color 0.15s ease-in-out;
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
