<script>
	import Notebook from '$lib/components/Notebook.svelte';
</script>

<div class="container mx-auto px-4 py-8">
	<div class="mb-8 text-center">
		<h1 class="text-3xl font-bold mb-4">Statue Notebook Component</h1>
		<p class="text-gray-600 dark:text-gray-400">
			Native rendering of Jupyter Notebooks (.ipynb) in Svelte
		</p>
	</div>

	<Notebook source="/sample.ipynb" />
</div>
