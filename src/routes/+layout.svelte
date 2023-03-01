<script>
	import '@skeletonlabs/skeleton/themes/theme-hamlindigo.css'

	import '@skeletonlabs/skeleton/styles/all.css'

	import '../app.css'

	import { autoModeWatcher } from '@skeletonlabs/skeleton'
</script>

<svelte:head
	>{@html `<script>${autoModeWatcher.toString()} autoModeWatcher();</script>`}</svelte:head
>
<div class="flex flex-col min-h-screen justify-between items-center py-14 space-y-4">
	<main class="flex flex-col items-center gap-4">
		<slot />
	</main>

	<footer class="italic text-surface-500">by leovoon</footer>
</div>
