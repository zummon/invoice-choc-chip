<script>
	import { onMount } from "svelte";
	export let data;

	let l = data[""].label[""];
	let q = data[""].q;

	const price = number => {
	  number = Number(number);
	  if (number === 0 || isNaN(number)) {
	    return "";
	  }
	  return `${q.currency} ${number.toLocaleString(undefined, {
	    minimumFractionDigits: 2,
	  })}`;
	};
	const qty = number => {
	  number = Number(number);
	  if (number === 0 || isNaN(number)) {
	    return "";
	  }
	  return number.toLocaleString(undefined, {
	    minimumFractionDigits: 0,
	  });
	};
	const rate = rate => {
	  rate = Number(rate) * 100;
	  if (!Number.isInteger(rate)) {
	    rate = rate.toFixed(2);
	  }
	  return `${rate} %`;
	};
	const addItem = () => {
	  q.itemDesc.push("");
	  q.itemPrice.push("");
	  q.itemQty.push("");
	  q = q;
	};
	const removeItem = () => {
	  q.itemDesc.pop();
	  q.itemPrice.pop();
	  q.itemQty.pop();
	  q = q;
	};

	onMount(() => {
	  const s = new URLSearchParams(location.search);
	  let obj = q;
	  Object.keys(q).forEach(key => {
	    const values = s.getAll(key);
	    if (values.length > 0) {
	      if (Array.isArray(q[key])) {
	        obj[key] = values;
	        return;
	      }
	      obj[key] = values[0];
	    }
	  });
	  q = { ...data[q.lang].q, ...obj };
	});
	$: {
	  document.body.style = data[q.lang]["font-style"];
	}
	
	$: l = {
	  ...data[q.lang].label[""],
	  ...data[q.lang].label[q.doc]
	};
	$: q.itemAmount = q.itemPrice.map((pr, i) => {
	  const num = Number(pr) * Number(q.itemQty[i]);
	  return num ? num : "";
	});
	$: q.totalAmount = q.itemAmount.reduce((a, b) => {
	  const num = Number(a) + Number(b);
	  return num ? num : "";
	}, 0);
	$: q.totalVat = Number(q.totalAmount) * Number(q.vatRate);
	$: q.totalWht = Number(q.totalAmount) * Number(q.whtRate);
	$: q.totalFinal =
	  Number(q.totalAmount) +
	  Number(q.totalVat) +
	  Number(q.totalWht) +
	  Number(q.totalAdjust);
</script>

<svelte:head>
	<link href={data[q.lang]['font-link']} rel="stylesheet"/>
</svelte:head>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
{#each Object.keys(data) as lng, i (`lang-${i}`)}
		<button class="block duration-300 py-1 px-2 rounded {q.lang === lng ? "bg-yellow-800 text-white" : "text-yellow-800 bg-yellow-300 hover:bg-yellow-500 focus:bg-yellow-500"}" on:click={() => {
			q.lang = lng
		}}>
			{data[lng]['']}
		</button>
	{/each}
	{#each Object.keys(data[q.lang].label) as dc, i (`doc-${i}`)}
		<button class="block duration-300 py-1 px-2 rounded {q.doc === dc ? "bg-yellow-800 text-white" : "text-yellow-800 bg-yellow-300 hover:bg-yellow-500 focus:bg-yellow-500"}" on:click={() => {
			q.doc = dc
		}}>
			{data[q.lang].label[dc].title}
		</button>
	{/each}
</div>

<div class="bg-white text-yellow-900 p-3 max-w-[60rem] mx-auto print:max-w-none print:mx-0" >
	<div class="flex">
		<div class="flex-1">
			<h1 class="text-4xl text-center border-b-4 border-yellow-300 pb-2">{l.title}</h1>
			<div class="flex pt-1">
				<div class="flex-1 pr-3 text-right">{l.ref}</div>
				<p class="flex-1" contenteditable="true" bind:textContent={q.ref}></p>
			</div>
			<div class="flex">
				<div class="flex-1 pr-3 text-right">{l.date}</div>
				<p class="flex-1" contenteditable="true" bind:textContent={q.date}></p>
			</div>
			{#if q.doc !== 'receipt'}
				<div class="flex">
					<div class="flex-1 pr-3 text-right">{l.duedate}</div>
					<p class="flex-1" contenteditable="true" bind:textContent={q.duedate}></p>
				</div>
			{/if}
		</div>
		<div class="text-right">
			<div class="">
				<img class="" src={q.vendorLogo} alt="" width="" height="">
			</div>
			<h3 class="text-xl" contenteditable="true" bind:textContent={q.vendorName}></h3>
			<p class="" contenteditable="true" bind:textContent={q.vendorId}></p>
			<p class="" contenteditable="true" bind:textContent={q.vendorAddress}></p>
		</div>
	</div>
	<div class="flex my-3">
		<div class="w-1/2 flex">
			<div class="pr-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.client}</span>
			</div>
			<div class="flex-1">
				<h3 class="text-xl" contenteditable="true" bind:textContent={q.clientName}></h3>
				<p class="" contenteditable="true" bind:textContent={q.clientId}></p>
				<p class="break-all" contenteditable="true" bind:textContent={q.clientAddress}></p>
			</div>
		</div>
		<div class="w-1/2 flex">
			<div class="px-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.paymethod}</span>
			</div>
			<div class="flex-1">
				<p class="break-all" contenteditable="true" bind:textContent={q.paymethod}></p>
			</div>
		</div>
	</div>
	<div class="flex">
		<div class="w-1/2 flex">
			<div class="pr-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.subject}</span>
			</div>
			<div class="flex-1">
				<p class="break-all" contenteditable="true" bind:textContent={q.subject}></p>
			</div>
		</div>
		<div class="w-1/2 flex">
			<div class="px-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.totalFinal}</span>
			</div>
			<div class="flex-1">
				<p class="" contenteditable="true"
				>
					{price(q.totalFinal)}
				</p>
			</div>
		</div>
	</div>
	<table class="w-full my-3">
		<thead class="text-center">
			<tr class="bg-yellow-300">
				<td class="p-2 w-px whitespace-nowrap">{l.itemNo}</td>
				<td class="">
					<div class="flex">
						<p class="p-2 flex-grow">{l.itemDesc}</p>
						<button class="p-2.5 focus:bg-yellow-100 hover:bg-yellow-100 transition duration-300 ease-in-out rounded print:hidden" on:click={addItem}>
							<!-- heroicons solid duplicate -->
							<svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
								<path d="M7 9a2 2 0 012-2h6a2 2 0 012 2v6a2 2 0 01-2 2H9a2 2 0 01-2-2V9z" />
								<path d="M5 3a2 2 0 00-2 2v6a2 2 0 002 2V5h8a2 2 0 00-2-2H5z" />
							</svg>
						</button>
						<button class="p-2.5 focus:bg-yellow-100 hover:bg-yellow-100 transition duration-300 ease-in-out rounded print:hidden" on:click={removeItem}>
							<!-- heroicons solid trash -->
							<svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
								<path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
							</svg>
						</button>
					</div>
				</td>
				<td class="p-2 w-px whitespace-nowrap">{l.itemPrice}</td>
				<td class="p-2 w-px whitespace-nowrap">{l.itemQty}</td>
				<td class="p-2 w-px whitespace-nowrap">{l.itemAmount}</td>
			</tr>
		</thead>
		<tbody class="divide-y">
			{#each q.itemDesc as _, i (`item-${i}`)}
				<tr class="">
					<td class="py-1 px-2 text-center whitespace-nowrap" contenteditable="true">{i + 1}</td>
					<td class="py-1 px-2" contenteditable="true" bind:textContent={q.itemDesc[i]}></td>
					<td class="py-1 px-2 text-right whitespace-nowrap" contenteditable="true" 
						on:focus={(e) => e.target.textContent = q.itemPrice[i]}
						on:input={(e) => q.itemPrice[i] = e.target.textContent}
						on:blur={(e) => e.target.textContent = price(q.itemPrice[i])}
					>
						{price(q.itemPrice[i])}
					</td>
					<td class="py-1 px-2 text-right whitespace-nowrap" contenteditable="true" 
						on:focus={(e) => e.target.textContent = q.itemQty[i]}
						on:input={(e) => q.itemQty[i] = e.target.textContent}
						on:blur={(e) => e.target.textContent = qty(q.itemQty[i])}
					>
						{qty(q.itemQty[i])}
					</td>
					<td class="py-1 px-2 text-right whitespace-nowrap">
						{price(q.itemAmount[i])}
					</td>
				</tr>
			{/each}
		</tbody>
		<tfoot class="text-right">
			<tr class="bg-yellow-300">
				<td class="p-2" colspan="4">{l.totalAmount}</td>
				<td class="p-2 whitespace-nowrap">
					{price(q.totalAmount)}
				</td>
			</tr>
			<tr class="">
				<td class="p-2" colspan="4">
					<span class="">{l.totalVat}</span>
					<span class=""> </span>
					<span class="py-1 px-2 bg-yellow-800 text-white rounded" contenteditable="true" 
						on:focus={(e) => e.target.textContent = q.vatRate}
						on:input={(e) => q.vatRate = e.target.textContent}
						on:blur={(e) => e.target.textContent = rate(q.vatRate)}
					>
						{rate(q.vatRate)}
					</span>
				</td>
				<td class="p-2 whitespace-nowrap">
					{price(q.totalVat)}
				</td>
			</tr>
			{#if q.doc === 'receipt'}
				<tr class="">
					<td class="p-2" colspan="4">
						<span class="">{l.totalWht}</span>
						<span class=""> </span>
						<span class="py-1 px-2 bg-yellow-800 text-white rounded" contenteditable="true" 
							on:focus={(e) => e.target.textContent = q.whtRate}
							on:input={(e) => q.whtRate = e.target.textContent}
							on:blur={(e) => e.target.textContent = rate(q.whtRate)}
						>
							{rate(q.whtRate)}
						</span>
					</td>
					<td class="p-2 whitespace-nowrap">
						{price(q.totalWht)}
					</td>
				</tr>
			{/if}
			<tr class="">
				<td class="p-2" colspan="4">{l.totalAdjust}</td>
				<td class="p-2 whitespace-nowrap" contenteditable="true" 
					on:focus={(e) => e.target.textContent = q.totalAdjust}
					on:input={(e) => q.totalAdjust = e.target.textContent}
					on:blur={(e) => e.target.textContent = price(q.totalAdjust)}
				>
					{price(q.totalAdjust)}
				</td>
			</tr>
		</tfoot>
	</table>
	<div class="flex">
		<div class="pr-2">
			<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.note}</span>
		</div>
		<div class="flex-1">
			<p class="break-all" contenteditable="true" bind:textContent={q.note}></p>
		</div>
	</div>
	<div class="flex my-3">
		<div class="w-1/2 flex">
			<div class="pr-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.vendorSign}</span>
			</div>
			<div class="flex-1 text-right">
				<p class="" contenteditable="true"></p>
				<p class="" contenteditable="true"></p>
				<p class="" contenteditable="true"></p>
			</div>
		</div>
		<div class="w-1/2 flex">
			<div class="px-2">
				<span class="py-1 px-2 bg-yellow-800 text-white rounded">{l.clientSign}</span>
			</div>
			<div class="flex-1 text-right">
				<p class="" contenteditable="true"></p>
				<p class="" contenteditable="true"></p>
				<p class="" contenteditable="true"></p>
			</div>
		</div>
	</div>
	<div class="flex justify-end">
		<h3 class="text-xl border-t-4 border-yellow-300 pt-2 min-w-[80px]">{l.thankMessage}</h3>
	</div>
</div>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
	<button class="block duration-300 py-1 px-2 rounded text-yellow-800 bg-yellow-300 hover:bg-yellow-800 hover:text-white focus:bg-yellow-800 focus:text-white" on:click={() => window.print()}>
		Print
	</button>
</div>