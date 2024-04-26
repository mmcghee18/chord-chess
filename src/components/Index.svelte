<script>
	import Midi from "$components/Midi.svelte";

	let currentNotes = [];
	let history = [];
</script>

<Midi bind:currentNotes bind:history />

<section>
	<div>
		<p>notes being played:</p>
		<ul class="current">
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
</section>

<style>
	section {
		display: flex;
		justify-content: space-between;
		max-width: 600px;
		margin: auto;
	}
	.current {
		color: cornflowerblue;
	}
</style>
