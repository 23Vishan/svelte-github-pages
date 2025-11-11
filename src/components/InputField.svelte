<script>
  export let Label = "";
  export let Value = "";

  switch (Label) {
    case "Entry Time":
      Value = "10:00";
      break;
    case "Spread Width":
      Value = "30";
      break;
    case "Entry Credit":
      Value = "1.5";
        break;
    case "Number of Spreads":
        Value = "3";
        break;  
    case "Stop Price":
        Value = "1.1";
        break;
    case "Limit Price":
        Value = "1";
        break;
    case "Stop Loss Multiplier":
        Value = "2";
        break;
    default:
      Value = "";
      break;
  }
</script>

<div class="input-field-container">
    <p class="input-field-label">{Label}</p>

    {#if Label === "Entry Time"}
        <input type="text" name="input-{Label}" required maxLength="5" bind:value={Value}>
    {:else if Label === "Spread Width"}
        <input type="text" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Entry Credit"}
        <input type="text" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Number of Spreads"}
        <input type="text" name="input-{Label}" required maxlength="1" bind:value={Value}>
    {:else if Label === "Stop Price"}
        <input type="text" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Limit Price"}
        <input type="text" name="input-{Label}" required maxlength="3" bind:value={Value}>
    {:else if Label === "Stop Loss Multiplier"}
        <input type="text" name="input-{Label}" required maxlength="3" bind:value={Value}>
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
        background-color: #242424;
        border: none;
        text-align: right;
        padding-right: 0.75rem;
    }

    input:hover {
        outline: 2px solid #4e4e4e;
    }

    input:focus {
        outline: 2px solid #888888;
    }
</style>