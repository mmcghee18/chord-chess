<script>
	import { onMount } from "svelte";
	import _ from "lodash";

	export let currentNotes;
	export let history;

	const GAP = 100;

	const handleMidi = (message) => {
		const data = message.data;
		const command = data[0] & 0xf0;
		const note = data[1];
		const velocity = data[2];

		if (command === 0x90 && velocity > 0) {
			currentNotes = [...currentNotes, note];

			let originalLength = currentNotes.length;

			setTimeout(() => {
				const newLength = currentNotes.length;
				if (originalLength === newLength) {
					const sortedNotes = _.sortBy(currentNotes);
					history = [...history, sortedNotes];
				}
			}, GAP);
		} else if (command === 0x80 || (command === 0x90 && velocity === 0)) {
			currentNotes = currentNotes.filter((n) => n !== note);
		}
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
