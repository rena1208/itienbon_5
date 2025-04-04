<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<svelte:head>
	<title>防災マップ</title>
	<link rel="manifest" href="/manifest.webmanifest" />
	<link rel="icon" type="image/png" href="/icons/icon-192x192.png" />
	<meta name="theme-color" content="#0066cc" />
</svelte:head>
{@render children()}
