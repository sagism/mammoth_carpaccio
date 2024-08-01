<script>
	import { onMount } from 'svelte';

	let price;
	let quantity;
	let state = 'CA'; // default
	let states = {
		CA: 8.15,
		TX: 6.25
	};
	$: tax_rate = () => {
		return states[state];
	};
	$: total = price * quantity * (1 + tax_rate() / 100);

	function formatPrice(price) {
		return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(price);
	}

	function clear() {
		price = 0.0;
		quantity = 1;
		document.querySelector('input[name="price"]').focus();
	}

	onMount(() => {
		document.querySelector('input[name="price"]').focus();
		document.addEventListener('keydown', (event) => {
			if (event.key === 'Escape') {
				clear();
			}
		});
	});
</script>

<div class="container mx-auto content-center p-4">
	<h1 class="p-4 text-center">Welcome to The Quotes Helper</h1>
	<div class="flex justify-center items-baseline gap-4">
		<select name="state" class="select select-bordered" bind:value={state}>
			{#each Object.entries(states) as [key, value] (key)}
				<option value={key}>
					{key}
				</option>
			{/each}
		</select>

		<div class="text-right align-baseline text-wrap-none w-18" id="total">
			tax: {tax_rate()}%
		</div>
		<div class="text-right align-baseline w-24" id="total">{formatPrice(total)}</div>

		<button class="btn btn-primary" on:click={clear()}>Clear</button>
	</div>
	<div class="flex gap-4 justify-center items-baseline">
		<input
			type="number"
			min="0"
			step="0.05"
			class="input input-bordered text-right align-baseline w-24"
			name="price"
			bind:value={price}
			placeholder="price"
		/>
		<input
			class="input input-bordered text-right align-baseline w-24"
			name="quantity"
			bind:value={quantity}
			placeholder="quantity"
		/>
	</div>
</div>
