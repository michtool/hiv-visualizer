<script lang="ts">

    import FastaInput from "$lib/components/input/FastaInput.svelte";
    import MetadataReview from "$lib/components/input/MetadataReview.svelte";

    type Stage = "input" | "metadata";

    type MetadataConfiguration = {
        delimiter: string | null;
        fields: {
            index: number;
            type: "subject" | "timepoint" | "sequence_id" | "ignore";
        }[];
    };

    let stage = $state<Stage>("input");
    let fastaText = $state("");

    function handleAnalyze(text: string) {
        fastaText = text;
        stage = "metadata";
    }

    function handleBack() {
        stage = "input";
    }

    function handleConfirm(
        configuration: MetadataConfiguration
    ) {
        console.log({
            fastaText,
            configuration
        });

        // TODO:
        // Send the complete FASTA + configuration to the backend.
        // The backend should return the canonical dataset JSON
        // used throughout EnvIsion.

        window.location.href = "/analysis";
    }
</script>
{#if stage === "input"}

    <FastaInput
        value={fastaText}
        onAnalyze={handleAnalyze}
    />

{:else}

    <MetadataReview
        {fastaText}
        onBack={handleBack}
        onConfirm={handleConfirm}
    />

{/if}