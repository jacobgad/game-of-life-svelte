<script lang="ts">
	import { generateGrid, generateRandomGrid, getNextGen, GridSize } from './utils';

	let gridSize: GridSize = { cols: 40, rows: 20 };
	let grid: number[][] = generateGrid(gridSize);
	let running = false;
	let showGridLines = true;
	let timer: ReturnType<typeof setInterval>;
	let gen = 0;
	let speed = 50;
	let dencity = 50;

	$: grid = generateGrid(gridSize);
	$: running ? runSimulation(speed) : clearInterval(timer);

	function runSimulation(speed: number) {
		clearInterval(timer);
		timer = setInterval(() => {
			const nextGen = getNextGen(grid);
			if (JSON.stringify(grid) === JSON.stringify(nextGen)) running = false;
			grid = nextGen;
			gen++;
		}, 1000 - speed * 10);
	}

	function handleClear() {
		grid = generateGrid(gridSize);
		gen = 0;
	}

	function handleRandom() {
		grid = generateRandomGrid(gridSize, dencity);
		gen = 0;
	}
</script>

<main class="grid justify-center mx-[2.5vw] mt-4 mb-8 gap-4">
	<h1 class="text-center text-3xl font-semibold">Game of Life</h1>

	<div class="flex justify-center gap-4">
		<button
			on:click={() => (running = !running)}
			class="{running ? 'bg-yellow-600 hover:bg-yellow-700' : 'bg-green-600 hover:bg-green-700'} 
        text-white font-bold py-2 px-4 rounded w-24">{running ? 'Stop' : 'Start'}</button
		>
		<button
			on:click={handleClear}
			class="bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded w-24 disabled:opacity-50 disabled:cursor-not-allowed"
			disabled={running}>Clear</button
		>
		<button
			on:click={handleRandom}
			class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-24 disabled:opacity-50 disabled:cursor-not-allowed"
			disabled={running}>Random</button
		>
	</div>

	<div class="grid xl:flex xl:justify-evenly gap-4 xl:gap-8 xl:w-[90vw]">
		<div class="flex justify-between items-center gap-4 w-full">
			<p class="font-semibold text-lg">Generation: {gen}</p>
			<div class="flex items-center gap-2">
				<input bind:checked={showGridLines} type="checkbox" name="gridLines" id="gridLines" />
				<label for="gridLines">Grid Lines</label>
			</div>
		</div>

		<div class="flex justify-between items-center gap-4 w-full">
			<div class="flex border-2 rounded-md overflow-hidden">
				<label class="bg-gray-100 border-r-2 p-2" for="width">Width</label>
				<input
					bind:value={gridSize.cols}
					class="px-3 w-full outline-none"
					type="number"
					name="width"
					id="width"
					placeholder="Width"
				/>
			</div>
			<div class=" flex border-2 rounded-md overflow-hidden">
				<label class="bg-gray-100 border-r-2 p-2" for="height">Height</label>
				<input
					bind:value={gridSize.rows}
					class="px-3 w-full outline-none"
					type="number"
					name="height"
					id="height"
					placeholder="Height"
				/>
			</div>
		</div>

		<div class="flex justify-between items-center gap-8 w-full">
			<div class="w-full grid gap-2">
				<label for="speed">Speed: {speed}</label>
				<input bind:value={speed} type="range" name="speed" id="speed" />
			</div>
			<div class="w-full grid gap-2">
				<label for="dencity">Dencity: {dencity}</label>
				<input
					bind:value={dencity}
					type="range"
					name="dencity"
					id="dencity"
					class="disabled:opacity-50 disabled:cursor-not-allowed"
					disabled={running}
				/>
			</div>
		</div>
	</div>
</main>

<div
	class="mx-[2.5vw] mb-6 border-t-[1px] border-l-[1px] 
	{!showGridLines && 'border-r-[1px] border-b-[1px]'}"
>
	{#each grid as row}
		<div class="flex">
			{#each row as cell}
				<div
					on:click={() => (cell ? (cell = 0) : (cell = 1))}
					class="w-full aspect-square  
          {cell && 'bg-blue-300'} 
          {showGridLines && 'border-b-[1px] border-r-[1px]'}"
				/>
			{/each}
		</div>
	{/each}
</div>
