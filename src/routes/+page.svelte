<script lang="ts">
    const regions = [
        { name: "V1", start: 131, end: 157 },
        { name: "V2", start: 158, end: 196 },
        { name: "V3", start: 296, end: 331 },
        { name: "V4", start: 385, end: 418 },
        { name: "V5", start: 460, end: 469 }
    ];
    const svgStart = 50;
    const svgEnd = 950;

    const hxb2Start = 131;
    const hxb2End = 469;

    let hoveredMutation = $state<number | null>(null);
    let selectedMutation = $state<number | null>(null);
    
    const referenceWeek = 4;
    const comparisonWeek = 53;

    const mutations = [
        {
            alignment_position: 280,
            hxb2_position: 280,
            sequence_a: "N",
            sequence_b: "D",
            notation: "N280D",
            type: "substitution",
            region: "V2",
            hypervariable: false
        },
        {
            alignment_position: 282,
            hxb2_position: 282,
            sequence_a: "V",
            sequence_b: "G",
            notation: "V282G",
            type: "substitution",
            region: "V2",
            hypervariable: true
        },
        {
            alignment_position: 301,
            hxb2_position: 301,
            sequence_a: "N",
            sequence_b: "S",
            notation: "N301S",
            type: "substitution",
            region: "V3",
            hypervariable: false
        }
    ];

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
    <section class="mutations">
        <h2>Mutations</h2>

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

    </section>
    <section class="visualization">
        <h2>Env</h2>
        <svg viewBox="0 0 1000 180" aria-label="HIV Env sequence map">
            <line
                x1="50"
                y1="100"
                x2="950"
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
                <circle
                    cx={mapPosition(mutation.hxb2_position)}
                    cy="100"
                    r="6"
                    class:selected={selectedMutation === mutation.hxb2_position}
                    role="button"
                    tabindex="0"
                    onclick={() => selectedMutation = mutation.hxb2_position}
                    onmouseenter={() => hoveredMutation = mutation.hxb2_position}
                    onmouseleave={() => hoveredMutation = null}   
                    onkeydown={(event) => {
                        if (event.key === "Enter" || event.key === " " || event.key === "Return") {
                            selectedMutation = mutation.hxb2_position;
                        }
                    }}
                />
                {#if hoveredMutation === mutation.hxb2_position}
                    <text
                        x={mapPosition(mutation.hxb2_position)}
                        y="135"
                        text-anchor="middle"
                    >
                        {mutation.notation}
                    </text>
                {/if}
            {/each}
            
        </svg>
    </section>
    

</main>

<style>
    main {
        max-width: 1000px;
        margin: 0 auto;
        padding: 4rem 2rem;
        font-family: system-ui, sans-serif;
    }

    header {
        margin-bottom: 4rem;
    }

    .eyebrow {
        font-size: 0.85rem;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        margin-bottom: 0.5rem;
    }

    h1 {
        font-size: 4rem;
        margin: 0;
    }

    .weeks {
        font-size: 1.25rem;
    }

    h2 {
        margin: 0 0 0.5rem;
        font-size: 2rem;
    }
    .mutations {
    margin-top: 3rem;
    }

    table {
        width: 100%;
        border-collapse: collapse;
    }

    th,
    td {
        padding: 0.75rem;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }

    th {
        font-weight: 600;
    }
    tr {
    cursor: pointer;
    }

    tr.selected {
        background: #f0f0f0;
    }
    circle {
        cursor: pointer;
    }

    circle.selected {
        fill:yellow;
    }
    circle:hover {
        fill:darkgray;
    }
</style>