<script lang="ts">
	import { onMount } from 'svelte';
	import { marked } from 'marked';
	import Prism from 'prismjs';
	import 'prismjs/themes/prism-tomorrow.css';

	export let source: string;

	let notebook: any = null;
	let error: string | null = null;
	let loading = true;

	onMount(async () => {
		try {
			const res = await fetch(source);
			if (!res.ok) throw new Error(res.statusText);
			notebook = await res.json();
		} catch (e: any) {
			error = e.message;
		} finally {
			loading = false;
			setTimeout(() => Prism.highlightAll(), 0);
		}
	});

	// Helpers to handle both string and array-of-strings formats in JSON
	const renderMarkdown = (text: string | string[]) =>
		marked.parse(Array.isArray(text) ? text.join('') : text) as string;

	const renderCode = (text: string | string[]) => (Array.isArray(text) ? text.join('') : text);
</script>

<div
	class="notebook-container text-left w-full max-w-4xl mx-auto p-4 bg-white dark:bg-gray-900 rounded-lg shadow-sm border border-gray-200 dark:border-gray-800"
>
	{#if loading}
		<div class="p-8 text-center text-gray-500">Loading notebook...</div>
	{:else if error}
		<div class="p-8 text-center text-red-500">Error: {error}</div>
	{:else if notebook}
		{#each notebook.cells as cell}
			<div class="cell mb-6 group relative">
				{#if cell.cell_type === 'markdown'}
					<div
						class="markdown-cell prose dark:prose-invert max-w-none p-4 rounded-md hover:bg-gray-50 dark:hover:bg-gray-800/50 transition-colors"
					>
						{@html renderMarkdown(cell.source)}
					</div>
				{:else if cell.cell_type === 'code'}
					<div
						class="code-cell bg-gray-50 dark:bg-gray-950 rounded-md border border-gray-200 dark:border-gray-800 overflow-hidden"
					>
						<!-- Input Area -->
						<div class="input-area flex">
							<div
								class="prompt w-12 pt-4 text-right pr-2 text-xs font-mono text-gray-400 select-none"
							>
								In [{cell.execution_count || ' '}]:
							</div>
							<div class="code flex-1 min-w-0">
								<pre class="!m-0 !p-4 !bg-transparent text-sm"><code class="language-python"
										>{renderCode(cell.source)}</code
									></pre>
							</div>
						</div>

						<!-- Output Area -->
						{#if cell.outputs && cell.outputs.length > 0}
							<div
								class="output-area border-t border-gray-200 dark:border-gray-800 bg-white dark:bg-gray-900"
							>
								{#each cell.outputs as output}
									<div class="output p-4 pl-12">
										{#if output.output_type === 'stream'}
											<pre
												class="whitespace-pre-wrap font-mono text-sm text-gray-600 dark:text-gray-300">{Array.isArray(
													output.text
												)
													? output.text.join('')
													: output.text}</pre>
										{:else if output.output_type === 'execute_result' || output.output_type === 'display_data'}
											{#if output.data['text/html']}
												<div class="prose dark:prose-invert max-w-none">
													{@html Array.isArray(output.data['text/html'])
														? output.data['text/html'].join('')
														: output.data['text/html']}
												</div>
											{:else if output.data['image/png']}
												<img
													src={`data:image/png;base64,${output.data['image/png'].replace(/\n/g, '')}`}
													alt="Output"
													class="max-w-full h-auto"
												/>
											{:else if output.data['text/plain']}
												<pre
													class="whitespace-pre-wrap font-mono text-sm text-gray-600 dark:text-gray-300">{Array.isArray(
														output.data['text/plain']
													)
														? output.data['text/plain'].join('')
														: output.data['text/plain']}</pre>
											{/if}
										{:else if output.output_type === 'error'}
											<div
												class="text-red-600 dark:text-red-400 font-mono text-sm whitespace-pre-wrap"
											>
												<span class="font-bold">{output.ename}:</span>
												{output.evalue}
												{#if output.traceback}
													<div class="mt-2 opacity-80">
														{output.traceback.join('\n')}
													</div>
												{/if}
											</div>
										{/if}
									</div>
								{/each}
							</div>
						{/if}
					</div>
				{/if}
			</div>
		{/each}
	{/if}
</div>
