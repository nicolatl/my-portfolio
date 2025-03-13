<script>
    import { page } from "$app/stores";
    let pages = [
        { url: "./", title: "Home" },
        { url: "./projects", title: "Projects" },
        { url: "./contact", title: "Contact" },
        {url: "resume", title: "Resume"},
        {url: "https://github.com/nicolatl", title: "Github"}
    ];

    
    let localStorage = globalThis.localStorage ?? {};
    let colorScheme = localStorage.colorScheme ?? "light dark";
    $: localStorage.colorScheme = colorScheme;
    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);



</script>

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="dark">Dark</option>
        <option value="light">Light</option>
    </select>
</label>
  
<nav>
    {#each pages as p}
    <a
        href={p.url}
        class:current={"." + $page.route.id === p.url}
        target={p.url.startsWith("http") ? "_blank" : null}
    >
        {p.title}
    </a>
            
    {/each}
</nav>
<style>
    .color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        display: inline-flex;
        gap: 4px;
    }
    nav {
        --border-color: oklch(50% 10% 200 / 40%);
        display: flex;
        margin-bottom: 1rem;
        border-bottom: 1px solid var(--border-color);
    }
    nav a {
        flex: 1;
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: 0.5em;
    }

    nav a.current {
    border-bottom: 0.4em solid var(--border-color);
    padding-bottom: 0.1em;
    }

    nav a:hover {
    border-bottom: 0.4em solid var(--color-accent);
    padding-bottom: 0.1em;
    }
</style>  
<slot />