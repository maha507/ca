<script>
    export let isPopupVisible;
    export let handleFileUpload;
    export let uploadCandidateId;
    export let handleSave;
    export let handleClose;
    
</script>

{#if isPopupVisible && uploadCandidateId !== null}
    <div class="popup">
		<div class="popup-content">
        <h4>Upload CV Upload</h4>
        <input type="file" on:change={handleFileUpload} />
        <button on:click={handleSave} class="btn btn-primary">Save</button>
        <button on:click={handleClose} class="btn btn-secondary">Close</button>
    </div>
</div>
    {/if}
    <style>
		.popup {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .popup-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    max-width: 400px;
    text-align: center;
  }

  .popup-content h2 {
    margin-bottom: 10px;
  }

  .popup-content label {
    display: block;
    margin-bottom: 5px;
    text-align: left;
  }

  .popup-content input[type="text"] {
    width: 100%;
    padding: 5px;
    margin-bottom: 10px;
  }

  .button-group {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }

  .button-group button {
    flex: 1;
    margin: 0 5px;
  }

  input[type="file"] {
    display: block;
    margin-bottom: 10px;
  }

  button[type="submit"] {
    margin-top: 10px;
  }
</style>
	  