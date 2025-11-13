<script>
  export let Label = "";
  export let Value = "";
  export let disabled = false;
  export let redText = false;
  export let greenText = false;
  export let whiteText = false;
  
  $: {
      if (Label === "Total Profit") {
        const totalProfit = parseFloat(Value);
        greenText = totalProfit > 0;
        redText = totalProfit < 0;
        whiteText = totalProfit === 0;
    }
  }
</script>

<div class="input-field-container">
    <p class="input-field-label">{Label}</p>

    {#if Label === "Entry Time"}
        <input type="text" class="input" name="input-{Label}" required maxLength="5" bind:value={Value}>
    {:else if Label === "Spread Width"}
        <input type="text" class="input" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Entry Credit"}
        <input type="text" class="input" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Number of Spreads"}
        <input type="text" class="input" name="input-{Label}" required maxlength="1" bind:value={Value}>
    {:else if Label === "Stop Price"}
        <input type="text" class="input" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Limit Price"}
        <input type="text" class="input" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Stop Loss Multiplier"}
        <input type="text" class="input" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else}
        <input type="text" name="input-{Label}" bind:value={Value} disabled={disabled} class:greenText={greenText} class:redText={redText} class:whiteText={whiteText}>
    {/if}
</div>

<style>
    .input-field-container {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        gap: 0.25rem;
        width: 100%;
    }

    .input-field-label {
        padding-left: 0.5rem;
    }

    input {
        width: 100%;
        height: 1.75rem;
        background-color: #1E1E1E;
        border: none;
        text-align: right;
        padding-right: 0.75rem;
        border-radius: 5px;
    }
    
    .input:hover {
        outline: 2px solid #4e4e4e;
    }

    .input:focus {
        outline: 2px solid #888888;
    }

    .greenText {
        color: #44AD65;
    }

    .redText {
        color: #F44336;
    }

    .whiteText {
        color: #FFFFFF;
    }
</style>