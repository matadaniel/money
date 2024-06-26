<script lang="ts">
	import AreaChart from '$lib/AreaChart.svelte';
	import NumInput from '$lib/NumInput.svelte';
	import { currency } from '$lib/stores';
	import { reporter } from '@felte/reporter-svelte';
	import { validator } from '@felte/validator-zod';
	import { createForm } from 'felte';
	import { z } from 'zod';

	const schema = z.object({
		expenses: z.number().nonnegative(),
		savings: z.number().nonnegative()
	});

	let deposit = 100000;
	let data: { id: string; data: { month: Date; total: number }[] }[];

	const { form } = createForm<z.infer<typeof schema>>({
		onSubmit: ({ expenses, savings }) => {
			expenses /= 12;

			const stockReturns = 0.0075;
			const bondReturns = 0.0048;
			const stockWeight = 0.6;
			const bondWeight = 0.4;
			const returns = stockReturns * stockWeight + bondReturns * bondWeight;

			const inflation = 0.0198 / 12;

			let balance = deposit + savings;

			const date = new Date();
			const currentMonth = date.getMonth();

			let months = 0;
			const maxMonths = 12 * 81;
			const expensesData: (typeof data)[number] = { id: 'expenses', data: [] };
			const balanceData: (typeof data)[number] = { id: 'balance', data: [] };
			const month = new Date(date);
			expensesData.data.push({ month, total: expenses });
			balanceData.data.push({ month, total: balance });
			while (balance - expenses >= 0 && months < maxMonths) {
				balance -= expenses;
				expenses *= 1 + inflation;
				balance *= 1 + returns;
				months++;

				const month = new Date(date);
				month.setMonth(currentMonth + months);
				expensesData.data.push({ month, total: expenses });
				balanceData.data.push({ month, total: balance });
			}

			data = [expensesData, balanceData];
			console.log(
				Math.trunc(months / 12),
				' years',
				months % 12 > 0 ? ` and ${months % 12} months` : ''
			);
		},
		extend: [validator({ schema }), reporter]
	});
</script>

<h1 class="mb-8">Congratulations!!</h1>
<h2 class="mb-8">An oil prince has just deposited $100,000 into your bank account!</h2>
<p class="mb-8">
	Are you able to sleep in tomorrow? And the day after that, and the day after that, and the day
	after that...
</p>

<br />

<form
	use:form
	class="mt-8 grid w-96 grid-cols-1 gap-14 border-primary-500 p-5 border-token rounded-container-token"
>
	<NumInput
		inputAttributes={{ id: 'expenses', required: true }}
		labelText="Yearly Expenses:"
		let:value
	>
		{#if typeof value === 'number'}<span
				>{value.toLocaleString(undefined, { style: 'currency', currency: $currency })}</span
			>{:else}<div class="placeholder mb-2 w-24 animate-pulse" />{/if}
	</NumInput>

	<NumInput
		inputAttributes={{ id: 'savings', required: true }}
		labelText="Current Savings:"
		let:value
	>
		{#if typeof value === 'number'}<span
				>{value.toLocaleString(undefined, { style: 'currency', currency: $currency })}</span
			>{:else}<div class="placeholder mb-2 w-24 animate-pulse" />{/if}
	</NumInput>

	<button class="btn-filled-primary btn">Submit</button>
</form>

<div class="card my-16 flex flex-col items-center p-4">
	{#key data}
		{#if data}
			{@const months = data[0].data.length - 1}

			<header class="card-header">
				<p class="unstyled text-center text-3xl font-medium">
					Your money should <a href="https://ssrn.com/abstract=3964908"><i>theoretically</i></a><br
					/>last you {#if months > 11}{`${Math.trunc(months / 12)} year${
							months > 23 ? 's' : ''
						} and `}{/if}{`${months % 12} ${
						months % 12 === 1 ? 'month' : 'months'
					}`}{#if months === 81 * 12}{' or more'}{/if}.
				</p>
			</header>

			<AreaChart {data} />

			<footer class="card-footer">
				<small>Extrapolated from historical data.</small>
			</footer>
		{:else}
			<div class="placeholder h-[503px] w-[600px] animate-pulse" />
		{/if}
	{/key}
</div>
