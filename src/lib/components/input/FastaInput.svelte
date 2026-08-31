<script lang="ts">
    import Logo from "$lib/components/Logo.svelte";

    let {
        value = "",
        onAnalyze
    }: {
        value?: string;
        onAnalyze: (fasta: string) => void;
    } = $props();

    let fastaText = $state(value);

    function parseHeaders(text: string): string[] {
        return text
            .split(/\r?\n/)
            .filter((line) => line.trim().startsWith(">"))
            .map((line) => line.trim().slice(1).trim())
            .filter(Boolean);
    }

    const sequenceCount = $derived(
        parseHeaders(fastaText).length
    );

    function analyze() {
        if (!fastaText.trim()) {
            return;
        }

        onAnalyze(fastaText);
    }

    function clear() {
        fastaText = "";
    }
</script>

<main class="input-page">

    <div class="hero">
        <Logo size="large"/>
        <h1>Explore HIV-1 Env sequences</h1>

        <p class="description">
            Paste an HXB2-aligned amino-acid FASTA alignment to begin. EnvIsion will validate the alignment and identify information contained in your sequence identifiers.
        </p>
    </div>


    <section>
        <div class="section-header">
            <div>
                <h2>Sequence alignment</h2>
                <p>
                    Input must be HXB2 aligned, and only include the Env. We recommend you use LANL's <a href="https://www.hiv.lanl.gov/content/sequence/GENE_CUTTER/cutter.html" target="_blank">Gene Cutter</a> tool to align and extract Env from your alignment. Include HXB2 as the first sequence.
                </p>
            </div>
        </div>

        <textarea
            bind:value={fastaText}
            spellcheck="false"
            placeholder="Paste your HXB2-aligned amino-acid FASTA here..."
        ></textarea>

        <div class="input-footer">
            <span>
                {sequenceCount} sequences
            </span>

            <div class="actions">
                <button
                        class="secondary-button"
                        disabled={!fastaText.trim()}
                        onclick={clear}
                >
                    Clear
                </button>

                <button
                        class="primary-button"
                        disabled={!fastaText.trim()}
                        onclick={analyze}
                >
                    Next
                </button>
            </div>
        </div>

    </section>

</main>

<style>
    @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:ital,wght@0,100..700;1,100..700&display=swap');
    .input-page {

        max-width: 1000px;
        margin: 0 auto;
        padding: 5rem 2rem;
        color: #171717;
        font-family: "IBM Plex Sans", sans-serif;
    }

    .hero {
        max-width: 720px;
        margin-bottom: 2.5rem;
    }

    h1 {
        margin: 0;
        font-size: 1.6rem;
        line-height: 2;
        letter-spacing: -0.04em;
    }

    .description {
        margin: 1rem 0 0;
        color: #666;
        font-size: 1.05rem;
        line-height: 1.6;
    }



    .section-header {
        margin-bottom: 1rem;
    }

    h2 {
        margin: 0;
        font-size: 1.1rem;
    }

    .section-header p {
        margin: 0.3rem 0 0;
        color: #777;
        font-size: 0.85rem;
    }

    textarea {
        display: block;
        box-sizing: border-box;
        width: 100%;
        min-height: 250px;
        padding: 1rem;
        resize: vertical;
        border: 1px solid #d5d5d5;
        border-radius: 8px;
        outline: none;
        background: #fcfcfc;
        font-family: monospace;
        font-size: 0.85rem;
        line-height: 1.5;
    }

    textarea:focus {
        border-color: #888;
    }

    .input-footer {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-top: 1rem;
    }

    .input-footer span {
        color: #777;
        font-size: 0.85rem;
    }

    .primary-button {
        border: 1px solid #171717;
        border-radius: 7px;
        padding: 0.65rem 1rem;
        background: rgb(39, 68, 130);
        color: white;
        font: inherit;
        font-weight: 600;
        cursor: pointer;
    }

    .primary-button:hover:not(:disabled) {
        background: rgb(31, 55, 105);
    }

    .primary-button:disabled {
        opacity: 0.4;
        cursor: not-allowed;
    }

    .actions {
        display: flex;
        gap: 0.5rem;
    }

    .secondary-button {
        border: 1px solid #d5d5d5;
        border-radius: 7px;
        padding: 0.65rem 1rem;
        background: white;
        color: #333;
        font: inherit;
        font-weight: 600;
        cursor: pointer;
    }

    .secondary-button:hover:not(:disabled) {
        background: #f5f5f5;
        border-color: #bbb;
    }

    .secondary-button:disabled {
        opacity: 0.4;
        cursor: not-allowed;
    }

    .requirements strong {
        color: #444;
    }

    @media (max-width: 700px) {
        .input-page {
            padding: 3rem 1rem;
        }

        h1 {
            font-size: 2rem;
        }

        .input-footer {
            align-items: flex-start;
            gap: 1rem;
            flex-direction: column;
        }
    }
</style>