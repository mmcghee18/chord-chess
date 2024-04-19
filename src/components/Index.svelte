<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";
	import { fade, scale } from "svelte/transition";

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
	const getNoteNumber = (note) => {
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
		const octave = parseInt(note.slice(-1)); // Extract the octave number
		const noteName = note.slice(0, -1); // Extract the note letter (and accidental if present)

		const noteIndex = notes.indexOf(noteName); // Get the index of the note in the notes array
		if (noteIndex === -1) {
			throw new Error("Invalid note name");
		}

		return noteIndex + (octave + 1) * 12; // Calculate the MIDI note number
	};
	const generateGrid = (totalNotes, fullWidth, fullHeight) => {
		const cols = Math.ceil(Math.sqrt(totalNotes));
		const rows = Math.ceil(totalNotes / cols);

		const xSpacing = fullWidth / cols;
		const ySpacing = fullHeight / rows;

		const coordinates = [];

		for (let row = 0; row < rows; row++) {
			for (let col = 0; col < cols; col++) {
				const x = col * xSpacing + xSpacing / 2;
				const y = row * ySpacing + ySpacing / 2;
				coordinates.push({ x, y });
				if (coordinates.length >= totalNotes) {
					return coordinates;
				}
			}
		}

		return coordinates;
	};

	const bottom = getNoteNumber("C3");
	const top = getNoteNumber("C6");
	const totalNotes = top - bottom + 1;
	let svgWidth;
	let svgHeight;
	let grid;
	let circles = [];
	let oscillator;
	let audioCtx;

	const handleMidi = (message) => {
		const data = message.data;
		const command = data[0] & 0xf0;
		const note = data[1];
		const velocity = data[2];

		if (command === 0x90 && velocity > 0) {
			const rScale = scaleLinear().domain([0, 127]).range([5, 200]);
			const newCircle = {
				x: grid[note - bottom].x,
				y: grid[note - bottom].y,
				r: rScale(velocity),
				fill: getColor(getNoteName(note).slice(0, -1)),
				ts: Date.now(),
				note: getNoteName(note)
			};
			circles = [...circles, newCircle];

			setTimeout(() => {
				circles = circles.filter((circle) => circle !== newCircle);
			}, 1000);
		}
	};

	const getColor = (note) => {
		const consonant = ["D", "F#", "A", "B"];
		const dissonant = ["C", "C#", "D#", "F", "G#", "A#"];
		const colorful = ["C", "D", "E", "F#", "G", "A", "B"];

		if (consonant.includes(note)) {
			return "rgba(0, 255, 0, 0.5)";
		} else if (dissonant.includes(note)) {
			return "rgba(255, 0, 0, 0.5)";
		} else if (colorful.includes(note)) {
			return "rgba(0, 0, 255, 0.5)";
		} else {
			return "rgba(255, 255, 255, 0.5)";
		}
	};

	const playMelody = () => {
		audioCtx = new (window.AudioContext || window.webkitAudioContext)();
		oscillator = audioCtx.createOscillator();
		oscillator.type = "sine";
		const frequencyD4 = 293.665;
		oscillator.frequency.setValueAtTime(frequencyD4, audioCtx.currentTime);
		oscillator.connect(audioCtx.destination);
		oscillator.start();
	};
	const stopMelody = () => {
		oscillator.stop();
		audioCtx.close();
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

		svgWidth = document.querySelector("svg").clientWidth;
		svgHeight = document.querySelector("svg").clientHeight;
		grid = generateGrid(totalNotes, svgWidth, svgHeight);
	});
</script>

<div>
	<h3>melody: D4</h3>
	<button on:click={playMelody}>play</button>
	<button on:click={stopMelody}>stop</button>
</div>
<svg>
	{#each circles as circle (`${circle.note}-${circle.ts}`)}
		<g in:fade={{ duration: 200 }} out:fade={{ duration: 2000 }}>
			<circle cx={circle.x} cy={circle.y} r={circle.r} fill={circle.fill} />
			<text x={circle.x} y={circle.y} fill="white" text-anchor="middle">
				{circle.note}
			</text>
		</g>
	{/each}
</svg>

<style>
	svg {
		background-color: #222;
		width: 100%;
		height: 100vh;
	}
	text {
		font-family: var(--mono);
		font-size: 14px;
	}
</style>
