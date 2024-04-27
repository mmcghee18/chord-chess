<script>
	import Midi from "$components/Midi.svelte";

	let currentNotes = [];
	let history = [];

	const getNoteName = (noteNumber) => {
		const notes = [
			"C",
			"C#",
			"D",
			"D#",
			"E",
			"F",
			"F#",
			"G",
			"G#",
			"A",
			"A#",
			"B"
		];
		const octave = Math.floor(noteNumber / 12) - 1;
		const noteIndex = noteNumber % 12;
		return notes[noteIndex] + octave;
	};
	const getDistance = (chordA, chordB) => {
		let total = 0;
		const longer = chordA.length > chordB.length ? chordA : chordB;
		const shorter = chordA.length > chordB.length ? chordB : chordA;

		total = longer.reduce((acc, current, i) => {
			if (i < shorter.length) {
				return acc + Math.abs(current - shorter[i]);
			}
			return acc;
		}, 0);
		return total;
	};
</script>

<Midi bind:currentNotes bind:history />

<section>
	<div>
		<p>notes being played:</p>
		<ul class="current">
			{#each currentNotes as note}
				<li>{getNoteName(note)}</li>
			{/each}
		</ul>
	</div>

	<div>
		<p>chord history:</p>
		<ul>
			{#each history as chord, i}
				{@const distance = i > 0 ? getDistance(chord, history[i - 1]) : 0}
				<li>{chord.join(", ")} ({distance})</li>
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
