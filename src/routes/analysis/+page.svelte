<script lang="ts">

    import mutations from "../../data/mutations.json";
    import MutationTable from "$lib/components/MutationTable.svelte";
    import MutationFilter from "$lib/components/MutationFilter.svelte";
    import EnvVisualization from "$lib/components/EnvVisualization.svelte";
    import AlignmentViewer from "$lib/components/AlignmentViewer.svelte";

    import { page } from '$app/state';

    type Mutation = {
        alignment_position: number;
        hxb2_position: number;
        sequence_a: string;
        sequence_b: string;
        notation: string;
        type: string;
        region: string;
        hypervariable: boolean;
    };

    import parsed from "../../data/parsed.json";

    const hxb2Name = parsed[0]?.name;
    const referenceName = $derived(page.url.searchParams.get('reference') ?? '');
    const sampleName = $derived(page.url.searchParams.get('sample') ?? '');

    const hxb2Seq = parsed[0]?.sequence;
    const referenceSeq = parsed.find(item => item.name === referenceName)?.sequence;
    const sampleSeq = parsed.find(item => item.name === sampleName)?.sequence;

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
        <p class="weeks">
            {referenceName} → {sampleName}
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

    <AlignmentViewer
        hxb2={{ name: hxb2Name, sequence: hxb2Seq}}
        reference={{ name: referenceName, sequence: referenceSeq }}
        sample={{ name: sampleName, sequence: sampleSeq }}
    />

    <MutationFilter />

    <div class="analysis-grid">

        <MutationTable
            {mutations}
            {selectedMutation}
            onSelect={(position: number) => selectedMutation = position}
        />

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
        grid-template-columns: 430px minmax(0, 1fr);
        gap: 2rem;
        align-items: start;
    }
</style>