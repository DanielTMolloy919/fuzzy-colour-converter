<script lang="ts">
	function parseInput(input: string) {
		return input;
	}

	let input = '';

	$: parsed_input = parseInput(input);
</script>

<main class="flex min-h-screen flex-col items-center p-12 max-w-2xl m-auto">
	<h1 class="text-4xl">Fuzzy Colour Converter</h1>
	<input
		class="mt-6 text-md block px-3 py-2 rounded-lg w-full
	bg-white border-2 border-gray-300 placeholder-gray-600 shadow-md focus:placeholder-gray-500 focus:bg-white focus:border-gray-600 focus:outline-none"
		bind:value={input}
	/>
	<p
		class="mt-6 text-md block px-3 py-2 rounded-lg w-full h-11
	bg-white border-2 border-gray-300 placeholder-gray-600 shadow-md focus:placeholder-gray-500 focus:bg-white focus:border-gray-600 focus:outline-none"
	>
		{parsed_input}
	</p>
</main>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
