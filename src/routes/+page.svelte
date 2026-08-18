<script lang="ts">

    import mutations from "../data/mutations.json";
    import MutationTable from "$lib/components/MutationTable.svelte";
    import MutationFilter from "$lib/components/MutationFilter.svelte";
    import EnvVisualization from "$lib/components/EnvVisualization.svelte";

    let selectedMutation = $state<number | null>(null);
    
    // to delete
    const referenceWeek = 4;
    const comparisonWeek = 53;
    const totalChanges = mutations.length;


    const substitutions = mutations.filter(
        (mutation) => mutation.type === "substitution"
    ).length;

    const insertions = mutations.filter(
        (mutation) => mutation.type === "insertion"
    ).length;

    const deletions = mutations.filter(
        (mutation) => mutation.type === "deletion"
    ).length;
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


    <MutationFilter />

    <div class="analysis-grid">

        <!-- where the table used to be -->
        <MutationTable
            {mutations}
            {selectedMutation}
            onSelect={(position: number) => selectedMutation = position}
        />

        <!-- where the svg used to be -->
        <EnvVisualization
            {mutations}
            {selectedMutation}
            onSelect={(position: number) => selectedMutation = position}
        />
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

    /* top two cols */
    .analysis-grid {
        display: grid;
        grid-template-columns: 400px minmax(0, 1fr);
        gap: 2rem;
        align-items: start;
    }
</style>