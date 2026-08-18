<script lang="ts">
    import parsed from "../../data/parsed.json";
    import { goto } from "$app/navigation";

    let referenceSequence = $state("");
    let sampleSequence = $state("");
    let selectionMethod = $state("dropdown");

    function continueToAnalysis() {
        goto(`/analysis?reference=${encodeURIComponent(referenceSequence)}&sample=${encodeURIComponent(sampleSequence)}`);
    }
</script>
<main>
    <h1>Select Sequences</h1>

    <p>Choose the two sequences you want to compare.</p>

    <div class="selection-method">
        <label>
            <input
                type="radio"
                name="selectionMethod"
                value="dropdown"
                bind:group={selectionMethod}
            />
            Select from list
        </label>

        <label>
            <input
                type="radio"
                name="selectionMethod"
                value="type"
                bind:group={selectionMethod}
            />
            Enter sequence names
        </label>
    </div>

    <div class="selection-controls">
        <label>
            Reference sequence

            {#if selectionMethod === "dropdown"}
                <select bind:value={referenceSequence}>
                    <option value="" disabled>Select a sequence</option>

                    {#each parsed.slice(1) as sequence}
                        <option value={sequence.name}>
                            {sequence.name}
                        </option>
                    {/each}
                </select>
            {:else}
                <input
                    type="text"
                    bind:value={referenceSequence}
                    placeholder="Enter sequence name"
                />
            {/if}
        </label>

        <label>
            Sample sequence

            {#if selectionMethod === "dropdown"}
                <select bind:value={sampleSequence}>
                    <option value="" disabled>Select a sequence</option>

                    {#each parsed.slice(1) as sequence}
                        <option
                            value={sequence.name}
                            disabled={sequence.name === referenceSequence}
                        >
                            {sequence.name}
                        </option>
                    {/each}
                </select>
            {:else}
                <input
                    type="text"
                    bind:value={sampleSequence}
                    placeholder="Enter sequence name"
                />
            {/if}
        </label>
    </div>

    <div class="alignment-container">
        {#each parsed as sequence}
            <div 
                class="sequence-row" 
                class:selected-reference={sequence.name === referenceSequence}
                class:selected-sample={sequence.name === sampleSequence}
            >
                <div class="sequence-name">{sequence.name}</div>
                <div class="sequence">{sequence.sequence}</div>
            </div>
        {/each}
    </div>

    <button
        class="continue-button"
        disabled={!referenceSequence || !sampleSequence}
        onclick={continueToAnalysis}
    >
        Continue
    </button>
</main>
<style>
    main {
        max-width: 1200px;
        margin: 0 auto;
        padding: 40px 24px;
        font-family: sans-serif;
    }
    .selection-method {
        display: flex;
        gap: 1.5rem;
        margin-bottom: 1rem;
    }

    .selection-controls {
        display: flex;
        gap: 2rem;
        margin-bottom: 2rem;
    }

    .selection-controls label {
        display: flex;
        flex-direction: column;
        gap: 0.25rem;
    }

    select,
    input[type="text"] {
        width: 300px;
    }

    .continue-button {
        width: 120px;
        cursor: pointer;
    }

    .alignment-container {
        overflow: auto;
        width: 100%;
        max-width: 1200px;
        max-height: 400px;
        border: 2px solid black;
    }

    .sequence-row {
        display: flex;
        width: max-content;
        white-space: nowrap;
        letter-spacing: 0.4px;
    }
    .sequence-row:nth-child(odd) {
        background: #ffffff;
    }
    .sequence-row:nth-child(even) {
        background: #f5f5f5;
    }

    .sequence-row.selected-reference {
        background: #e8f1ff;
    }

    .sequence-row.selected-sample {
        background: #f0eaff;
    }

    .sequence-name {
        width: 220px;
        flex-shrink: 0;
        position: sticky;
        left: 0;
        background: inherit;
        z-index: 1;
        font-family: monospace;
        border-right: 1px solid #d0d0d0;
        box-shadow: 2px 0 4px rgba(0, 0, 0, 0.08);
        padding: 5px;

    }

    .sequence {
        font-family: monospace;
        padding: 5px;
    }
</style>
