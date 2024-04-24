<script>
	import { onMount } from "svelte";

	export let currentNotes;

	const handleMidi = (message) => {
		const data = message.data;
		const command = data[0] & 0xf0;
		const note = data[1];
		const velocity = data[2];
		const noteName = getNoteName(note);

		if (command === 0x90 && velocity > 0) {
			currentNotes = [...currentNotes, noteName];
		} else if (command === 0x80 || (command === 0x90 && velocity === 0)) {
			currentNotes = currentNotes.filter((n) => n !== noteName);
		}
	};

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

	onMount(() => {
		if (navigator.requestMIDIAccess) {
			navigator
				.requestMIDIAccess()
				.then((midiAccess) => {
					console.log("MIDI access obtained.");
					const inputs = midiAccess.inputs.values();
					for (let input of inputs) {
						input.onmidimessage = handleMidi;
					}
				})
				.catch((err) => {
					console.error("MIDI access denied.", err);
				});
		} else {
			console.error("Web MIDI API not supported!");
		}
	});
</script>
