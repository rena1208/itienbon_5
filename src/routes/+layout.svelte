<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<svelte:head>
	<title>防災マップ</title>
</svelte:head>
{@render children()}
