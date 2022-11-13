<script>
	var randomWords = require('random-words')
	$: dlugosc = 3

	var shift = []
	const calculateShift = () => {
		shift = []
		for (var i = 0; i < slowo.length; i++) {
			const ch = slowo[i]
			shift.push(hasla[i].indexOf(ch))
		}
		console.log({ hasla, shift })
		updateGrid()
	}

	var hasla = []
	var opisy = []
	$: slowo = ''
	async function getDefinitions(hasla) {
		opisy = []
		const promises = []
		hasla.forEach((h) => {
			promises.push(fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${h}`))
		})
		const results = await Promise.all(promises)
		const dataResponses = results.map((result) => result.json())
		const finalData = await Promise.all(dataResponses)

		finalData.forEach((data, index) => {
			opisy.push([])
			data.forEach((def) => {
				opisy[index].push(def.meanings[0].definitions[0].definition)
			})
		})

		opisy = opisy
		calculateShift()
	}

	const newSlowo = () => {
		if (dlugosc < 3) {
			dlugosc = 3
		} else if (dlugosc > 14) {
			dlugosc = 14
		}
		slowo = ''
		while (slowo.length != dlugosc) {
			slowo = randomWords({ exactly: 1, maxLength: dlugosc })[0]
		}
		hasla = []
		for (var i = 0; i < slowo.length; i++) {
			const ch = slowo[i]
			while (true) {
				const haslo = randomWords({ exactly: 1 })[0]
				if (haslo.includes(ch)) {
					hasla.push(haslo)
					break
				}
			}
		}

		getDefinitions(hasla)
	}

	var gridWidth = 1
	var najwShift = 0
	var najwWidthShift = 1
	const updateGrid = () => {
		shift.forEach((s, i) => {
			najwShift = s > najwShift ? s : najwShift
			najwWidthShift = hasla[i].length - s > najwWidthShift ? hasla[i].length - s : najwWidthShift
		})

		gridWidth = najwShift + najwWidthShift
	}
	$: gridHeight = hasla.length ?? 1
</script>

<main>
	<h1>Crossword!</h1>
	<p>Długość krzyżówki: <input type="number" bind:value={dlugosc} /></p>
	<button on:click={newSlowo}>New Crossword</button>
	<h2>Długość: {dlugosc}</h2>
	<h3>Słowo: {slowo}</h3>
	<div class="crossword" style="--grid-width: {gridWidth}; --grid-height: {gridHeight}">
		{#each Array(gridHeight) as _, row}
			{#each Array(gridWidth) as _, column}
				{#if column >= najwShift - shift[row] && column < najwShift + hasla[row].length}
					<input type="text" class="box" />
				{:else}
					<input type="text" class="box" disabled />
				{/if}
			{/each}
		{/each}
	</div>

	{#each opisy as def, index}
		<p>{index + 1}: {def}</p>
	{/each}
</main>

<style>
	.box {
		border: 1px solid #ccc;
		width: 1.75em;
		height: 1.75em;
		text-align: center;
		transition: 0.35s border-color;
	}
	.box:focus-visible {
		border-color: #ff3e00;
		outline: none;
	}
	.box:disabled {
		background-color: #ccc;
	}

	.crossword {
		font-size: 1.5em;
		margin: 2rem auto;
		width: min-content;
		color: #ccc;
		display: grid;
		grid-template-columns: repeat(var(--grid-width), auto);
		grid-template-rows: repeat(var(--grid-height), auto);
		align-content: center;
	}
</style>
