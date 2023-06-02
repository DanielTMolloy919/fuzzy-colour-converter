<script lang="ts">
	function rgbToHex(red: number, green: number, blue: number) {
		// Ensure the input values are within the valid range of 0-255
		red = Math.max(0, Math.min(255, red));
		green = Math.max(0, Math.min(255, green));
		blue = Math.max(0, Math.min(255, blue));

		// Convert each component to its hexadecimal representation
		let redHex = red.toString(16).padStart(2, '0');
		let greenHex = green.toString(16).padStart(2, '0');
		let blueHex = blue.toString(16).padStart(2, '0');

		// Concatenate the components and return the hex color string
		return `#${redHex}${greenHex}${blueHex}`;
	}

	function parseInput(input: string) {
		const whitespace_removed = input.replace(/\s/g, '');

		// Testing for RGB
		let match = input.match(/(\d+)\D+(\d+)\D+(\d+)/);
		if (match) {
			console.log(match[1], match[2], match[3]);
			return rgbToHex(parseInt(match[1]), parseInt(match[2]), parseInt(match[3]));
		}

		return '';
	}

	let input = '';

	$: parsed_hex = parseInput(input);
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
		{parsed_hex}
	</p>
</main>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
