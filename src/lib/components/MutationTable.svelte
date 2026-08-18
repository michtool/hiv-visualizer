<script lang="ts">
    let { mutations, selectedMutation, onSelect } = $props();
</script>

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
                    onclick={() => onSelect(mutation.hxb2_position)}
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
<style>

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
</style>