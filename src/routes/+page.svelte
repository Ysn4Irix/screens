<script lang="ts">
	import loader from '$src/icons/loader.svg'
	import logo from '$src/icons/logo.svg'

	let vars = {
		errorMsg: '',
		loading: false,
		url: '',
		width: 1920,
		height: 1080,
		image: ''
	}

	const takeScreenshot = async (): Promise<void> => {
		vars.errorMsg = ''
		vars.loading = true

		const options = {
			method: 'POST',
			body: JSON.stringify({
				url: vars.url,
				width: vars.width,
				height: vars.height
			}),
			headers: {
				'Content-Type': 'application/json'
			}
		}

		const apiUrl =
			process.env.NODE_ENV === 'production'
				? 'https://screens.ysnirix.live/api/screenshot'
				: 'http://localhost:4444/api/screenshot'

		try {
			const response = await fetch(apiUrl, options)
			if (response.ok) {
				const { results } = await response.json()
				vars.image = results.base64
				vars.loading = false
			} else if (response.status === 422) {
				const { errors } = await response.json()
				vars.errorMsg = errors
				vars.loading = false
			} else {
				const { message } = await response.json()
				vars.errorMsg = message
				vars.loading = false
			}
		} catch (err: any) {
			vars.errorMsg = err.message
			vars.loading = false
		}
	}
</script>

<svelte:head>
	<title>Screens</title>
</svelte:head>

<div class="flex flex-col justify-center text-center items-center font-fresca">
	<img class="w-56 mb-4" src={logo} alt="screens" />

	<input
		type="text"
		placeholder="Website url..."
		class="input input-secondary input-bordered w-full max-w-xs"
		bind:value={vars.url}
	/>

	<div class="flex flex-row mt-4">
		<input
			type="text"
			placeholder="Image width..."
			class="input input-secondary input-bordered mx-4 w-full max-w-xs"
			bind:value={vars.width}
		/>

		<input
			type="text"
			placeholder="Image height..."
			class="input input-secondary input-bordered mx-4 w-full max-w-xs"
			bind:value={vars.height}
		/>
	</div>
	<button
		on:click={takeScreenshot}
		class:btn-disabled={vars.url === '' || vars.loading}
		type="button"
		class="btn btn-secondary mt-2 shadow-sm w-72"
	>
		Take Screenshot
	</button>
	{#if vars.errorMsg}
		<h3 class="text-center text-red-500 mt-5">{vars.errorMsg}</h3>
	{:else if vars.loading}
		<img class="w-36 mt-5" src={loader} alt="loading" />
	{:else}
		<div class:hidden={!vars.image} class="py-4 px-12">
			<img
				class="rounded-lg shadow-md"
				src={`data:image/png;base64,${vars.image}`}
				alt="screenshot"
			/>
		</div>
	{/if}
</div>
