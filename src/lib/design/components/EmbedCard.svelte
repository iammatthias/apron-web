<script lang="ts">
	/* any kind described by og (link previews, unknown kinds with og). Previews come from the server; the client never unfurls.
	   An og.video or og.audio plays in the card (§4.8.1), above the text, which is the link. Media that waits for a click
	   (its `ask`: a video or audio that can't play, else an image that can't show) shows under the card as a MediaAsk.
	   Its `class` and `testid` go on the card, or on what holds the card and the ask. */
	import EmbedMedia from './EmbedMedia.svelte';
	import MediaAsk from './MediaAsk.svelte';
	import type { EmbedComponentProps } from './types';
	import { safeHttp } from './util';

	let props: EmbedComponentProps = $props();
	const og = $derived(props.og || {});
	const url = $derived(safeHttp(props.url));
	const video = $derived(og.video && safeHttp(og.video.url));
	const player = $derived(video || (og.audio && safeHttp(og.audio.url)));
	const img = $derived(!video && og.image && safeHttp(og.image.url));
	const ask = $derived(player ? undefined : og.video?.ask ? { ...og.video.ask, what: 'Video' } : og.audio?.ask ? { ...og.audio.ask, what: 'Audio' } : !img && og.image?.ask ? { ...og.image.ask, what: og.image.alt || 'Image' } : undefined);
</script>

{#snippet text()}
	{#if og.site_name}<span class="ap-embed-site">{og.site_name}</span>{/if}
	<span class="ap-embed-title">{og.title || props.title || url || props.kind}</span>
	{#if og.description}<span class="ap-embed-desc">{og.description}</span>{/if}
{/snippet}

{#snippet card(className: EmbedComponentProps['class'], testid: string | undefined)}
	<svelte:element this={url ? 'a' : 'div'} href={url} rel={url ? 'noreferrer noopener' : undefined} target={url ? '_blank' : undefined} data-testid={testid} class={['ap-embed', 'ap-embed-ogcard', img && 'ap-embed-ogcard-img', className]}>
		{#if img}<img class="ap-embed-ogimg" src={img} alt={og.image?.alt || ''} loading="lazy" />{/if}
		<span class="ap-embed-cardtext">{@render text()}</span>
	</svelte:element>
{/snippet}

{#if player}
	<div data-testid={props.testid} class={['ap-embed', 'ap-embed-ogcard', 'ap-embed-ogcard-player', props.class]}>
		{#if img}<img class="ap-embed-ogimg" src={img} alt={og.image?.alt || ''} loading="lazy" />{/if}
		<EmbedMedia {og} />
		<svelte:element this={url ? 'a' : 'span'} href={url} rel={url ? 'noreferrer noopener' : undefined} target={url ? '_blank' : undefined} class="ap-embed-cardtext">{@render text()}</svelte:element>
	</div>
{:else if ask}
	<div class={['ap-embed-held', props.class]} data-testid={props.testid}>
		{@render card(undefined, undefined)}
		<!-- spread: what Load and Always load do, and what waits: MediaAsk's own props -->
		<MediaAsk {...ask} />
	</div>
{:else}
	{@render card(props.class, props.testid)}
{/if}
