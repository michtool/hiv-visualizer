<script lang="ts">
    import Logo from "$lib/components/Logo.svelte";

    type FieldType =
        | "subject"
        | "timepoint"
        | "sequence_id"
        | "ignore";

    type IdentifierField = {
        index: number;
        value: string;
        type: FieldType;
    };

    type SequenceHeader = {
        name: string;
        fields: string[];
    };

    type MetadataConfiguration = {
        delimiter: string | null;
        fields: {
            index: number;
            type: FieldType;
        }[];
    };

    let {
        fastaText,
        onBack,
        onSkip,
        onConfirm
    }: {
        fastaText: string;
        onBack: () => void;
        onSkip: () => void;
        onConfirm: (
            configuration: MetadataConfiguration
        ) => void;
    } = $props();

    let delimiter = $state("");
    let sequences = $state<SequenceHeader[]>([]);
    let fields = $state<IdentifierField[]>([]);

    const fieldOptions: {
        value: FieldType;
        label: string;
    }[] = [
        {
            value: "subject",
            label: "Subject"
        },
        {
            value: "timepoint",
            label: "Timepoint"
        },
        {
            value: "sequence_id",
            label: "Sequence ID"
        },
        {
            value: "ignore",
            label: "Ignore"
        }
    ];

    function parseHeaders(text: string): string[] {
        return text
            .split(/\r?\n/)
            .filter((line) =>
                line.trim().startsWith(">")
            )
            .map((line) =>
                line.trim().slice(1).trim()
            )
            .filter(Boolean);
    }

    function isHXB2(name: string): boolean {
        return name.trim().toUpperCase() === "HXB2";
    }

    function guessDelimiter(names: string[]): string {
        const candidates = [
            "_",
            "|",
            "/",
            ":",
            ";",
            "-"
        ];

        let bestDelimiter = "";
        let bestScore = 0;

        for (const candidate of candidates) {
            const counts = names.map(
                (name) =>
                    name.split(candidate).length - 1
            );

            const nonZero = counts.filter(
                (count) => count > 0
            );

            if (
                nonZero.length <
                Math.ceil(names.length * 0.5)
            ) {
                continue;
            }

            const frequency: Record<number, number> =
                {};

            for (const count of nonZero) {
                frequency[count] =
                    (frequency[count] ?? 0) + 1;
            }

            const mostCommonCount = Number(
                Object.entries(frequency).sort(
                    (a, b) => b[1] - a[1]
                )[0]?.[0] ?? 0
            );

            const consistency =
                nonZero.filter(
                    (count) =>
                        count === mostCommonCount
                ).length / names.length;

            if (consistency > bestScore) {
                bestScore = consistency;
                bestDelimiter = candidate;
            }
        }

        return bestDelimiter;
    }

    function looksLikeTimepoint(
        value: string
    ): boolean {
        return /^(D|DAY|W|WEEK|M|MONTH)[-_]?\d+$/i.test(
            value.trim()
        );
    }

    function buildSequences(
        names: string[],
        separator: string
    ): SequenceHeader[] {
        return names.map((name) => ({
            name,
            fields: separator
                ? name.split(separator)
                : [name]
        }));
    }

    function buildFields(
        sequenceHeaders: SequenceHeader[],
        separator: string
    ): IdentifierField[] {
        if (sequenceHeaders.length === 0) {
            return [];
        }

        const maxFields = Math.max(
            ...sequenceHeaders.map(
                (sequence) =>
                    sequence.fields.length
            )
        );

        const result: IdentifierField[] =
            Array.from(
                { length: maxFields },
                (_, index) => ({
                    index,
                    value:
                        sequenceHeaders[0]
                            ?.fields[index] ?? "",
                    type: "ignore" as FieldType
                })
            );

        if (!separator || result.length === 1) {
            return result;
        }

        for (
            let index = 0;
            index < result.length;
            index++
        ) {
            const values = sequenceHeaders.map(
                (sequence) =>
                    sequence.fields[index] ?? ""
            );

            const timepointCount =
                values.filter(
                    looksLikeTimepoint
                ).length;

            if (
                timepointCount >=
                Math.ceil(values.length * 0.5)
            ) {
                result[index].type = "timepoint";
            }
        }

        return result;
    }

    function initialize() {
        const names = parseHeaders(fastaText);

        const metadataNames =
            names.filter(
                (name) => !isHXB2(name)
            );

        if (metadataNames.length === 0) {
            sequences = [];
            delimiter = "";
            fields = [];
            return;
        }

        delimiter =
            guessDelimiter(metadataNames);

        sequences = buildSequences(
            metadataNames.slice(0, 20),
            delimiter
        );

        fields = buildFields(
            sequences,
            delimiter
        );
    }

    function refreshMetadataPreview() {
        sequences = sequences.map(
            (sequence) => ({
                ...sequence,
                fields: delimiter
                    ? sequence.name.split(
                        delimiter
                    )
                    : [sequence.name]
            })
        );

        fields = buildFields(
            sequences,
            delimiter
        );
    }

    function handleFieldChange(
        index: number,
        event: Event
    ) {
        const select =
            event.currentTarget as HTMLSelectElement;

        const type =
            select.value as FieldType;

        fields = fields.map(
            (field) => {
                if (
                    field.index === index
                ) {
                    return {
                        ...field,
                        type
                    };
                }

                /*
                 * Only one field can be assigned
                 * to each metadata type.
                 */
                if (
                    type !== "ignore" &&
                    field.type === type
                ) {
                    return {
                        ...field,
                        type: "ignore"
                    };
                }

                return field;
            }
        );
    }

    function confirm() {
        onConfirm({
            delimiter:
                delimiter || null,

            fields: fields.map(
                (field) => ({
                    index: field.index,
                    type: field.type
                })
            )
        });
    }

    initialize();
</script>

<main class="metadata-page">

    <div class="hero">
        <Logo size="small"/>

        <h1>
            Sequence Identifiers
        </h1>

        <p class="description">
            Sequence names may include information such
            as subject, timepoint, or sequence ID.
            EnvIsion can use this information in
            downstream analyses. Timepoint data is
            required for longitudinal analysis.
            This step is optional.
        </p>
    </div>

    {#if sequences.length > 0}

        <!-- delimiter -->

        <div class="delimiter-section">

            <div class="delimiter-heading">

                <h2>
                    {#if delimiter}
                        Using
                        <code>{delimiter}</code>
                        as a delimiter.
                    {:else}
                        Delimiter not identified.
                    {/if}
                </h2>

            </div>

            <div class="delimiter-editor">

                <div class="delimiter-copy">

                    <strong>
                        Not correct?
                    </strong>

                    <p>
                        Change the delimiter EnvIsion
                        should use to split your
                        sequence names.
                    </p>

                </div>

                <div class="delimiter-control">

                    <label for="delimiter">
                        Delimiter
                    </label>

                    <input
                            id="delimiter"
                            bind:value={delimiter}
                            maxlength="10"
                            spellcheck="false"
                            onblur={refreshMetadataPreview}
                            oninput={refreshMetadataPreview}
                    />

                </div>

            </div>

        </div>

        <!-- examples -->

        <div class="examples">

            <div class="subsection-heading">

                <h3>
                    Example names from your alignment
                </h3>

            </div>

            {#each sequences.slice(0, 3) as sequence}

                <code class="example-name">
                    {sequence.name}
                </code>

            {/each}

        </div>

        <!-- field mapping -->

        <div class="field-mapping">

            {#if delimiter}

                <div class="subsection-heading">

                    <div>

                        <h3>
                            Define each field
                        </h3>

                        <p>
                            Select what each field
                            represents. EnvIsion
                            automatically identifies
                            recognizable timepoint
                            fields; other fields are
                            set to Ignore by default.
                        </p>

                    </div>

                </div>

                <div class="mapping-table">

                    <div class="mapping-header">
                        <span>Field</span>
                        <span>Example</span>
                        <span>Meaning</span>
                    </div>

                    {#each fields as field}

                        <div class="mapping-row">

                            <span class="field-number">
                                {field.index + 1}
                            </span>

                            <code>
                                {field.value || "—"}
                            </code>

                            <select
                                    value={field.type}
                                    onchange={(event) =>
                                    handleFieldChange(
                                        field.index,
                                        event
                                    )
                                }
                            >

                                {#each fieldOptions as option}

                                    <option
                                            value={option.value}
                                    >
                                        {option.label}
                                    </option>

                                {/each}

                            </select>

                        </div>

                    {/each}

                </div>

            {/if}

        </div>

        <!-- preview -->

        <div class="preview">

            <div class="subsection-heading">

                <div>

                    <h3>
                        Preview
                    </h3>

                    <p>
                        This is how EnvIsion will
                        interpret your sequence names.
                    </p>

                </div>

            </div>

            {#each sequences.slice(0, 5) as sequence}

                <div class="preview-row">

                    <code class="preview-name">
                        {sequence.name}
                    </code>

                    <div class="metadata-preview">

                        {#each fields as field}

                            {#if field.type !== "ignore"}

                                <span
                                        class="metadata-item"
                                >

                                    <strong>
                                        {#if field.type === "sequence_id"}
                                            Sequence ID
                                        {:else if field.type === "timepoint"}
                                            Timepoint
                                        {:else}
                                            Subject
                                        {/if}
                                    </strong>

                                    <code>
                                        {sequence.fields[
                                            field.index
                                            ] ?? "—"}
                                    </code>

                                </span>

                            {/if}

                        {/each}

                    </div>

                </div>

            {/each}

        </div>

    {:else}

        <!-- no metadata -->

        <div class="no-metadata">

            <div class="info-icon">
                i
            </div>

            <div>

                <h2>
                    No sequence metadata detected
                </h2>

                <p>
                    That's completely fine.
                    EnvIsion will preserve your
                    original sequence names and
                    won't assign subject or
                    timepoint information.
                </p>

            </div>

        </div>

    {/if}

    <!-- actions -->

    <div class="actions">

        <button
                class="skip-button"
                onclick={onSkip}
        >
            Skip
        </button>

        <div class="right-actions">

            <button
                    class="secondary-button"
                    onclick={onBack}
            >
                Back
            </button>

            <button
                    class="primary-button"
                    onclick={confirm}
            >
                Continue
            </button>

        </div>

    </div>

</main>

<style>
    @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:ital,wght@0,100..700;1,100..700&display=swap');

    .metadata-page {
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
        font-size: 2.6rem;
        line-height: 1.1;
    }

    .description {
        margin: 1rem 0 0;
        color: #666;
        font-size: 1.05rem;
        line-height: 1.6;
    }

    .delimiter-section {
        padding-bottom: 1.75rem;
        border-bottom: 1px solid #e5e5e5;
    }

    .delimiter-heading h2 {
        font-size: 1.25rem;
    }

    .delimiter-heading code {
        padding: 0.15rem 0.35rem;
        border-radius: 2px;
        background: #f0f0f0;
        font-family: monospace;
    }

    .delimiter-editor {
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: 2rem;
        margin-top: 1.25rem;
        padding: 1rem;
        border: 1px solid #ddd;
        border-radius: 2px;
        background: #fafafa;
    }

    .delimiter-copy strong {
        font-size: 0.9rem;
    }

    .delimiter-copy p {
        margin: 0.25rem 0 0;
        color: #777;
        font-size: 0.8rem;
    }

    .delimiter-control {
        display: flex;
        align-items: center;
        gap: 0.6rem;
        flex-shrink: 0;
    }

    .delimiter-control label {
        color: #666;
        font-size: 0.8rem;
        font-weight: 600;
    }

    .delimiter-control input {
        box-sizing: border-box;
        width: 70px;
        padding: 0.5rem;
        border: 1px solid #bbb;
        border-radius: 2px;
        outline: none;
        background: white;
        font-family: monospace;
        font-size: 1rem;
        text-align: center;
    }

    .delimiter-control input:focus {
        border-color: #555;
        box-shadow: 0 0 0 2px #eee;
    }

    .examples {
        display: flex;
        flex-direction: column;
        gap: 0.45rem;
        margin-top: 1.5rem;
    }

    .subsection-heading {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        gap: 1rem;
        margin-bottom: 0.75rem;
    }

    .subsection-heading h3 {
        margin: 0;
        font-size: 0.9rem;
    }

    .subsection-heading p {
        margin: 0.25rem 0 0;
        color: #777;
        font-size: 0.8rem;
    }

    .subsection-heading > span {
        color: #888;
        font-size: 0.7rem;
    }

    .example-name {
        font-family: monospace;
        font-size: 0.8rem;
    }

    .field-mapping {
        margin-top: 2rem;
    }

    .mapping-table {
        overflow: hidden;
        border: 1px solid #ddd;
        border-radius: 2px;
    }

    .mapping-header,
    .mapping-row {
        display: grid;
        grid-template-columns: 120px 1fr 180px;
        align-items: center;
        gap: 1rem;
        padding: 0.75rem 1rem;
    }

    .mapping-header {
        background: #f3f3f3;
        color: black;
        font-size: 0.85rem;
        font-weight: 700;
    }

    .mapping-row {
        border-top: 1px solid #e5e5e5;
    }

    .field-number {
        font-size: 0.85rem;
    }

    .mapping-row code {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        font-size: 0.8rem;
    }

    .mapping-row select {
        width: 100%;
        padding: 0.45rem 0.6rem;
        border: 1px solid #d0d0d0;
        border-radius: 2px;
        background: white;
        font: inherit;
    }

    .preview {
        margin-top: 2rem;
    }

    .preview-row {
        padding: 0.8rem 0;
        border-bottom: 1px solid #eee;
    }

    .preview-name {
        display: block;
        margin-bottom: 0.5rem;
        font-size: 0.8rem;
    }

    .metadata-preview {
        display: flex;
        flex-wrap: wrap;
        gap: 0.6rem 1.25rem;
    }

    .metadata-item {
        display: flex;
        gap: 0.35rem;
        color: #666;
        font-size: 0.75rem;
    }

    .metadata-item strong {
        color: #444;
    }

    .no-metadata {
        display: flex;
        gap: 1rem;
        align-items: flex-start;
        padding: 1.25rem;
        border: 1px solid #ddd;
        border-radius: 2px;
        background: #fafafa;
    }

    .no-metadata h2 {
        margin-bottom: 0.35rem;
        font-size: 1rem;
    }

    .no-metadata p {
        margin: 0;
        color: #666;
        font-size: 0.85rem;
        line-height: 1.5;
    }

    .info-icon {
        display: grid;
        width: 24px;
        height: 24px;
        flex: 0 0 auto;
        place-items: center;
        border: 1px solid #aaa;
        border-radius: 50%;
        color: #666;
        font-size: 0.75rem;
        font-weight: 700;
    }

    .actions {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-top: 1rem;
    }

    .right-actions {
        display: flex;
        gap: 0.5rem;
        align-items: center;
    }

    .skip-button,
    .primary-button,
    .secondary-button {
        border-radius: 7px;
        padding: 0.65rem 1rem;
        font: inherit;
        font-weight: 600;
        cursor: pointer;
    }

    .skip-button {
        border: none;
        padding: 0.65rem 0;
        background: transparent;
        font: inherit;
        font-weight: 500;
        cursor: pointer;
    }

    .skip-button:hover {
        color: #333;
        text-decoration: underline;
    }

    .primary-button {
        border: 1px solid #171717;
        background: rgb(39, 68, 130);
        color: white;
    }

    .primary-button:hover {
        background: rgb(31, 55, 105);
    }

    .secondary-button {
        border: 1px solid #d5d5d5;
        background: white;
        color: #333;
    }

    @media (max-width: 700px) {
        .metadata-page {
            padding: 3rem 1rem;
        }

        h1 {
            font-size: 2rem;
        }

        .delimiter-editor {
            flex-direction: column;
            align-items: stretch;
        }

        .delimiter-control {
            justify-content: space-between;
        }

        .mapping-header {
            display: none;
        }

        .mapping-row {
            grid-template-columns: 1fr;
            gap: 0.4rem;
        }
    }
</style>
