<script lang="ts">
    let { mutations, selectedMutation, onSelect } = $props();

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

    // for sliders
    let zoom = $state(1);
    let tickWidth = $state(2);

    // svg drawing
    const svgStart = 25;
    const svgEnd = 975;

    const hxb2Start = 1;
    const hxb2End = 856;

    // tooltip
    let hoveredMutation = $state<number | null>(null);
    let tooltipX = $state(0);
    let tooltipY = $state(0);

    // color coding
    let colorBy = $state("none");
    let distinguishHypervariable = $state(false);

    function mapPosition(position: number) {
        return (
            svgStart +
            ((position - hxb2Start) / (hxb2End - hxb2Start)) *
            (svgEnd - svgStart) *
            zoom
        );
    }

    function getMutationClass(
        mutation: { type: any; region: any; hypervariable: any; }
    ) {
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
                        (mutation: { region: string; hypervariable: any; }) =>
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

<section class="visualization">
    <h2>Env</h2>
    <label for="zoom">Zoom</label>

    <input
        id="zoom"
        type="range"
        min="0.75"
        max="8"
        step="0.25"
        bind:value={zoom}
    />

    <span>{zoom}×</span>

    <label for="tick-width">Tick width</label>

    <input
        id="tick-width"
        type="range"
        min="1"
        max="6"
        step="0.5"
        bind:value={tickWidth}
    />

    <span>{tickWidth}px</span>


    <div class="visualization-container">
        <svg viewBox={`0 0 ${1000 * zoom} 150`} aria-label="HIV Env sequence map" style:width={`${1000 * zoom}px`}>
        <line
            x1={svgStart}
            y1="100"
            x2={svgEnd * zoom}
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
                    y1={selectedMutation === mutation.hxb2_position ? 86 : 92}
                    x2={mapPosition(mutation.hxb2_position)}
                    y2={selectedMutation === mutation.hxb2_position ? 114 : 108}
                    class:selected={selectedMutation === mutation.hxb2_position}
                    stroke-width={tickWidth}
                    role="button"
                    tabindex="0"
                    onclick={() => onSelect(mutation.hxb2_position)}
                    onmouseleave={() => hoveredMutation = null}
                    onkeydown={(event) => {
                        if (event.key === "Enter" || event.key === " " || event.key === "Return") {
                            onSelect(mutation.hxb2_position);
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

<style>
    /* visualization */
    .visualization-container { /* note that this is just the svg container that scrolls */
        width: 100%;
        overflow-x: scroll;
        overflow-y: hidden;
        position: relative;

    }

    /* zoom */
    .zoom-control {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        margin-bottom: 0.75rem;
        font-size: 0.8rem;
        cursor: pointer;
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
        stroke: black;
    }

    .mutation-mark:focus {
        outline: none;
    }

    .mutation-mark:hover {
        filter: brightness(0.75);
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