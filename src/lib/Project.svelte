<script>
    export let data = {};
    export let hLevel = 2;
</script>
<article>
    <svelte:element this={"h" + hLevel}>{data.title}</svelte:element>
    <img src={data.image} alt="">
    <p>{data.description}</p>
</article>
<style>
article {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}

h2 {
  margin: 0;
}
</style>