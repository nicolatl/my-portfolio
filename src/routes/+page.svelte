<h1>Nicola</h1>
    <p>I am a masters student from Hamilton, Canada currently living in Cambridge, USA. Lorem ipsum dolor sit amet, consectetur adipisicing elit. Aliquid, dolore repudiandae nobis minus iusto ut necessitatibus similique autem modi culpa! Ut voluptatem ea fugit earum consequatur temporibus in quaerat? Fugit?</p>
    <img src="images/nicola.jpeg" width=400 alt="Black-and-white photo of Nicola smiling and holding an umbrella in front of a cat">
    <script>
        import projects from "$lib/projects.json";
        import Project from "$lib/Project.svelte";
    </script>
    {#await fetch("https://api.github.com/users/nicolatl")}
    <p>Loading...</p>
  {:then response}
    {#await response.json()}
      <p>Decoding...</p>
    {:then data}
      <section>
        <h2>My GitHub Stats</h2>
        <dl>
          <dt>Followers:</dt>
          <dd>{data.followers}</dd>
          <dt>Following:</dt>
          <dd>{data.following}</dd>
          <dt>Public Repositories:</dt>
          <dd>{data.public_repos}</dd>
        </dl>
      </section>
    {:catch error}
      <p class="error">Something went wrong: {error.message}</p>
    {/await}
  {:catch error}
    <p class="error">Something went wrong: {error.message}</p>
  {/await}  
    <h2>Latest Projects</h2>
    <div class="projects">
        {#each projects.slice(0, 3) as p}
        <Project data={p} hLevel="3" />
        {/each}
    </div>

<style>
    dl {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        padding: 20px;
        box-shadow: 2px 2px 2px grey, -2px -2px 2px grey;
    }
    dt {
        grid-row: 1;
        text-align: center;
    }
    dd {
        grid-row: 2;
        padding-left: 0;
        text-align: center;
    }
</style>
    