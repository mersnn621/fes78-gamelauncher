<script>
    let { children } = $props();
    import "../app.css";
  </script>

<main class="h-screen w-screen">
{@render children()}
</main>