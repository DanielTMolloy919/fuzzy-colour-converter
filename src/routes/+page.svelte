<script lang="ts">
	import { Clipboard } from 'lucide-svelte';
	import { copy } from 'svelte-copy';

	function rgbToHex(red: number, green: number, blue: number) {
		if (red < 0 || red > 255 || green < 0 || green > 255 || blue < 0 || blue > 255) {
			return '';
		}

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

	function hexToRgb(hex: string) {
		// Remove the # symbol if present
		hex = hex.replace('#', '');

		// Check if the hex value is valid
		if (!/^[0-9A-Fa-f]{6}$/.test(hex)) {
			return '';
		}

		// Extract the RGB components
		const r = parseInt(hex.slice(0, 2), 16);
		const g = parseInt(hex.slice(2, 4), 16);
		const b = parseInt(hex.slice(4, 6), 16);

		// Return the RGB values as an object
		return `${r}, ${g}, ${b}`;
	}

	function hexToHsl(hex: string) {
		// Convert hex to RGB first
		let r = 0,
			g = 0,
			b = 0;
		if (hex.length == 4) {
			r = parseInt('0x' + hex[1] + hex[1]);
			g = parseInt('0x' + hex[2] + hex[2]);
			b = parseInt('0x' + hex[3] + hex[3]);
		} else if (hex.length == 7) {
			r = parseInt('0x' + hex[1] + hex[2]);
			g = parseInt('0x' + hex[3] + hex[4]);
			b = parseInt('0x' + hex[5] + hex[6]);
		}
		// Then to HSL
		r /= 255;
		g /= 255;
		b /= 255;
		let cmin = Math.min(r, g, b),
			cmax = Math.max(r, g, b),
			delta = cmax - cmin,
			h = 0,
			s = 0,
			l = 0;

		if (delta == 0) h = 0;
		else if (cmax == r) h = ((g - b) / delta) % 6;
		else if (cmax == g) h = (b - r) / delta + 2;
		else h = (r - g) / delta + 4;

		h = Math.round(h * 60);

		if (h < 0) h += 360;

		l = (cmax + cmin) / 2;
		s = delta == 0 ? 0 : delta / (1 - Math.abs(2 * l - 1));
		s = +(s * 100).toFixed(1);
		l = +(l * 100).toFixed(1);

		return `${h}, ${s}%, ${l}%`;
	}

	function parseInput(input: string) {
		const whitespace_removed = input.replace(/\s/g, '');

		// Testing for Hex
		let match = input.match(/#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})\b/);
		if (match) {
			return match[0];
		}

		// Testing for RGB
		match = input.match(/(\d+)\D+(\d+)\D+(\d+)/);
		if (match) {
			return rgbToHex(parseInt(match[1]), parseInt(match[2]), parseInt(match[3]));
		}

		return '';
	}

	let input = '';

	$: parsed_hex = parseInput(input);
	$: parsed_rgb = hexToRgb(parsed_hex);
	$: parsed_hsl = parsed_hex ? hexToHsl(parsed_hex) : '';

	$: preview_background = parsed_hex ? `${parsed_hex}` : 'white';
</script>

<main class="flex min-h-screen flex-col items-center p-12 max-w-2xl m-auto">
	<h1 class="text-4xl">Fuzzy Colour Converter</h1>
	<input
		class="mt-6 text-md block px-3 py-2 rounded-lg w-full
	bg-white border-2 border-gray-300 placeholder-gray-600 shadow-md focus:placeholder-gray-500 focus:bg-white focus:border-gray-600 focus:outline-none"
		placeholder="Paste whatever..."
		bind:value={input}
	/>
	<div class="grid grid-cols-2 gap-4 w-full">
		<div use:copy={parsed_hex}>
			<h1 class="mt-3 text-xl">Hexadecimal</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 transition-colors cursor-pointer flex items-center justify-between"
			>
				{parsed_hex}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
		<div use:copy={parsed_rgb} class="relative">
			<!-- <div
				class="absolute top-2 right-24 bg-gray-800 text-gray-200 px-2 py-1 rounded"
				style="z-index: 999"
			>
				Copied!
			</div> -->
			<h1 class="mt-3 text-xl">RGB</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 focus:bg-pink-400 transition-colors cursor-pointer flex items-center justify-between"
			>
				{parsed_rgb}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
		<div use:copy={parsed_hsl} class="relative">
			<!-- <div
				class="absolute top-2 right-24 bg-gray-800 text-gray-200 px-2 py-1 rounded"
				style="z-index: 999"
			>
				Copied!
			</div> -->
			<h1 class="mt text-xl">HSL</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 focus:bg-pink-400 transition-colors cursor-pointer flex items-center justify-between"
			>
				{parsed_hsl}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
	</div>
	<!-- border-2 border-gray-400  -->
	<div
		class="rounded-2xl h-40 w-40 rounded-box bg-white mt-10 color-preview shadow-lg"
		style="background-color: {preview_background}"
	/>
</main>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
