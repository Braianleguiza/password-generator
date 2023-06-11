<script lang="ts">
	import Copy from 'carbon-icons-svelte/lib/Copy.svelte';
	import { onMount } from 'svelte';
	import { RangeSlider } from '@skeletonlabs/skeleton';
	import { AppBar, LightSwitch, clipboard } from '@skeletonlabs/skeleton';
	import { Renew, ThumbsUpFilled } from 'carbon-icons-svelte';
	import Logos from '../components/Logos.svelte';

	// Password information
	let value: number = 10;
	let max: number = 16;
	let generatedPassword: string = 'Click in the options below';

	// Checkbox options
	let includeUppercase = true;
	let includeLowercase = true;
	let includeNumbers = false;
	let includeSymbols = false;

	// First advice
	let passwordStrength: string = '';
	let strengthLevel: number = 0;

	function generatePassword(length: number): string {
		const lowercase = 'abcdefghijklmnopqrstuvwxyz';
		const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
		const numbers = '0123456789';
		const symbols = '!@#$%&@&++';

		let characters = '';

		if (includeLowercase) {
			characters += lowercase;
		}
		if (includeUppercase) {
			characters += uppercase;
		}
		if (includeNumbers) {
			characters += numbers;
		}
		if (includeSymbols) {
			characters += symbols;
		}

		let password = '';

		for (let i = 0; i < length; i++) {
			const randomIndex = Math.floor(Math.random() * characters.length);
			password += characters.charAt(randomIndex);
		}

		return password;
	}

	function updateGeneratedPassword() {
		generatedPassword = generatePassword(value);
		calculatePasswordStrength();
	}

	function handleCheckboxChange() {
		updateGeneratedPassword();
	}

	let strengthStat = '';
	function calculatePasswordStrength() {
		strengthLevel = generatedPassword.length;
		passwordStrength = `${strengthLevel}`;

		let passNumber = Number(passwordStrength);

		if (passNumber < 4) {
			passwordStrength = 'WEAK';
			strengthStat = 'error-500';
			strengthLevel = strengthLevel / 4;
		} else if (passNumber < 8) {
			passwordStrength = 'NORMAL';
			strengthStat = 'warning-500';
			strengthLevel = strengthLevel / 4;
		} else if (passNumber < 12) {
			passwordStrength = 'GOOD';
			strengthStat = 'success-500';
			strengthLevel = strengthLevel / 4;
		} else {
			passwordStrength = 'STRONG';
			strengthStat = 'green-500';
			strengthLevel = strengthLevel / 4;
		}
	}

	function handleClick() {
		updateGeneratedPassword();
	}

	onMount(() => {
		updateGeneratedPassword();
	});

	let isCopied = false;
	let isButtonDisabled = false;
	let buttonCopyColor = '';

	async function handleCopy() {
		if (!isButtonDisabled) {
			isButtonDisabled = true;
			await navigator.clipboard.writeText(generatedPassword);
			isCopied = true;

			setTimeout(() => {
				isCopied = false;
				isButtonDisabled = false;
				buttonCopyColor = '';
			}, 1000);

			buttonCopyColor = '-primary';
		}
	}
</script>

<AppBar class="absolute w-full shadow">
	<svelte:fragment slot="lead"><img src="passLogoIcon.png" alt="Logo generator" /></svelte:fragment
	><b class="font-mono text-lg">MySecurePassword</b>
	<div class="absolute right-4 top-5"><LightSwitch /></div>
</AppBar>
<div class="container h-full mx-auto flex flex-col justify-center items-center">
	<h3 class="h4 mb-2 flex items-center">Password Generator</h3>
	<div class="card shadow p-5 mb-3 md:w-[550px] w-72">
		<p class="flex justify-between items-center">
			<span id="password" class="font-mono">{generatedPassword}</span>
			<button
				class="btn variant-filled{buttonCopyColor} rounded"
				use:clipboard={generatedPassword}
				disabled={isButtonDisabled}
				on:click={handleCopy}
			>
				{#if isCopied}
					<ThumbsUpFilled class="mr-1" /> Copied
				{:else}
					<Copy class="mr-1" /> Copy
				{/if}
			</button>
		</p>
	</div>
	<div class="card shadow p-5 md:w-[550px] w-72">
		<RangeSlider name="range-slider" bind:value {max} step={1} ticked>
			<div class="flex justify-between items-center py-4">
				<div class="font-semibold">Character Length</div>
				<div class="text font-semibold text-success-500">{value} / {max}</div>
			</div>
		</RangeSlider>
		<div class="space-y-2 my-8">
			<label class="flex items-center space-x-2">
				<input
					class="checkbox"
					type="checkbox"
					bind:checked={includeUppercase}
					on:change={handleCheckboxChange}
				/>
				<p>Include Uppercase Letters</p>
			</label>
			<label class="flex items-center space-x-2">
				<input
					class="checkbox"
					type="checkbox"
					bind:checked={includeLowercase}
					on:change={handleCheckboxChange}
				/>
				<p>Include Lowercase Letters</p>
			</label>
			<label class="flex items-center space-x-2">
				<input
					class="checkbox"
					type="checkbox"
					bind:checked={includeNumbers}
					on:change={handleCheckboxChange}
				/>
				<p>Include Numbers</p>
			</label>
			<label class="flex items-center space-x-2">
				<input
					class="checkbox"
					type="checkbox"
					bind:checked={includeSymbols}
					on:change={handleCheckboxChange}
				/>
				<p>Include Symbols</p>
			</label>
		</div>
		<div class="card shadow-sm p-4 flex justify-between mt-5">
			<span>STRENGTH</span>
			<p class="text-{strengthStat} font-semibold">{passwordStrength}</p>
			<div class="card flex">
				{#each Array(4) as _, index}
					<div
						class="rounded-sm border border-surface-400 border-opacity-50 p-1 mr-0.5 {index <
						strengthLevel
							? 'bg-' + strengthStat
							: ''}"
					/>
				{/each}
			</div>
		</div>

		<button
			on:click={handleClick}
			class="btn variant-filled-success w-full mt-4"
			data-sveltekit-preload-data="hover"
		>
			<span>Generate</span>
			<Renew />
		</button>
	</div>
	<Logos />
</div>

<style>
	img {
		height: 32px;
	}
</style>
