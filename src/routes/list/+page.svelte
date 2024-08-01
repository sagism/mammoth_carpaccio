<script>
	import { writable, get } from 'svelte/store';

	import { onMount } from 'svelte';

	// Initialize prices and quantities as writable stores
	const prices = writable([null]);
	const quantities = writable([null]);
	let log = [];

	let discounts_table = {
		1000: 10,
		2000: 15,
		3000: 20
	};

	let state = 'CA'; // default
	let states = {
		CA: 8.15,
		TX: 6.25
	};
	$: tax_rate = () => {
		return states[state];
	};

	$: discount_rate = () => {
		let discount_rate = 0;
		for (let limit in discounts_table) {
			if (preTaxTotal >= limit) {
				discount_rate = discounts_table[limit];
			} else {
				break;
			}
		}

		return discount_rate;
	};

	// Calculate totals
	let totals = [];

	$: preTaxTotal = totals.reduce((sum, total) => sum + total, 0);
	// Calculate grand total
	$: grandTotal =
		(1 - discount_rate() / 100) *
		totals.reduce((sum, total) => sum + total, 0) *
		(1 + tax_rate() / 100);

	// Update totals whenever prices or quantities change
	$: {
		totals = $prices.map((price, index) => price * $quantities[index]);
	}

	// Add a new row
	async function addRow() {
		prices.update((n) => [...n, null]);
		quantities.update((n) => [...n, null]);
		await new Promise((resolve) => requestAnimationFrame(resolve));

		// Focus the first input in the newly added row
		const newInputs = document.querySelectorAll('input');
		newInputs[newInputs.length - 2].focus();
	}

	// Remove a row
	function removeRow(index) {
		prices.update((n) => n.filter((_, i) => i !== index));
		quantities.update((n) => n.filter((_, i) => i !== index));
	}

	function formatPrice(price) {
		return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(price);
	}

	async function handleKeydown(event) {
		if (event.key === 'Tab' && event.shiftKey === false) {
			const inputs = document.querySelectorAll('input');
			const lastInput = inputs[inputs.length - 1];
			if (lastInput === document.activeElement) {
				addRow();
			}
		}
		if (event.key === 'Escape') {
			clear();
			await new Promise((resolve) => requestAnimationFrame(resolve));
			document.querySelector('input[name="price"]').focus();
		}
	}

	onMount(() => {
		const input = document.querySelector('input');
		if (input) {
			input.focus();
		}

		document.addEventListener('keydown', handleKeydown);

		const storedLog = localStorage.getItem('log');
		if (storedLog) {
			log = JSON.parse(storedLog);
		}

		// Cleanup the event listener when the component is destroyed
		return () => {
			document.removeEventListener('keydown', handleKeydown);
		};
	});

	function populateFromLog(index) {
		const item = log[index];
		console.table(item);
		price = item.price;
		quantity = item.quantity;
	}

	function items() {
		const currentPrices = get(prices);
		const currentQuantities = get(quantities);

		return currentPrices
			.map((price, index) => ({
				price: price,
				quantity: currentQuantities[index]
			}))
			.filter(
				(item) =>
					item.price !== null && item.price !== 0 && item.quantity !== null && item.quantity !== 0
			);
	}

	// Save the item to local storage
	const saveQuote = () => {
		if (!grandTotal) {
			return;
		}

		const newItem = {
			lines: items(),
			total: grandTotal,
			preTaxTotal: preTaxTotal,
			timestamp: new Date(),
			state: state,
			discount_rate: discount_rate(),
			tax_rate: tax_rate()
		};
		log = [newItem, ...log];
		localStorage.setItem('log', JSON.stringify(log));
	};

	function clear() {
		prices.set([null]);
		quantities.set([null]);
		document.querySelector('input[name="price"]').focus();
	}

	const clearLog = () => {
		if (confirm('Are you sure you want to clear the log?')) {
			log = [];
			localStorage.removeItem('log');
		}
	};
</script>

<div class=" flex flex-col gap-4 justify-center m-4">
	<h1 class="w-full text-center text-3xl font-bold">Quotes helper</h1>
	<div class="text-center text-md">
		<div class="text-xl">
			Grand Total: <span class="font-bold text-sky-700">{formatPrice(grandTotal)}</span>
		</div>
		({formatPrice(preTaxTotal)} - {formatPrice((discount_rate() / 100) * preTaxTotal)}
		discount +
		{formatPrice((tax_rate() / 100) * preTaxTotal)} tax )
	</div>

	<div class="flex flex-row justify-center items-baseline gap-4">
		<div class="align-baseline">State</div>
		<select name="state" class="select select-bordered bg-white" bind:value={state}>
			{#each Object.entries(states) as [key, value] (key)}
				<option value={key}>
					{key}
				</option>
			{/each}
		</select>
		<div class="w-24">{tax_rate()}%</div>
		<div class="w-48">discount rate: {discount_rate()}%</div>
	</div>
	<div class="container mx-auto">
		<table class="table">
			<thead>
				<tr>
					<th>Price</th>
					<th>Quantity</th>
					<th>Total</th>
					<th>Action</th>
				</tr>
			</thead>
			<tbody>
				{#each $prices as price, index}
					<tr>
						<td class="text-right p-0"
							><input
								class="text-right py-1"
								name="price"
								type="number"
								bind:value={$prices[index]}
								min="0"
							/></td
						>
						<td class="text-right p-0"
							><input
								class="text-right py-1"
								type="number"
								bind:value={$quantities[index]}
								min="0"
							/></td
						>
						<td class="text-right p-0">{formatPrice(totals[index])}</td>
						<td class="text-center"
							><button class="btn btn-secondary btn-xs" on:click={() => removeRow(index)}
								>Remove</button
							></td
						>
					</tr>
				{/each}
			</tbody>
		</table>
		<button class="btn btn-primary m-4" on:click={addRow}>Add Row</button>
		<button class="btn btn-primary m-4" on:click={saveQuote}>Save quote</button>

		<!-- <table class="log-list border-2 w-full mx-2" id="log-entries">
			<thead>
				<tr class="log-item border-b-2">
					<th class="px-4">Date</th>
					<th class="px-4">Amount</th>
				</tr>
			</thead>
			<tbody>
				{#each log as item, index (item.timestamp)}
					<tr
						class="log-item even:bg-stone-100 border-b-2 cursor-pointer log-entry"
						item-index={index}
						on:click={() => console.log(index)}
					>
						<td class="px-4 font-mono text-stone-600"
							>{new Date(item.timestamp).toLocaleString()}</td
						>
						<td class="amount total">{formatPrice(item.value)}</td>
						<td class="italic px-4 text-sky-800">{formatPrice(item.total)}</td>
					</tr>
				{/each}
			</tbody>
		</table> -->

		<table class="table border-gray-300 border-2 table-zebra">
			<thead>
				<tr>
					<th>Timestamp</th>
					<th>Total</th>
				</tr>
			</thead>
			<tbody>
				{#each log as item}
					<tr on:click={() => console.log(item)}>
						<td class="border-1 border-black">{new Date(item.timestamp).toLocaleString()}</td>
						<td>{formatPrice(item.total)}</td>
					</tr>
				{/each}
			</tbody>
		</table>

		<button class="btn btn-primary m-4" on:click={clearLog}>Clear log</button>
	</div>
</div>
