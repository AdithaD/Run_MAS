<script lang="ts">
	import { page } from '$app/state';
	import { onMount } from 'svelte';
	const BENCHMARK_DISTANCE = 1600;
	type Training = {
		name: string;
		setCount: number;
		repCount: number;
		repDuration: number;
		repRecovery: number;
		setRecovery: number | null;
		intensity: number;
	};

	let speed = $state<number | null>(null);

	let benchMins = $state<number | null>(null);
	let benchSeconds = $state<number | null>(null);

	onMount(() => {
		const min = page.url.searchParams.get('min');
		const sec = page.url.searchParams.get('sec');

		if (min && sec) {
			try {
				benchMins = parseInt(min);
				let parsedSec = parseInt(sec);
				if (parsedSec > 59) {
					parsedSec = 59;
				}
				benchSeconds = parsedSec;
				setState(benchMins * 60 + benchSeconds);

				return;
			} catch (error) {
				console.error("Couldn't parse query params");
			}
		}

		// query overrides local storage
		const string = localStorage.getItem('mas');
		if (string) {
			try {
				const seconds = parseInt(string);
				setState(seconds);
			} catch (error) {
				console.error("Couldn't parse localstorage");
			}
		}
	});

	function setState(seconds: number) {
		speed = BENCHMARK_DISTANCE / seconds;
		benchMins = Math.floor(seconds / 60);
		benchSeconds = seconds - benchMins * 60;
	}

	function calculateSpeed() {
		const seconds = (benchMins ?? 0) * 60 + (benchSeconds ?? 0);
		localStorage.setItem('mas', seconds.toString());

		speed = 1600 / seconds;
	}

	const masTraining = <Training[]>[
		{
			name: 'Short HIIT',
			setCount: 2,
			repCount: 8,
			repDuration: 45,
			repRecovery: 15,
			setRecovery: 5 * 60,
			intensity: 0.975
		},
		{
			name: 'Long HIIT',
			setCount: 1,
			repCount: 4,
			repDuration: 4 * 60,
			repRecovery: 2 * 60,
			setRecovery: null,
			intensity: 0.925
		},
		{
			name: 'Threshold HIIT',
			setCount: 1,
			repCount: 2,
			repDuration: 8 * 60,
			repRecovery: 2 * 60,
			setRecovery: null,
			intensity: 0.9
		}
	];
</script>

<svelte:head>
	<meta
		name="description"
		content="A calculator for determining distance for Maximum Aerobic Speed Training."
	/>
</svelte:head>

<div class="bg-gradient-from-b flex justify-center bg-gradient-to-t from-accent to-secondary">
	<div class="flex min-h-screen w-full flex-col bg-base-100 lg:w-2/3">
		<div class="flex flex-col gap-4 px-16 pt-16">
			<h1 class="text-4xl font-bold">Run MAS</h1>
			<p>
				Maximum Aerobic Speed (MAS) HIIT is a training method that tailors high-intensity intervals
				to an individual’s aerobic capacity, ensuring each session is both efficient and effective.
				MAS represents the lowest running speed at which maximal oxygen uptake (VO₂max) is reached,
				making it a precise marker for conditioning work. In practice, athletes perform short
				bursts—often between 15 seconds and 4 minutes—at or above their MAS, interspersed with
				active or passive recovery intervals. This approach allows for significant time spent near
				VO₂max without the unsustainable strain of continuous maximal running. Over time, MAS HIIT
				improves aerobic power, running economy, and lactate threshold, making it a powerful tool
				for endurance athletes and team-sport players seeking to enhance fitness while minimizing
				excessive training volume.
			</p>
			<p>
				The provided distance is your goal distance to meet in the alloted time. Try to reach this
				distance consistently each rep right at the end of the duration. Once you're finding this
				too easy, run another benchmark run.
			</p>
			<div class="divider"></div>
			<h2 class="text-2xl font-bold">MAS Speed Calculator</h2>
			<p>Conduct a 1.6k run benchmark time and input your result below.</p>
			<form class="flex gap-4">
				<label for="min" class="input">
					<input
						type="number"
						name="min"
						id="min"
						min="0"
						placeholder="Minutes"
						bind:value={benchMins}
					/>
					<span class="label">Minutes</span>
				</label>
				<label for="sec" class="input">
					<input
						type="number"
						name="sec"
						id="sec"
						min="0"
						max="59"
						placeholder="Seconds"
						bind:value={benchSeconds}
					/>
					<span class="label">Seconds</span>
				</label>
				<button class="btn btn-primary" onclick={calculateSpeed}>Calculate!</button>
			</form>
			{#if speed}
				<p class="text-lg">
					Your speed is: <span class="font-bold">{speed?.toFixed(1)}</span> metres / second
				</p>
			{/if}
			<div class="flex flex-col gap-4">
				{#each masTraining as { name, intensity, repCount, repDuration, repRecovery, setCount, setRecovery }}
					<div class="card bg-base-200">
						<div class="card-body">
							<div class="card-title flex justify-between">
								<div class="font-bold">{name}</div>
								{#if speed}
									<div class="text-2xl font-bold">
										{(speed * intensity * repDuration).toFixed(0)} m / rep
									</div>
								{/if}
							</div>
							<div>
								{setCount} x {repCount} reps @ {intensity * 100}% MAS
							</div>
							<div>
								{repDuration}s on with {repRecovery}s passive recovery{#if setRecovery}, {setRecovery}s
									between sets
								{/if}
							</div>
						</div>
					</div>
				{/each}
			</div>
		</div>
		<footer class="footer-center my-8 footer footer-horizontal text-neutral">
			By Aditha Doratiyawa
		</footer>
	</div>
</div>
