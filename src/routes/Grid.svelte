<script lang="ts">
	import { goto } from '$app/navigation';
	import { Counter, PaintingPaths, Paintings } from '$lib/schema';
	import { useCoState } from 'jazz-svelte';
	import { Group } from 'jazz-tools';
	import Preview from './Preview.svelte';
	import { Alert, Box, Button, Heading } from '@fuxui/base';
	import { base } from '$app/paths';

	function createPainting() {
		const group = Group.create();
		group.addMember('everyone', 'writer');

		const newPainting = Counter.create(
			{
				painting: PaintingPaths.create([], group),
				count: 0
			},
			group
		);

		if (!paintings.current) {
			let newPaintings = Paintings.create([newPainting], group);
			console.log('created new paintings', newPaintings.id);
		} else {
			paintings.current?.push(newPainting);
		}

		goto(base + '/?id=' + newPainting.id, {});
	}

	// old id: co_zNQSuU459j4goZJdGAit732RiN9

	const paintings = $derived(
		useCoState(Paintings, 'co_zFLBdv7QU89Jaq8EJEF9hAWLkPs', {
			resolve: {
				$each: {
					painting: {
						$each: {
							segments: {
								$each: true
							}
						}
					}
				}
			}
		})
	);

	let emptyPaintingsCount = $derived(
		paintings.current?.filter((painting) => painting.count === 0).length
	);
</script>

<div class="mx-auto max-w-5xl px-4 py-16">
	<Heading class="mb-4">Multiplayer Drawings</Heading>

	<Alert title="Everything is public" class="mb-8">
		<p>Please note that all drawings are public and can be viewed and edited by anyone. Also this is just a demo, so your drawings might be deleted at any time.</p>
	</Alert>

	<div class="grid gap-4 sm:grid-cols-2 md:grid-cols-3">
		<Button onclick={createPainting} class="h-50 w-full cursor-pointer overflow-hidden" size="lg">
			Create Painting
		</Button>

		{#if paintings.current}
			{#each paintings.current.toReversed() as painting, index (painting.id)}
				{#if painting.count > 0 && index < 10}
					<Box class="h-50 p-0">
						<button
							class="absolute inset-0 h-full w-full cursor-pointer overflow-hidden transition-all duration-200 hover:scale-105 active:scale-95"
							onclick={() => {
								goto(base + '/?id=' + painting.id);
							}}
						>
							<Preview {painting} />
						</button>
					</Box>
				{/if}
			{/each}
		{/if}
	</div>

	{#if emptyPaintingsCount && emptyPaintingsCount > 0}
		<div class="text-base-500 mt-4 text-xs">{emptyPaintingsCount} empty paintings not shown</div>
	{/if}
</div>
