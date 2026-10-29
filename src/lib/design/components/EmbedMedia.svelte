<script lang="ts">
	/* the player for an embed's og.video (poster: og.image) or, without one, og.audio (§4.8.1); nothing when neither loads.
	   Shared by uploads and link cards. */
	import type { OG } from './types';
	import { ogRatio, safeHttp } from './util';

	let { og }: { og: OG } = $props();
	const video = $derived(og.video && safeHttp(og.video.url));
	const audio = $derived(og.audio && safeHttp(og.audio.url));
	const image = $derived(og.image && safeHttp(og.image.url));
</script>

{#if video}
	<!-- svelte-ignore a11y_media_has_caption -->
	<video class="ap-embed-media" src={video} poster={image} controls preload="metadata" style={ogRatio(og.video) || ogRatio(og.image)}></video>
{:else if audio}
	<audio class="ap-embed-audio" src={audio} controls preload="none"></audio>
{/if}
