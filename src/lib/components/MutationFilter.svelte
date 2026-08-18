<script lang="ts">
    let filterOpen = $state(false);
</script>
<div class="filter-controls">
    <button
        class="filter-button"
        onclick={() => filterOpen = !filterOpen}
    >
        Filter
    </button>

    {#if filterOpen}
        <div class="filter-panel">
            <div class="filter-column">
                <h3>Type</h3>

                <label>
                    <input type="checkbox" checked />
                    Substitution
                </label>

                <label>
                    <input type="checkbox" checked />
                    Insertion
                </label>

                <label>
                    <input type="checkbox" checked />
                    Deletion
                </label>
            </div>
        </div>
    {/if}
</div>

<style>
    /* filter panel */
    .filter-controls {
        position: relative;
        margin-top: 2rem;
        margin-bottom: 1rem;
    }

    .filter-button {
        padding: 0.35rem 0.7rem;
        border: 1px solid #ccc;
        border-radius: 3px;
        background: white;
        font-size: 0.85rem;
        cursor: pointer;
    }

    .filter-button:hover {
        background: #f5f5f5;
    }

    .filter-panel {
        position: absolute;
        top: calc(100% + 0.5rem);
        left: 0;
        z-index: 10;

        display: grid;
        grid-template-columns: repeat(3, minmax(130px, 1fr));
        gap: 2rem;

        min-width: 450px;
        padding: 1rem 1.25rem;

        background: white;
        border: 1px solid #ddd;
        border-radius: 4px;
    }

    .filter-column {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
    }

    .filter-column h3 {
        margin: 0 0 0.25rem;
        font-size: 0.8rem;
        font-weight: 600;
    }

    .filter-column label {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        font-size: 0.85rem;
        cursor: pointer;
    }
</style>