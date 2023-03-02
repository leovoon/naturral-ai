<script lang="ts">
	import type { CreateCompletionResponse } from 'openai'
	import { SSE } from 'sse.js'
	import { clipboard } from '@skeletonlabs/skeleton'

	let context = ''
	let loading = false
	let error = false
	let answer = ''
	let answers: Array<string> = []
	let contextLimit = 256

	$: contextLength = context.length
	$: maxlengthHit = contextLength >= contextLimit

	const handleSubmit = async () => {
		loading = true
		error = false
		answer = ''
		answers = []

		const eventSource = new SSE('/api/explain', {
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({ context })
		})

		context = ''

		eventSource.addEventListener('error', (e) => {
			error = true
			loading = false
			alert('Something went wrong!')
		})

		eventSource.addEventListener('message', (e) => {
			try {
				loading = false

				if (e.data === '[DONE]') {
					return
				}

				const completionResponse: CreateCompletionResponse = JSON.parse(e.data)

				const [{ text }] = completionResponse.choices

				if ((text?.match(/\n/) || []).length) {
					return
				}

				answer = (answer ?? '') + text
				if (answer.includes('-')) {
					let nextAnswer = answer.split('-')
					nextAnswer = nextAnswer.map((answer) => answer.trim())
					answers.push(...nextAnswer.filter((answer) => answer !== ''))
					answers = answers
					answer = ''
				}
			} catch (err) {
				error = true
				loading = false
				console.error(err)
				alert('Something went wrong!')
			}
		})

		eventSource.stream()
	}
</script>

<h1 class="gradient-heading ">naturral</h1>
<form on:submit|preventDefault={() => handleSubmit()}>
	<div class="grid gap-2">
		<label class="label text-center text-surface-50" for="context"
			>If you want your text to be standard English and sound natural,</label
		>
		<textarea
			class="textarea"
			name="context"
			maxlength="256"
			rows="5"
			bind:value={context}
			placeholder="Enter it here"
		/>
		{#if contextLength}
			<span class="text-sm place-self-end {maxlengthHit ? 'text-error-400' : 'text-surface-400'} "
				>{contextLength}/{contextLimit}</span
			>
		{/if}
		<button class="self-end btn variant-filled-primary" disabled={!context}>Get it</button>
	</div>
	<div class="pt-4 space-y-4">
		{#if loading}
			<div class="flex flex-col gap-4">
				<div class="grid gap-1">
					<div class="placeholder animate-pulse h-14" />
					<div class="place-self-end placeholder animate-pulse w-14" />
				</div>
				<div class="grid gap-1">
					<div class="placeholder animate-pulse h-14" />
					<div class="place-self-end placeholder animate-pulse w-14" />
				</div>

				<div class="grid gap-1">
					<div class="placeholder animate-pulse h-14" />
					<div class="place-self-end placeholder animate-pulse w-14" />
				</div>
			</div>
		{/if}

		{#if error}
			<h2>Result:</h2>
			<p class="text-error">Something went wrong.</p>
		{/if}

		{#if answers.length}
			<h2>Result:</h2>
			<ul class="grid gap-6">
				{#each answers as answer, id}
					<li class="flex flex-col gap-1">
						<textarea class="block textarea" value={answer} data-clipboard={id} />
						<span class="place-self-end chip variant-filled-primary" use:clipboard={{ input: id }}
							>Copy</span
						>
					</li>
				{/each}
			</ul>
		{/if}
	</div>
</form>
