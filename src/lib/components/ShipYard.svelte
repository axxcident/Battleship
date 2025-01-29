<script lang="ts">
	import { createEventDispatcher } from "svelte";

	export let currentShip: {size: number, orientation: string};
	export let placedShips: Set<number>;
	export let cellSize: number;

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

	function handleDragStart(ship: any, event: MouseEvent) {
		if (placedShips.has(ship.id)) return;

		isDragging = true;
		draggedShip = ship;

		const shipElement = event.target as HTMLElement;
		const rect = shipElement.getBoundingClientRect();

		dragPosition = {
			x: event.clientX - rect.left,
			y: event.clientY - rect.top,
		};

		// Update the dragged element's style immediately
		if (shipElement) {
			shipElement.style.position = 'fixed';
			shipElement.style.zIndex = '1000';
			shipElement.style.pointerEvents = 'none';
			updateDragPosition(event);
        }

		dispatch("dragstart", {
       	 	ship,
        	grabOffset: dragPosition
    	});
	}

	function updateDragPosition(event: MouseEvent) {
        if (!isDragging || !draggedShip) return;

        const ship = document.getElementById(`ship-${draggedShip.id}`);
        if (ship) {
            ship.style.left = `${event.clientX - dragPosition.x}px`;
            ship.style.top = `${event.clientY - dragPosition.y}px`;
        }
    }

	function handleDrag(event: MouseEvent) {
        updateDragPosition(event);
    }

	function handleDragEnd() {
        if (!isDragging || !draggedShip) return;

        const ship = document.getElementById(`ship-${draggedShip.id}`);
        if (ship) {
            // Reset the ship's style
            ship.style.position = '';
            ship.style.left = '';
            ship.style.top = '';
            ship.style.zIndex = '';
            ship.style.pointerEvents = '';
        }

        isDragging = false;
        draggedShip = null;
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
			{#if !placedShips.has(ship.id)}
				<div
					id="ship-{ship.id}"
					class="ship"
					class:vertical={currentShip.orientation === 'vertical'}
					style="--size: {ship.size}"
					on:mousedown={(e) => handleDragStart(ship, e)}
					role="button"
					aria-roledescription="draggable"
					tabindex="0"
				>
					{ship.name}
				</div>
			{/if}
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
		position: relative;
	}

	.ships {
		display: flex;
		gap: 1rem;
		margin-top: 1rem;
		position: relative;
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

	/* .ship.placed {
		opacity: 0.5;
		cursor: not-allowed;
		pointer-events: none;
	} */
</style>
