<script>
	import Midi from "$components/Midi.svelte";

	let currentNotes = [];
	let history = [];
</script>

<Midi bind:currentNotes bind:history />

<div>
	<p>notes being played:</p>
	<ul>
		{#each currentNotes as note}
			<li>{note}</li>
		{/each}
	</ul>
</div>

<div>
	<p>chord history:</p>
	<ul>
		{#each history as chord}
			<li>{chord.join(", ")}</li>
		{/each}
	</ul>
</div>

<style>
	div {
		max-width: 600px;
		margin: auto;
	}
</style>
