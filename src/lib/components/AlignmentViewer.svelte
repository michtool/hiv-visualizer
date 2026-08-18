<script lang="ts">
    let { hxb2, reference, sample } = $props();
</script>
<main>
    <div class="alignment-container">
        <div class="sequence-name">{hxb2.name}</div>
        <div class="sequence">{hxb2.sequence}</div>

        <div class="sequence-name">{reference.name} (ref)</div>
        <div class="sequence">{reference.sequence}</div>

        <div class="sequence-name">{sample.name}</div>
        <div class="sequence">{sample.sequence}</div>
    </div>
</main>
<style>
    .alignment-container {
        display: grid;
        grid-template-columns: max-content max-content;
        width: max-content;
        min-width: 100%;
    }
    .alignment-container {
        overflow-x: auto;
        width: 100%;
        border: 1px solid #d0d0d0;
    }


    .sequence-name {
        position: sticky;
        left: 0;
        padding: 5px;
        background: #ffffff;
        border-right: 1px solid #d0d0d0;
        z-index: 1;
        font-family: monospace;
        box-shadow: 2px 0 4px rgba(0, 0, 0, 0.08);
    }

    .sequence {
        padding: 5px;
        font-family: monospace;
        letter-spacing: 1px;
    }
</style>