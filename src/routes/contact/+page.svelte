<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>
    <p>Please do not contact me in this way.</p>
    <form action="mailto:77587217+nicolatl@users.noreply.github.com" method="GET">
        <label>
            Email:
            <input type="email" name="email" required>
        </label>
        <label>
            Subject:
            <input name="subject" required>
        </label>
        <label>
            Message:
            <textarea name="body" required></textarea>
        </label>
        <button>Send</button>
    </form>
