<script>
    import Graph from "./Graph.svelte";
</script>
<section class="container" transform="translate(100, 10)">
    <svelte:component this={Graph}/></section>
<style>
    .container {
    max-width: 750px;
    border: solid 1px black;
    margin: auto;
    }
</style>