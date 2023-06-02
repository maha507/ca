<script>
    import { onMount } from "svelte";
    import { createEventDispatcher } from "svelte";
    import 'bootstrap/dist/css/bootstrap.min.css';

    let jsonData = [];
    let tableVisible = false;
    let selectedCandidate = null;
    let uploadCandidateId = null;
	let deletePopupVisible = false;

    let isPopupVisible = false;
    let file = null;
	let editing = false;
    const dispatch = createEventDispatcher();

    onMount(async () => {
      await fetchData();
      tableVisible = true;
    });

    async function fetchData() {
      const response = await fetch("https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E");
      const responseData = await response.json();
      jsonData = responseData.data;
    }

    function handleTitleClick(candidate) {
  if (candidate) {
    selectedCandidate = candidate;
    editing = true;
  } else {
    selectedCandidate = {
      firstName: "",
      surname: "",
      email: "",
      mobile: ""
    };
    editing = false;
  }
  dispatch("showCandidatePopup");
}

    function handleUploadCV(candidateId) {
      uploadCandidateId = candidateId;
      isPopupVisible = true;
    }

    function handleFileUpload(event) {
      file = event.target.files[0];
      // Perform further actions with the uploaded file

      // Example: Update the backend API URL with the file upload
      const formData = new FormData();
      formData.append("file", file);

      fetch(`https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${uploadCandidateId}`, {
        method: "POST",
        body: formData
      })
        .then(response => {
          // Handle the response accordingly
          if (response.ok) {
            console.log("CV uploaded successfully!");
          } else {
            console.error("CV upload failed.");
          }
        })
        .catch(error => {
          console.error("CV upload error:", error);
        });
    }

    function handleSave() {
      // Perform save logic
      // In this case, we're updating the backend API URL in the handleFileUpload function
      isPopupVisible = false;
    }

    function handleClose() {
      // Perform close logic
      isPopupVisible = false;
    }
	async function handleSaveClick(event) {
		event.preventDefault(); 
  if (editing) {
    // Send the updated candidate data to the API
    const apiUrl = `https://api.recruitly.io/api/candidate/?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
    const response = await fetch(apiUrl, {
      method: 'POSt',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(selectedCandidate)
    });

    if (response.ok) {
      // Update the candidate in the jsonData array with the edited values
      const index = jsonData.findIndex(candidate => candidate.id === selectedCandidate.id);
      if (index !== -1) {
        jsonData[index] = { ...selectedCandidate };
      }
    }
  } else {
    // Send the new candidate data to the API
    const apiUrl = `https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(selectedCandidate)
    });

    if (response.ok) {
      // Fetch the updated data to reflect the new candidate in the table
      await fetchData();
    }
  }

  // Reset selectedCandidate and editing to their initial values
  selectedCandidate = null;
  editing = false;
 
}
function handleDeleteClick(candidate) {
  const confirmed = confirm("Are you sure you want to delete this candidate?");
  if (confirmed) {
    deleteCandidate(candidate);
	
  }
}

async function deleteCandidate(candidate) {
  const apiUrl = `https://api.recruitly.io/api/candidate/${candidate.id}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
  const response = await fetch(apiUrl, {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json'
    }
  });

  if (response.ok) {
    // Remove the deleted candidate from the jsonData array
    jsonData = jsonData.filter(c => c.id !== candidate.id);
    selectedCandidate = null;
  }
}
</script>

<main class="container mt-4">
    {#if tableVisible}
    <table class="table">
        <thead class="thead-light">
			<tr>
				<th colspan="4" style="text-align: right;">
				<button class="btn btn-primary" on:click={() => handleTitleClick(null)}>Add Candidate</button>
			  </th>
			</tr>
            <tr>
                <th>First Name</th>
                <th>Surname</th>
                <th>Email</th>
                <th>Mobile</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {#each jsonData as candidate}
            <tr>
                <td><a href="#" on:click|preventDefault={() => handleTitleClick(candidate)}>{candidate.firstName}</a></td>
                <td>{candidate.surname}</td>
                <td>{candidate.email}</td>
                <td>{candidate.mobile}</td>
                <td>
                    <button on:click={() => handleUploadCV(candidate.id)} class="btn btn-primary">Upload CV</button>
					<button class="btn btn-danger btn-sm" on:click|preventDefault={() => handleDeleteClick(candidate)}>Delete</button>

                </td>
            </tr>
            {/each}
        </tbody>
    </table>
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
    {/if}
</main>
{#if selectedCandidate}
	  <div class="popup">
		<div class="popup-content">
		  <h2>{editing ? 'Edit Candidate' : 'Add Candidate'}</h2>
		  <form on:submit|preventDefault={handleSaveClick}>
			<label for="firstName">First Name:</label>
			<input type="text" id="firstName" bind:value={selectedCandidate.firstName} on:input={(e) => handleFieldChange(e, 'firstName')} />
			<label for="surname">Surname:</label>
			<input type="text" id="surname" bind:value={selectedCandidate.surname} on:input={(e) => handleFieldChange(e, 'surname')} />
			<label for="email">Email:</label>
			<input type="text" id="email" bind:value={selectedCandidate.email} on:input={(e) => handleFieldChange(e, 'email')} />
			<label for="mobile">Mobile:</label>
			<input type="text" id="mobile" bind:value={selectedCandidate.mobile} on:input={(e) => handleFieldChange(e, 'mobile')} />
			<button class="btn btn-primary" type="submit" on:click={handleSaveClick}>Save</button>
			<button class="btn btn-secondary" on:click={() => (selectedCandidate = null)}>Close</button>
		  </form>
		</div>
	  </div>
	{/if}
	{#if deletePopupVisible}
  <div class="popup">
    <div class="popup-content">
      <h4>Confirm Delete</h4>
      <p>Are you sure you want to delete this candidate?</p>
      <div class="button-group">
        <button on:click={handleDelete} class="btn btn-danger">Delete</button>
        <button on:click={handleCancelDelete} class="btn btn-secondary">Cancel</button>
      </div>
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
	  