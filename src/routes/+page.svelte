<script>
	let price;
	let quantity;
	$: total = price * quantity;

	function formatPrice(price) {
		return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(price);
	}
</script>

<h1>Welcome to The Quotes Helper</h1>
<input name="price" bind:value={price} placeholder="price" />
<input name="quantity" bind:value={quantity} placeholder="quantity" />
<div id="total">{formatPrice(total)}</div>
