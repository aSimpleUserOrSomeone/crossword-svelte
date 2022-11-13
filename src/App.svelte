<script>
	import { identity } from 'svelte/internal'

	var randomWords = require('random-words')
	var isWin = false
	var dlugosc = 3
	var startTime = Date.now()
	var endTime = 0
	var shift = []
	var hasla = []
	var opisy = []
	var slowo = ''

	const newSlowo = () => {
		if (document.querySelector('.odpowiedz')) document.querySelector('.odpowiedz').classList.remove('poprawna')
		startTime = Date.now()
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

		getDefinitions()
		document.querySelector('button').disabled = false
	}
	async function getDefinitions() {
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
		console.log(`Podpowiedź`, hasla)
		calculateShift()
	}
	const calculateShift = () => {
		shift = []
		for (var i = 0; i < slowo.length; i++) {
			const ch = slowo[i]
			shift.push(hasla[i].indexOf(ch))
		}
		updateGrid()
	}

	var tablicaLiterek = [[]]
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
		checkWin()
	}
	$: gridHeight = hasla.length ?? 1

	const lockTiles = () => {
		const boxes = document.querySelectorAll('.box:not(:disabled)')
		boxes.forEach((e) => {
			e.style = 'pointer-events: none;'
			if (e.classList.contains('answer')) {
				e.style = 'color: #ffa081; pointer-events: none'
				e.disabled = true
				document.querySelector('input[type=number]').focus()
			}
		})
	}
	const checkWin = () => {
		var podane = ''
		for (var i = 0; i < hasla.length; i++) {
			podane += tablicaLiterek[najwShift][i]
		}
		if (slowo.toUpperCase() == podane) {
			isWin = true
			document.querySelector('.odpowiedz').classList.add('poprawna')
			endTime = Math.floor((Date.now() - startTime) / 10) / 100
			lockTiles()
		} else {
			isWin = false
		}
	}
	const letterInput = (e) => {
		const myBox = e.target
		myBox.setSelectionRange(myBox.value.length, myBox.value.length)
		myBox.value = myBox.value[myBox.value.length - 1].toUpperCase()
		checkWin()
		selectNext(e)
	}
	const selectNext = (e) => {
		const box = e.target
		let target = null
		let placeholder = box.nextElementSibling
		if (isWin) return
		try {
			while (placeholder) {
				if (placeholder.classList.contains('dis') == false) {
					target = placeholder
					break
				}

				placeholder = placeholder.nextElementSibling
			}
			target.focus()
		} catch (err) {
			document.querySelector('.box:not(.dis)').focus()
		}
	}
	const manouverArrow = (e) => {
		const box = e.target
		let target = box
		let placeholderN = box.nextElementSibling
		let placeholderP = box.previousElementSibling
		if (e.keyCode == '39') {
			//right
			try {
				while (placeholderN) {
					if (placeholderN.classList.contains('dis') == false) {
						target = placeholderN
						break
					}

					placeholderN = placeholderN.nextElementSibling
				}
				target.focus()
			} catch (err) {
				document.querySelector('.box:not(.dis)').focus()
			}
		} else if (e.keyCode == '40') {
			//down
			try {
				for (var i = 1; i < gridWidth; i++) {
					placeholderN = placeholderN.nextElementSibling
				}
				while (placeholderN) {
					if (placeholderN.classList.contains('dis') == false) {
						target = placeholderN
						break
					}

					placeholderN = placeholderN.nextElementSibling
				}
				target.focus()
			} catch (err) {
				document.querySelector('.box:not(.dis)').focus()
			}
		} else if (e.keyCode == '37') {
			//left
			try {
				while (placeholderP) {
					if (placeholderP.classList.contains('dis') == false) {
						target = placeholderP
						break
					}

					placeholderP = placeholderP.previousElementSibling
				}
				target.focus()
			} catch (err) {
				var tmp = document.querySelectorAll('.box:not(.dis)')
				tmp[tmp.length - 1].focus()
			}
		} else if (e.keyCode == '38') {
			//up
			try {
				for (var i = 1; i < gridWidth; i++) {
					placeholderP = placeholderP.previousElementSibling
				}
				while (placeholderP) {
					if (placeholderP.classList.contains('dis') == false) {
						target = placeholderP
						break
					}

					placeholderP = placeholderP.previousElementSibling
				}
				target.focus()
			} catch (err) {
				var tmp = document.querySelectorAll('.box:not(.dis)')
				tmp[tmp.length - 1].focus()
			}
		}
	}
</script>

<main>
	<h1>Crossword!</h1>
	<p>Długość krzyżówki: <input type="number" bind:value={dlugosc} /></p>
	<button on:click={newSlowo}>New Crossword</button>
	<h2>Długość: {dlugosc}</h2>
	<div class="crossword" style="--grid-width: {gridWidth}; --grid-height: {gridHeight}">
		{#each Array(gridHeight) as _, row}
			{#each Array(gridWidth) as _, column}
				{#if column >= najwShift - shift[row] && column < najwShift + hasla[row].length - shift[row]}
					{#if column == najwShift}
						<input
							type="text"
							class="box answer"
							on:keydown={manouverArrow}
							on:input={letterInput}
							bind:value={tablicaLiterek[column][row]}
						/>
					{:else}
						<input
							type="text"
							class="box"
							on:keydown={manouverArrow}
							on:input={letterInput}
							bind:value={tablicaLiterek[column][row]}
						/>
					{/if}
				{:else}
					<input type="text" class="box dis" disabled />
				{/if}
			{/each}
		{/each}
	</div>
	{#if slowo != ''}
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
	{/if}

	{#if isWin}
		<p>Czas: {endTime}s</p>
	{/if}

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
		transition: 0.2s border-color, 1s color;
	}
	.box:focus-visible {
		border-color: #ff3e00;
		outline: none;
	}
	.box.dis {
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
</style>
