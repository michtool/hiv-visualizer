<script lang="ts">

    import mutations from "../data/mutations.json";

    const regions = [
        { name: "V1", start: 131, end: 157 },
        { name: "V2", start: 158, end: 196 },
        { name: "V3", start: 296, end: 331 },
        { name: "V4", start: 385, end: 418 },
        { name: "V5", start: 460, end: 469 }
    ];
    const svgStart = 25;
    const svgEnd = 975;

    const hxb2Start = 1;
    const hxb2End = 856;

    let selectedMutation = $state<number | null>(null);
    let hoveredMutation = $state<number | null>(null);
    let tooltipX = $state(0);
    let tooltipY = $state(0);
    
    const referenceWeek = 4;
    const comparisonWeek = 53;


    const substitutions = mutations.filter(
        (mutation) => mutation.type === "substitution"
    ).length;

    const insertions = mutations.filter(
        (mutation) => mutation.type === "insertion"
    ).length;

    const deletions = mutations.filter(
        (mutation) => mutation.type === "deletion"
    ).length;

    const totalChanges = mutations.length;

    function mapPosition(position: number) {
        return (
            svgStart +
            ((position - hxb2Start) / (hxb2End - hxb2Start)) *
            (svgEnd - svgStart)
        );
    }
    function tooltipPosition(position: number) {
        return `${(mapPosition(position) / 1000) * 100}%`;
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
                            {region.name}
                        </text>

                        <text
                            x={mapPosition(region.start)}
                            y="75"
                            text-anchor="middle"
                        >
                            {region.start}
                        </text>

                        <text
                            x={mapPosition(region.end)}
                            y="75"
                            text-anchor="middle"
                        >
                            {region.end}
                        </text>
                    {/each}

                    {#each mutations as mutation}
                        <line class="mutation-mark"
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
        background-color: #f0f0f0; /* Crucial: Prevents body data from showing underneath */
        z-index: 2;
        
    }


    tr.selected {
        background: #eaeeff;
    }

    tr:hover:not(.selected) {
        background: #f0f0f0;
    }

    /* svg */
    .visualization-container {
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
        stroke-width: 3px;
        stroke: black;
    }

    .mutation-mark.selected {
        stroke:yellow;
    }
    .mutation-mark:hover {
        stroke:darkgray;
    }
</style>