<h1>Nicola</h1>
    <p>I am a masters student from Hamilton, Canada currently living in Cambridge, USA. Lorem ipsum dolor sit amet, consectetur adipisicing elit. Aliquid, dolore repudiandae nobis minus iusto ut necessitatibus similique autem modi culpa! Ut voluptatem ea fugit earum consequatur temporibus in quaerat? Fugit?</p>
    <img src="images/nicola.jpeg" width=400 alt="Black-and-white photo of Nicola smiling and holding an umbrella in front of a cat">
    <script>
        import projects from "$lib/projects.json";
        import Project from "$lib/Project.svelte";
    </script>
    <h2>Latest Projects</h2>
    <div class="projects">
        {#each projects.slice(0, 3) as p}
        <Project data={p} hLevel="3" />
        {/each}
    </div>
    