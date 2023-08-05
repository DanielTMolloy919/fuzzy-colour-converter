<script lang="ts">
	import { Clipboard } from 'lucide-svelte';
	import { copy } from 'svelte-copy';
	var namer = require('color-namer');

	function rgbToHex(red: number, green: number, blue: number, alpha = 1) {
		// Convert each component to its hexadecimal representation
		let redHex = red.toString(16).padStart(2, '0');
		let greenHex = green.toString(16).padStart(2, '0');
		let blueHex = blue.toString(16).padStart(2, '0');

		// Concatenate the components and return the hex color string
		let hex_colour = `#${redHex}${greenHex}${blueHex}`;

		if (alpha !== 1) {
			let alphaHex = Math.round(alpha * 255)
				.toString(16)
				.padStart(2, '0');
			hex_colour += alphaHex;
		}

		return hex_colour;
	}

	function hexToRgb(hex: string) {
		// Remove the # symbol if present
		hex = hex.replace('#', '');

		// Extract the RGB components
		const r = parseInt(hex.slice(0, 2), 16);
		const g = parseInt(hex.slice(2, 4), 16);
		const b = parseInt(hex.slice(4, 6), 16);
		const a = hex.length === 8 ? parseInt(hex.slice(6, 8), 16) / 255 : null;

		// Return the RGB values as an object
		return a ? `${r}, ${g}, ${b}, ${a.toFixed(2)}` : `${r}, ${g}, ${b}`;
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
		} else {
			r = parseInt('0x' + hex[1] + hex[2]);
			g = parseInt('0x' + hex[3] + hex[4]);
			b = parseInt('0x' + hex[5] + hex[6]);
		}

		const a = hex.length === 9 ? parseInt(hex.slice(7, 9), 16) / 255 : null;

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

		return a ? `${h}, ${s}%, ${l}%, ${a.toFixed(2)}` : `${h}, ${s}%, ${l}%`;
	}

	function parseInput(input: string) {
		// Testing for Hex 8
		let match = input.match(/\b([A-Fa-f0-9]{8})\b/);
		if (match) {
			let hex = match[0];
			if (hex.slice(6, 8) == 'ff') {
				hex = hex.slice(0, 6);
			}
			return `#${hex}`;
		}

		// Testing for RGBA
		match = input.match(/(\d+)\D+(\d+)\D+(\d+)\D+(\d+\.\d+)/);
		if (match) {
			const red = parseInt(match[1]);
			const green = parseInt(match[2]);
			const blue = parseInt(match[3]);
			const alpha = parseFloat(match[4]);

			if (
				red >= 0 &&
				red <= 255 &&
				green >= 0 &&
				green <= 255 &&
				blue >= 0 &&
				blue <= 255 &&
				alpha >= 0 &&
				alpha <= 1
			) {
				return rgbToHex(red, green, blue, alpha);
			}
		}

		// Testing for Hex 6
		match = input.match(/\b([A-Fa-f0-9]{6})\b/);
		if (match) {
			return `#${match[0]}`;
		}

		// Testing for RGB
		match = input.match(/(\d+)\D+(\d+)\D+(\d+)/);
		if (match) {
			const red = parseInt(match[1]);
			const green = parseInt(match[2]);
			const blue = parseInt(match[3]);

			if (red >= 0 && red <= 255 && green >= 0 && green <= 255 && blue >= 0 && blue <= 255) {
				return rgbToHex(red, green, blue);
			}
		}

		// Testing for Hex 3
		match = input.match(/#([A-Fa-f0-9]{3})\b/);
		if (match) {
			let hex = match[0].replace('#', '');
			hex = hex
				.split('')
				.map((char) => char + char)
				.join('');
			return `#${hex}`;
		}

		return '';
	}

	let input = '';

	$: parsed_hex = parseInput(input);
	$: parsed_rgb = parsed_hex ? hexToRgb(parsed_hex) : '';
	$: parsed_hsl = parsed_hex ? hexToHsl(parsed_hex) : '';

	$: javascript_rgb_prefix = parsed_hex.length === 9 ? 'rgba' : 'rgb';
	$: javascript_rgb = parsed_hex
		? `${javascript_rgb_prefix}(${parsed_rgb.replaceAll(' ', '')})`
		: '';

	$: javascript_hsl_prefix = parsed_hex.length === 9 ? 'hsla' : 'hsl';
	$: javascript_hsl = parsed_hex
		? `${javascript_hsl_prefix}(${parsed_hsl.replaceAll(' ', '')})`
		: '';

	$: preview_background = parsed_hex ? `${parsed_hex}` : 'white';

	$: name = parsed_hex ? namer(parsed_hex).ntc[0].name : '';
</script>

<main class="flex min-h-screen flex-col items-center p-12 max-w-4xl m-auto">
	<h1 class="text-4xl">Fuzzy Colour Converter</h1>
	<input
		class="mt-6 text-md block px-3 py-2 rounded-lg w-full
	bg-white border-2 border-gray-300 placeholder-gray-300 shadow-md focus:placeholder-gray-500 focus:bg-white focus:border-gray-600 focus:outline-none"
		placeholder="Paste whatever..."
		bind:value={input}
	/>
	<div class="grid grid-cols-3 gap-4 w-full">
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
			<h1 class="mt-3 text-xl">RGB</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 transition-colors cursor-pointer flex items-center justify-between"
			>
				{parsed_rgb}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
		<div use:copy={parsed_hsl} class="relative">
			<h1 class="mt-3 text-xl">HSL</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 transition-colors cursor-pointer flex items-center justify-between"
			>
				{parsed_hsl}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
		<div />
		<div use:copy={javascript_rgb} class="relative">
			<h1 class="mt-3 text-l text-gray-400">For CSS</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-white hover:bg-gray-200 transition-colors cursor-pointer flex items-center justify-between"
			>
				{javascript_rgb}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
		<div use:copy={javascript_hsl} class="relative">
			<h1 class="mt-3 text-l text-gray-400">For CSS</h1>
			<div
				class="text-md px-3 py-2 rounded-lg w-full h-10 bg-gray-300 hover:bg-gray-200 transition-colors cursor-pointer flex items-center justify-between"
			>
				{javascript_hsl}
				<div class="ml-auto">
					<Clipboard size="20" />
				</div>
			</div>
		</div>
	</div>
	<div
		class="rounded-2xl h-40 w-40 rounded-box bg-white mt-10 color-preview shadow-lg hover:shadow-2xl transition-shadow duration-500"
		style="background-color: {preview_background}"
	/>
	<h2 class="capitalize text-xl pt-2">{name}</h2>

	<hr class="mt-8 mb-4 border-gray-300 w-full" />
	<div class="flex text-sm text-gray-700 text-center justify-between w-full">
		<div class="my-auto">Made by Daniel Molloy</div>
		<a
			class="p-2 hover:bg-gray-700 hover:text-gray-200 transition-colors rounded-lg"
			href="https://github.com/DanielTMolloy919/fuzzy-colour-converter">Find on Github</a
		>
	</div>
</main>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
