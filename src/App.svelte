<script lang="ts">
	interface GridSize {
		cols: number;
		rows: number;
	}

	const neighbours = [
		[-1, -1],
		[-1, 0],
		[-1, 1],
		[0, -1],
		[0, 1],
		[1, -1],
		[1, 0],
		[1, 1],
	];

	function generateGrid(gridSize: GridSize) {
		let grid: number[][] = [];
		for (let i = 0; i < gridSize.rows; i++) {
			grid.push(Array.from(Array(gridSize.cols), () => 0));
		}
		return grid;
	}

	function generateRandomGrid(gridSize: GridSize, randPopDenc: number) {
		let grid: number[][] = [];
		for (let i = 0; i < gridSize.rows; i++) {
			grid.push(
				Array.from(Array(gridSize.cols), () => (Math.random() < randPopDenc / 100 ? 1 : 0))
			);
		}
		return grid;
	}

	function getNeighbours(grid: number[][], i: number, j: number) {
		let numNeighbours = 0;
		neighbours.map((n) => {
			const realI = i + n[1];
			const realJ = j + n[0];
			if (realI >= 0 && realI < grid.length && realJ >= 0 && realJ < grid[0].length)
				numNeighbours += grid[realI][realJ];
		});
		return numNeighbours;
	}

	function getNextGen(grid: number[][]) {
		let newGrid = grid.map((arr) => arr.slice());
		for (let i = 0; i < grid.length; i++) {
			for (let j = 0; j < grid[0].length; j++) {
				const neighbours = getNeighbours(grid, i, j);
				if (neighbours < 2 || neighbours > 3) newGrid[i][j] = 0;
				if (neighbours === 3) newGrid[i][j] = 1;
			}
		}
		return newGrid;
	}

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
</script>

<main class="grid justify-center mx-[2.5vw] my-6 gap-4">
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
			on:click={() => (grid = generateRandomGrid(gridSize, dencity))}
			class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-24 disabled:opacity-50 disabled:cursor-not-allowed"
			disabled={running}>Random</button
		>
	</div>

	<div class="flex justify-between items-center gap-4">
		<div class="flex border-2 rounded-md overflow-hidden">
			<label class="bg-gray-100 border-r-2 p-2" for="width">Width</label>
			<input
				bind:value={gridSize.cols}
				class="p-1 w-full"
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
				class="p-1 w-full"
				type="number"
				name="height"
				id="height"
				placeholder="Height"
			/>
		</div>
	</div>

	<div class="flex justify-between items-center gap-4">
		<p class="font-semibold text-lg">Generation: {gen}</p>
		<div>
			<input bind:checked={showGridLines} type="checkbox" name="gridLines" id="gridLines" />
			<label for="gridLines">Grid Lines</label>
		</div>
	</div>

	<div class="flex justify-between items-center gap-8">
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
</main>

<div class="mx-[2.5vw] mb-6 border-t-2 border-l-2 {!showGridLines && 'border-r-2 border-b-2'}">
	{#each grid as row}
		<div class="flex">
			{#each row as cell}
				<div
					on:click={() => (cell ? (cell = 0) : (cell = 1))}
					class="w-full aspect-square  
          {cell && 'bg-blue-300'} 
          {showGridLines && 'border-b-2 border-r-2'}"
				/>
			{/each}
		</div>
	{/each}
</div>
