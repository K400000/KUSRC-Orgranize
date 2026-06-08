<script lang="ts">
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href="https://admissions.src.ku.ac.th/assets/img/faviconx.png" />
</svelte:head>

{@render children()}
