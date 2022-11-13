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
		hasla = []
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
		document.querySelector('button').disabled = true
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
		document.querySelector('button').disabled = false
	}

	$: tablicaLiterek = [[]]
	var gridWidth = 1
	var najwShift = 0
	var najwWidthShift = 1
	const updateGrid = () => {
		gridWidth = 1
		najwShift = 0
		najwWidthShift = 1
		shift.forEach((s, i) => {
			najwShift = s > najwShift ? s : najwShift
			najwWidthShift = hasla[i].length - s > najwWidthShift ? hasla[i].length - s : najwWidthShift
		})

		gridWidth = najwShift + najwWidthShift

		tablicaLiterek = new Array(gridWidth)
		for (var i = 0; i < gridWidth; i++) {
			tablicaLiterek[i] = Array(gridHeight)
			for (var j = 0; j < gridHeight; j++) {
				tablicaLiterek[i][j] = ''
			}
		}
		console.log(tablicaLiterek)
	}
	$: gridHeight = hasla.length ?? 1

	const letterInput = (e) => {
		const myBox = e.target
		myBox.setSelectionRange(myBox.value.length, myBox.value.length)
		myBox.value = myBox.value[myBox.value.length - 1].toUpperCase()
		console.log(myBox.value)
	}
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
				{#if column >= najwShift - shift[row] && column < najwShift + hasla[row].length - shift[row]}
					{#if column == najwShift}
						<input type="text" class="box answer" on:input={letterInput} bind:value={tablicaLiterek[column][row]} />
					{:else}
						<input type="text" class="box" on:input={letterInput} bind:value={tablicaLiterek[column][row]} />
					{/if}
				{:else}
					<input type="text" class="box" disabled />
				{/if}
			{/each}
		{/each}
	</div>
	<h3>
		Hasło:
		<span class="odpowiedz">
			{#each tablicaLiterek[najwShift] as l}
				{#if l != ''}
					{l}
				{:else}
					{'_'}
				{/if}
			{/each}
		</span>
	</h3>

	{#each opisy as def, index}
		<p>{index + 1}: {def}</p>
	{/each}
</main>

<style>
	.box {
		border: 1px solid #ccc;
		width: 1.8em;
		height: 1.8em;
		text-align: center;
		transition: 0.2s border-color;
	}
	.box:focus-visible {
		border-color: #ff3e00;
		outline: none;
	}
	.box:disabled {
		background-color: #ccc;
		border-radius: 0;
	}

	.box.answer {
		background-color: #dfffff;
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

	.odpowiedz {
		letter-spacing: 0.2em;
		text-transform: capitalize;
	}
</style>
