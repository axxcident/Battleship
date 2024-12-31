<script lang="ts">
	import { createEventDispatcher } from "svelte";

	export let currentShip: {size: number, orientation: string};

	// Add prop for cell size
	export let cellSize: number; // Match your grid cell size

	export let ships = [
		{ id: 1, size: 5, name: "Carrier" },
		{ id: 2, size: 4, name: "Battleship" },
		{ id: 3, size: 3, name: "Cruiser" },
		{ id: 4, size: 3, name: "Submarine" },
		{ id: 5, size: 2, name: "Destroyer" },
	];

	const dispatch = createEventDispatcher();
	let isDragging = false;
	let draggedShip:any = null;
	let dragPosition = { x: 0, y: 0 };
	let isVertical = false;

	function handleDragStart(ship: any, event: MouseEvent) {
		isDragging = true;
		draggedShip = ship;

		const shipElement = event.target as HTMLElement;
		const rect = shipElement.getBoundingClientRect();

		dragPosition = {
			x: event.clientX - rect.left,
			y: event.clientY - rect.top,
		};

		dispatch("dragstart", {
       	 	ship,
        	grabOffset: dragPosition
    	});
	}

	function handleDrag(event: MouseEvent) {
		if (!isDragging || !draggedShip) return;

		const ship = document.getElementById(`ship-${draggedShip.id}`);

		if ( ship instanceof HTMLElement ) {
			ship.style.left = `${event.clientX - dragPosition.x}px`;
			ship.style.top = `${event.clientY - dragPosition.y}px`;
			ship.style.zIndex = "1000";
			ship.style.opacity = "0.75";
		}
	}

	function handleDragEnd() {
		isDragging = false;
		dispatch("dragend");
	}

	function toggleOrientation() {
		currentShip.orientation = currentShip.orientation === 'horizontal' ? 'vertical' : 'horizontal';
	}
</script>

<div class="shipyard" style="--cell-size: {cellSize}px">
	<button on:click={toggleOrientation}>
		Rotate Ships ({currentShip.orientation})
	</button>

	<div class="ships">
		{#each ships as ship (ship.id)}
			<div
				id="ship-{ship.id}"
				class="ship"
				class:vertical={isVertical}
				class:dragging={isDragging && draggedShip?.id === ship.id}
				style="--size: {ship.size}"
				on:mousedown={(e) => handleDragStart(ship, e)}
				role="button"
				aria-roledescription="draggable"
				tabindex="0"
			>
				{ship.name}
			</div>
		{/each}
	</div>
</div>

<svelte:window
	on:mousemove={handleDrag}
	on:mouseup={handleDragEnd}
/>

<style>
	.shipyard {
		padding: 1rem;
	}

	.ships {
		display: flex;
		gap: 1rem;
		margin-top: 1rem;
	}

	.ship {
		position: relative;
		width: calc(var(--size) * var(--cell-size, 51px));
		height: var(--cell-size, 51px);
		background: #666;
		cursor: move;
		display: flex;
		align-items: center;
		justify-content: center;
		user-select: none;
	}

	.ship.vertical {
        width: var(--cell-size, 51px);
        height: calc(var(--size) * var(--cell-size, 51px));
		/* flex-direction: column; */
	}

	.ship.dragging {
		/* position: absolute; */
		position: fixed;
		pointer-events: none;
	}
</style>
