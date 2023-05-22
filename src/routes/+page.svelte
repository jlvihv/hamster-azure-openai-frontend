<script lang="ts">
	import { SSE } from 'sse.js';
	import SvelteMarkdown from 'svelte-markdown';

	interface Response {
		content: string;
	}
	interface Message {
		text: string;
		sender: 'user' | 'gpt';
	}

	let message = '';
	let messages: Array<Message> = [];

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Enter') {
			sendMessage();
		}
	}

	async function sendMessage() {
		messages = [...messages, { text: message, sender: 'user' }];
		var source = new SSE('http://175.24.179.2:10000', {
			// var source = new SSE('http://127.0.0.1:10000', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({
				message
			})
		});
		messages = [...messages, { text: '', sender: 'gpt' }];
		source.addEventListener('message', (e: any) => {
			try {
				let result: Response = JSON.parse(e.data);
				let lastMessage = messages[messages.length - 1];
				lastMessage.text += result.content;
				messages[messages.length - 1] = lastMessage;
				messages = messages;
			} catch (e) {
				console.error(e);
			}
		});
		source.stream();
		message = '';
	}
</script>

<div class="flex min-h-screen flex-col justify-center bg-gray-100 py-6 sm:py-12">
	<div class="relative py-3 sm:mx-auto sm:max-w-xl">
		<div
			class="to-light-blue-500 absolute inset-0 -skew-y-6 transform bg-gradient-to-r from-cyan-400 shadow-lg sm:-rotate-6 sm:skew-y-0 sm:rounded-3xl"
		/>
		<div class="relative bg-white px-4 py-10 shadow-lg sm:rounded-3xl sm:p-20">
			<div class="mx-auto max-w-lg">
				<div class="divide-y divide-gray-200">
					<div class="space-y-4 py-8 text-base leading-6 text-gray-700 sm:text-lg sm:leading-7">
						<div class="flex flex-col">
							{#each messages as msg}
								<div
									class={`mb-4 rounded-lg p-2 ${
										msg.sender === 'user'
											? 'bg-blue-200 text-blue-800'
											: 'bg-green-200 text-green-800'
									}`}
								>
									<SvelteMarkdown source={msg.text} />
								</div>
							{/each}
						</div>
						<div class="mt-6 flex">
							<input
								type="text"
								bind:value={message}
								class="block w-full flex-1 rounded-md border border-gray-300 px-2 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
								placeholder="Type your message..."
								on:keydown={handleKeydown}
							/>
							<button
								class="ml-3 flex-shrink-0 rounded-md border border-transparent bg-indigo-600 px-6 py-2 text-base font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
								on:click={sendMessage}>Send</button
							>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
