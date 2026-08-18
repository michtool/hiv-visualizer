<script lang="ts">

    import mutations from "../data/mutations.json";

    const regions = [
        { name: "V1", start: 131, end: 157 },
        { name: "V2", start: 158, end: 196 },
        { name: "V3", start: 296, end: 331 },
        { name: "V4", start: 385, end: 418 },
        { name: "V5", start: 460, end: 469 }
    ];
    const mutationTypes = [
        { value: "substitution", label: "Substitution" },
        { value: "insertion", label: "Insertion" },
        { value: "deletion", label: "Deletion" }
    ];

    const regionColors = [
        { value: "V1", label: "V1" },
        { value: "V2", label: "V2" },
        { value: "V3", label: "V3" },
        { value: "V4", label: "V4" },
        { value: "V5", label: "V5" }
    ];

    // filter
    let filterOpen = $state(false);
    let showSubstitutions = $state(true);
    let showInsertions = $state(true);
    let showDeletions = $state(true);

    // svg
    const svgStart = 25;
    const svgEnd = 975;

    const hxb2Start = 1;
    const hxb2End = 856;

    let selectedMutation = $state<number | null>(null);
    let hoveredMutation = $state<number | null>(null);
    let tooltipX = $state(0);
    let tooltipY = $state(0);
    
    // to delete
    const referenceWeek = 4;
    const comparisonWeek = 53;
    const totalChanges = mutations.length;

    // color coding
    let colorBy = $state("none");
    let distinguishHypervariable = $state(false);

    const substitutions = mutations.filter(
        (mutation) => mutation.type === "substitution"
    ).length;

    const insertions = mutations.filter(
        (mutation) => mutation.type === "insertion"
    ).length;

    const deletions = mutations.filter(
        (mutation) => mutation.type === "deletion"
    ).length;


    function mapPosition(position: number) {
        return (
            svgStart +
            ((position - hxb2Start) / (hxb2End - hxb2Start)) *
            (svgEnd - svgStart)
        );
    }

    // for color coding
    function getMutationClass(mutation: { type: any; region: any; hypervariable: any; }) {
        if (colorBy === "type") {
            return mutation.type;
        }

        if (colorBy === "region") {
            if (!mutation.region) return "none";

            if (distinguishHypervariable && mutation.hypervariable) {
                return `${mutation.region}-hypervariable`;
            }

            return mutation.region;
        }

        return "none";
    }

    // for color coding legend
    function getLegendItems() {
        if (colorBy === "type") {
            return mutationTypes;
        }

        if (colorBy === "region") {
            const items = [];

            for (const region of ["V1", "V2", "V3", "V4", "V5"]) {
                items.push({
                    label: region,
                    value: region
                });

                if (
                    distinguishHypervariable &&
                    mutations.some(
                        (mutation) =>
                            mutation.region === region &&
                            mutation.hypervariable
                    )
                ) {
                    items.push({
                        label: `${region} HVR`,
                        value: `${region}-hypervariable`
                    });
                }
            }

            return items;
        }

        return [];
    }
</script>

<main>
    <header>
        <p class="eyebrow">HIV Env Evolution</p>

        <h1>CH505</h1>

        <p class="weeks">
            Week {referenceWeek} → Week {comparisonWeek}
        </p>
    </header>

    <section class="summary">
        <h2>{totalChanges} changes</h2>

        <p>
            {substitutions} substitutions ·
            {insertions} insertions ·
            {deletions} deletions
        </p>
    </section>

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

    <div class="analysis-grid">

        <section class="mutations">
            <h2>Mutations</h2>
            <div class="table-container">
                <table>
                    <thead>
                        <tr>
                            <th>Name</th>
                            <th>HXB2</th>
                            <th>Change</th>
                            <th>Region</th>
                            <th>Type</th>
                        </tr>
                    </thead>
                    
                    <tbody>
                        {#each mutations as mutation}
                        <tr
                            class:selected={selectedMutation === mutation.hxb2_position}
                            onclick={() => selectedMutation = mutation.hxb2_position}
                        >       <td>{mutation.notation}</td>
                                <td>{mutation.hxb2_position}</td>
                                <td>{mutation.sequence_a} → {mutation.sequence_b}</td>
                                <td>
                                    {mutation.region}
                                    {#if mutation.hypervariable}
                                        Hypervariable
                                    {/if}
                                </td>
                                <td>{mutation.type}</td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            </div>

        </section>

        <section class="visualization">
            <h2>Env</h2>
            <div class="visualization-container">
                <svg viewBox="0 0 1000 150" aria-label="HIV Env sequence map">
                    <line
                        x1={svgStart}
                        y1="100"
                        x2={svgEnd}
                        y2="100"
                        stroke="black"
                        stroke-width="3"
                    />
                    {#each regions as region}
                        <line
                            x1={mapPosition(region.start)}
                            y1="55"
                            x2={mapPosition(region.end)}
                            y2="55"
                            stroke="black"
                            stroke-width="3"
                        />

                        <text
                            x={mapPosition((region.start + region.end) / 2)}
                            y="45"
                            text-anchor="middle"
                        >
                            <title>HXB2 {region.start} - {region.end}</title> <!-- replace this with a good looking tooltip -->
                            {region.name}
                        </text>
                    {/each}

                    {#each mutations as mutation}
                        <line class={`mutation-mark ${getMutationClass(mutation)}`}
                            x1={mapPosition(mutation.hxb2_position)}
                            y1="92"
                            x2={mapPosition(mutation.hxb2_position)}
                            y2="108"
                            class:selected={selectedMutation === mutation.hxb2_position}
                            role="button"
                            tabindex="0"
                            onclick={() => selectedMutation = mutation.hxb2_position}
                            onmouseleave={() => hoveredMutation = null}
                            onkeydown={(event) => {
                                if (event.key === "Enter" || event.key === " " || event.key === "Return") {
                                    selectedMutation = mutation.hxb2_position;
                                }
                            }}
                            onmouseenter={(event) => {
                                hoveredMutation = mutation.hxb2_position;
                                tooltipX = event.clientX;
                                tooltipY = event.clientY;
                            }}
                        />

                    {/each}
                    
                </svg>
                {#if hoveredMutation !== null}
                    {#each mutations as mutation}
                        {#if hoveredMutation === mutation.hxb2_position}
                            <div
                                class="mutation-tooltip"
                                style:left={`${tooltipX - 50}px`}
                                style:top={`${tooltipY + 20}px`}
                            >
                                <strong>{mutation.notation}</strong>
                                <span>HXB2 {mutation.hxb2_position}</span>
                                <span>Alignment {mutation.alignment_position}</span>
                                <span>{mutation.sequence_a} → {mutation.sequence_b}</span>

                                {#if mutation.region}
                                    <span>
                                        {mutation.region}
                                        {#if mutation.hypervariable}
                                            Hypervariable
                                        {/if}
                                    </span>
                                {/if}
                            </div>
                        {/if}
                    {/each}
                {/if}
            </div>
            <label>
                Color by
                <select bind:value={colorBy} class="color-dropdown">
                    <option value="none">None</option>
                    <option value="type">Mutation type</option>
                    <option value="region">Region</option>
                    <option value="amino-acid">Amino acid</option>
                </select>
            </label>
            {#if colorBy === "region"}
                <label>
                    <input type="checkbox" class="hypervariable-checkbox" bind:checked={distinguishHypervariable} />
                    Distinguish hypervariable regions (HVRs)
                </label>
            {/if}
            {#if colorBy !== "none"}
                <div class="legend">
                    {#each getLegendItems() as item}
                        <span class="legend-item">
                            <span class={`legend-dot ${item.value}`}></span>
                            {item.label}
                        </span>
                    {/each}
                </div>
            {/if}
        </section>
    </div>



    

</main>

<style>
    main {
        max-width: 1100px;
        margin: 0 auto;
        padding: 4rem 2rem;
        font-family: system-ui, sans-serif;
    }

    header {
        margin-bottom: 2rem;
    }

    .eyebrow {
        font-size: 0.85rem;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        margin-bottom: 0.5rem;
    }

    h1 {
        font-size: 3rem;
        margin: 0;
    }

    .weeks {
        font-size: 1rem;
    }

    h2 {
        margin: 0 0 0.5rem;
        font-size: 2rem;
    }
    .mutations {
        margin-top: 3rem;
    }

    /* top two cols */
    .analysis-grid {
        display: grid;
        grid-template-columns: 400px minmax(0, 1fr);
        gap: 2rem;
        align-items: start;
    }

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

    /* table */

    .table-container {
        width: 100%;
        max-height: 500px;
        background-color: #fcfcfc;
        overflow-y: auto;
    }

    table {
        width: 100%;
        border-collapse: collapse;
    }

    th,
    td {
        padding: 0.4rem;
        text-align: left;
        border-bottom: 1px solid #ddd;
        font-size: 0.85rem;
        cursor: pointer;
    }

    th {
        font-weight: 600;
        position: sticky;
        top: 0;
        background-color: #f0f0f0;
        z-index: 2;
        
    }


    tr.selected {
        background: #eaeeff;
    }

    tr:hover:not(.selected) {
        background: #f0f0f0;
    }

    /* visualization */
    .visualization-container { /* note that this is just the svg container that scrolls */
        width: 100%;
        overflow-x: scroll;
        overflow-y: hidden;
        position: relative;

    }

    .mutation-tooltip {
        position: fixed;
        top: 10px;
        left: 50%;

        display: flex;
        flex-direction: column;
        gap: 0.15rem;

        padding: 0.6rem 0.75rem;

        background: white;
        border: 1px solid #ddd;
        border-radius: 6px;

        font-size: 0.8rem;
        white-space: nowrap;


        pointer-events: none;
    }

    .visualization-container svg {
        width: 1000px;
        height: 150px;
        display: block;
        background-color: #fcfcfc;
    }

    .mutation-mark {
        cursor: pointer;
        stroke-width: 2px;
        stroke: black;
    }

    .mutation-mark.selected {
        stroke-width: 4px; /* not sure if thickness change makes sense here */
    }

    .mutation-mark:hover {
        stroke:darkgray;
    }

    /* mutation mark color coding */
    .visualization {
        --substitution: #2563eb;
        --insertion: #16a34a;
        --deletion: #dc2626;

        --v1: #0072B2;
        --v2: #E69F00;
        --v3: #009E73;
        --v4: #CC79A7;
        --v5: #D55E00;
    }

    .color-dropdown, .hypervariable-checkbox {
        cursor: pointer;
    }

    .legend {
        display: flex;
        align-items: center;
        gap: 1rem;
        margin-top: 0.75rem;
        font-size: 0.8rem;
    }

    .legend-item {
        display: flex;
        align-items: center;
        gap: 0.35rem;
    }

    .legend-dot {
        width: 8px;
        height: 8px;
        border-radius: 50%;
        display: inline-block;
    }

    /* type color coding */
    .mutation-mark.substitution {
        stroke: var(--substitution);
    }

    .mutation-mark.insertion {
        stroke: var(--insertion);
    }

    .mutation-mark.deletion {
        stroke: var(--deletion);
    }

    .legend-dot.substitution {
        background: var(--substitution);
    }

    .legend-dot.insertion {
        background: var(--insertion);
    }

    .legend-dot.deletion {
        background: var(--deletion);
    }

    /* regions */
    .mutation-mark.V1 {
        stroke: var(--v1);
    }

    .mutation-mark.V2 {
        stroke: var(--v2);
    }

    .mutation-mark.V3 {
        stroke: var(--v3);
    }

    .mutation-mark.V4 {
        stroke: var(--v4);
    }

    .mutation-mark.V5 {
        stroke: var(--v5);
    }
    
    .mutation-mark.V1-hypervariable {
        stroke: color-mix(in srgb, var(--v1), white 55%);
    }

    .mutation-mark.V2-hypervariable {
        stroke: color-mix(in srgb, var(--v2), white 55%);
    }

    .mutation-mark.V3-hypervariable {
        stroke: color-mix(in srgb, var(--v3), white 55%);
    }

    .mutation-mark.V4-hypervariable {
        stroke: color-mix(in srgb, var(--v4), white 55%);
    }

    .mutation-mark.V5-hypervariable {
        stroke: color-mix(in srgb, var(--v5), white 55%);
    }

    .legend-dot.V1 {
        background: var(--v1);
    }

    .legend-dot.V2 {
        background: var(--v2);
    }

    .legend-dot.V3 {
        background: var(--v3);
    }

    .legend-dot.V4 {
        background: var(--v4);
    }

    .legend-dot.V5 {
        background: var(--v5);
    }
    .legend-dot.V1-hypervariable {
        background: color-mix(in srgb, var(--v1), white 55%);
    }

    .legend-dot.V2-hypervariable {
        background: color-mix(in srgb, var(--v2), white 55%);
    }

    .legend-dot.V3-hypervariable {
        background: color-mix(in srgb, var(--v3), white 55%);
    }

    .legend-dot.V4-hypervariable {
        background: color-mix(in srgb, var(--v4), white 55%);
    }

    .legend-dot.V5-hypervariable {
        background: color-mix(in srgb, var(--v5), white 55%);
    }

</style>