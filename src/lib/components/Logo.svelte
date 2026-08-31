<script lang="ts">
    let {
        size = "large"
    }: {
        size?: "small" | "large";
    } = $props();
</script>

<h1 class:small={size === "small"} class:large={size === "large"}>
    <span class="en">En</span><span class="v">v</span><span class="ision">Ision</span>
</h1>

<style>
    @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500;1,600;1,700&display=swap');
    h1 {
        margin: 0;
        font-family: "IBM Plex Sans", system-ui, sans-serif;
        font-weight: 600;
        letter-spacing: -0.03em;
    }

    .large {
        font-size: 3rem;
        line-height: 1;
    }

    .small {
        font-size: 1.4rem;
        line-height: 1;
    }

    .en {
        color: rgb(39, 68, 130);
    }

    .v {
        color: rgb(49, 29, 122);
    }

    .ision {
        color: rgb(101, 32, 66);
    }
</style>