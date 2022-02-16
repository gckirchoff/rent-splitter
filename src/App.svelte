<script lang="ts">
	let people: { name: string; salary: number, id: string }[] = [];
	let bills: { name: string, amount: number, payer: string, id: string }[] = [];

	let newName: string = '';
	let newSalary: number;

	let newBillName: string = '';
	let newBillAmount: number;
	let newBillPayer: string;

	const generateId = (name, quantity, index) => {
		return `${name}-${quantity}-${index}`;
	}

	const addPerson = (): void => {
		const personNumber = people.length;
		people = [...people, { id: generateId(newName, newSalary, personNumber),  name: newName, salary: newSalary }]
		newName = '';
		newSalary = undefined;
	}

	const addBill = (): void => {
		const billNumber = bills.length;
		bills = [...bills, { name: newBillName, amount: newBillAmount, payer: newBillPayer, id: generateId(newBillName, newBillAmount, billNumber) }];
		newBillName = '';
		newBillAmount = 0;
		newBillPayer;
	}

	let payments = [];
	$: {
		const totalCost = bills.reduce((acc, { amount }) => acc + amount, 0);
		const totalSalary = people.reduce((acc, { salary }) => acc + salary, 0);

		const amountToBePaidByPerson = people.reduce((acc, { name, salary }) => {
		acc[name] = (salary * totalCost) / totalSalary;
		return acc;
		}, {});

		const amountPaidByPersonSoFar = bills.reduce((acc, { payer, amount }) => {
			if (!(payer in acc)) acc[payer] = 0;
			acc[payer] += amount;
			return acc;
		}, {});

		people.forEach(person => {
			const { name } = person;
			if (!(name in amountPaidByPersonSoFar)) {
				amountPaidByPersonSoFar[name] = 0
			}
		});

		const sortedIncreasingDeltas = people.map(({ name }) => {
			const paid = amountPaidByPersonSoFar[name];
			const shouldBePaid = amountToBePaidByPerson[name];
			return {
				name,
				delta: shouldBePaid - paid,
			}
		}).sort((person1, person2) => person2.delta - person1.delta);

		let leftPointer = 0;
		let rightPointer = sortedIncreasingDeltas.length - 1;

		let richerDeltaTemp = Infinity;
		let poorerDeltaTemp = -Infinity;
		
		const paymentsToBeMade = [];
		while (leftPointer < rightPointer) {
		const { name: richerName, delta: richerDelta } = sortedIncreasingDeltas[leftPointer];
		const { name: poorerName, delta: poorerDelta } = sortedIncreasingDeltas[rightPointer];
		richerDeltaTemp = richerDelta < richerDeltaTemp ? richerDelta : richerDeltaTemp;
		poorerDeltaTemp = poorerDelta > poorerDeltaTemp ? poorerDelta : poorerDeltaTemp;
		if (Math.abs(richerDeltaTemp) === Math.abs(poorerDeltaTemp)) {
			paymentsToBeMade.push([richerName, richerDeltaTemp, poorerName]);
			leftPointer++;
			rightPointer--;
			richerDeltaTemp = Infinity;
			poorerDeltaTemp = -Infinity;
		} else if (Math.abs(richerDeltaTemp) > Math.abs(poorerDeltaTemp)) {
			richerDeltaTemp -= Math.abs(poorerDeltaTemp);
			paymentsToBeMade.push([richerName, Math.abs(poorerDeltaTemp), poorerName]);
			rightPointer--;
			poorerDeltaTemp = -Infinity;
		} else if (Math.abs(richerDeltaTemp) < Math.abs(poorerDeltaTemp)) {
			poorerDeltaTemp += richerDeltaTemp;
			paymentsToBeMade.push([richerName, richerDeltaTemp, poorerName]);
			leftPointer++;
			richerDeltaTemp = Infinity;
			}
		}
		payments = paymentsToBeMade;
	}
</script>

<main>
	<form on:submit|preventDefault={addPerson}>
		<input bind:value={newName} placeholder="Name of Renter" />
		<input type="number" bind:value={newSalary} placeholder="Salary of Renter" />
		<button>Add person</button>
	</form>

	<form on:submit|preventDefault={addBill}>
		<input bind:value={newBillName} placeholder="Name of Bill" />
		<input bind:value={newBillAmount} type="number" placeholder="Bill amount" />
		<select bind:value={newBillPayer} placeholder="Paid by">
			{#each people as { name }}
				<option value={name}>{name}</option>
			{/each}
		</select>
		<button>Add bill</button>
	</form>

	{#if people.length > 0}
		<h4>Renters</h4>
		<p>{people.map(({name, salary}) => `${name}: $${salary.toLocaleString()}`).join(', ')}</p>
	{/if}

	{#if bills.length > 0}
		<h4>Bills</h4>
		{#each bills as bill (bill.id)}
			<p>{bill.payer} paid {bill.amount} for {bill.name.toLowerCase()}</p>
		{/each}
	{/if}

	{#if payments.length > 0}
		<h4>Payments</h4>
		{#each payments as [payer, amount, receiver]}
			<p>{payer} owes {receiver} ${Math.round((amount + Number.EPSILON) * 100) / 100}</p>
		{/each}
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>