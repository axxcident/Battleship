<script lang="ts">
	import { createEventDispatcher } from "svelte";

	export let currentShip: {size: number, orientation: string};

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
		currentShip.size = ship.size;
		const rect = (event.target as HTMLElement).getBoundingClientRect();
		dragPosition = {
			x: event.clientX - rect.left,
			y: event.clientY - rect.top,
		};

		dispatch("dragstart");
	}

	function handleDrag(event: MouseEvent) {
		const ship = document.getElementById(`ship-${draggedShip.id}`);
		if (!isDragging) return;

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

<div class="shipyard">
	<button on:click={toggleOrientation}>
		Rotate Ships ({currentShip.orientation})
	</button>

	<div class="ships">
		{#each ships as ship (ship.id)}
			<div
				id="ship-{ship.id}"
				class="ship"
				class:vertical={isVertical}
				class:dragging={isDragging && ship.id === draggedShip?.id}
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
		width: calc(var(--size) * 40px);
		height: 40px;
		background: #666;
		cursor: move;
		display: flex;
		align-items: center;
		justify-content: center;
		user-select: none;
	}

	.ship.vertical {
		width: 40px;
		height: calc(var(--size) * 40px);
		/* flex-direction: column; */
	}

	.ship.dragging {
		position: absolute;
		pointer-events: none;
	}
</style>
