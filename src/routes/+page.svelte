<script lang="ts">
    let fastaText = $state("");
    function analyze() {
        console.log(fastaText);
    }
</script>
<a href="/select">Go to selection page (will be next in the flow)</a>
<h1>HIV Visualizer</h1>
<textarea
    bind:value={fastaText}
    placeholder={">HXB2\nMRVKE---KYQHL...\n>Sequence_1\n..."}
></textarea>

<button disabled={!fastaText.trim()} onclick={analyze}>
    Analyze Alignment
</button>
