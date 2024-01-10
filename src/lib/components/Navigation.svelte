<script>
    import { getDrawerStore } from "@skeletonlabs/skeleton";

    const drawerStore = getDrawerStore();

    function drawerClose() {
        drawerStore.close();
    }
</script>

<nav class="list-nav p-4">
    <ul>
        <li><a href="/" on:click={drawerClose}>placeholder</a></li>
        <li><a href="/" on:click={drawerClose}>placeholder</a></li>
    </ul>
</nav>