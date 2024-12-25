<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<section class="layout">
	{@render children()}
</section>

<style>
	:global(body) {
		height: 100vh;
		margin: 0;
		padding: 0;
		display: flex;
		flex-direction: column;
	}

	.layout {
	  flex: 1;
	  background-color: #1a1a1a;
	  padding: 2rem;
	  color: aliceblue;
	}
  </style>
