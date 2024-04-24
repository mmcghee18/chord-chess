<script>
	import Midi from "$components/Midi.svelte";

	let currentNotes = [];
</script>

<Midi bind:currentNotes />

<div>
	<p>notes being played:</p>
	<ul>
		{#each currentNotes as note}
			<li>{note}</li>
		{/each}
	</ul>
</div>

<style>
	div {
		max-width: 600px;
		margin: auto;
	}
</style>
