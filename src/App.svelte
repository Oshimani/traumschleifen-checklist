<script>
	import { onMount } from "svelte";

	let loadedTS = false;
	let traumschleifen = [];

	$: totalCompleted = traumschleifen.reduce((pv, cv) => {
		return cv.completed === true ? (pv = pv + 1) : pv;
	}, 0);

	onMount(() => {
		fetch("/assets/data/traumschleifen.json").then((data) =>
			data.json().then((tsdata) => {
				console.log("Traumschleifen", tsdata);
				loadedTS = true;

				const completionData = JSON.parse(localStorage.getItem("ts"));
				traumschleifen = tsdata.map((ts) => {
					const match = completionData.indexOf(ts.name);
					return {
						name: ts.name,
						length: ts.length,
						elevation: ts.elevation,
						completed: match > -1 ? true : false,
					};
				});
			})
		);
	});

	const onCheck = (ts) => {
		ts.completed = !ts.completed;
	};

	const onSave = () => {
		localStorage.setItem(
			"ts",
			JSON.stringify(
				traumschleifen.filter((ts) => ts.completed).map((ts) => ts.name)
			)
		);
	};
</script>

<main class="container">
	<div class="row text-center">
		<div class="col">
			<h1>Traumschleifen Checklist</h1>
			<h2>{totalCompleted}/{traumschleifen.length}</h2>
		</div>
	</div>

	<div class="row">
		<div class="col">
			<ul class="list-group">
				{#each traumschleifen as ts (ts.name)}
					<!-- content here -->
					<li class="list-group-item">
						<input
							class="form-check-input me-1"
							type="checkbox"
							name={ts.name}
							id={ts.name}
							bind:checked={ts.completed}
							on:change={(ts) => onCheck(ts)}
						/>
						<label for={ts.name}>{ts.name}</label>
					</li>
				{/each}
			</ul>
		</div>
	</div>
</main>

<section class="position-fixed bottom-0 start-50 pb-2">
	<button type="button" on:click={() => onSave()} class="btn btn-primary"
		>Save</button
	>
</section>

<style>
	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}
	.list-group label {
		cursor: pointer;
	}
</style>
