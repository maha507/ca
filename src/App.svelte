<script  src="https://cdn.jsdelivr.net/npm/gridjs@1.2.0/dist/gridjs.production.min.js">
	import 'bootstrap/dist/css/bootstrap.min.css';
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
  
	let jsonData = [];
	let tableVisible = false;
	let selectedCandidate = null;
	let editing = false;
	let cvFile = null;
	let cvPopupVisible = false;

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

  
	function handleFieldChange(event, field) {
	  selectedCandidate[field] = event.target.value;
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
function handleCVChange(event) {
  cvFile = event.target.files[0];
}

async function handleCVSubmit() {
    if (cvFile) {
      const formData = new FormData();
      formData.append('cv', cvFile);

      const apiUrl = `https://api.recruitly.io/api/candidate/${selectedCandidate.id}/upload-cv?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
      const response = await fetch(apiUrl, {
        method: 'POST',
        body: formData
      });

      if (response.ok) {
        // CV submitted successfully
        // You can add any additional logic here
        alert('CV submitted successfully!');
      }
    }
  }

  </script>
  
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
  
	.popup-content p {
	  margin-bottom: 5px;
	}
  
	.popup-content input {
	  width: 100%;
	  padding: 5px;
	  margin-bottom: 10px;
	}
  
	.popup-content button {
	  margin-top: 20px;
	}
	input[type="file"] {
  display: block;
  margin-bottom: 10px;
}

button[type="submit"] {
  margin-top: 10px;
}

  </style>
  
  <main>
	<div id="grid"></div>
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
			  <td>
				<a href="#" on:click|preventDefault={() => handleTitleClick(candidate)}>{candidate.firstName}</a>
			  </td>
			  <td>{candidate.surname}</td>
			  <td>{candidate.email}</td>
			  <td>{candidate.mobile}</td>
			  <td>
				<button class="btn btn-primary" on:click={() => (cvPopupVisible = true)}>Submit CV</button>
				<button class="btn btn-danger btn-sm" on:click|preventDefault={() => handleDeleteClick(candidate)}>Delete</button>
			  </td>
			</tr>
		  {/each}
		</tbody>
	  </table>
	{/if}
  
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
  
	{#if cvPopupVisible}
	  <div class="popup">
		<div class="popup-content">
		  <h2>Submit CV</h2>
		  <input type="file" id="cv" on:change={handleCVChange} accept=".pdf,.doc,.docx" />
		  <button class="btn btn-primary" on:click={handleCVSubmit}>Submit</button>
		  <button class="btn btn-secondary" on:click={() => (cvPopupVisible = false)}>Close</button>
		</div>
	  </div>
	{/if}
  </main>
  
  
  {#if selectedCandidate}
  <div class="popup">
	<div class="popup-content">
	  <h2>{editing ? 'Edit Candidate' : 'Add Candidate'}</h2>
	  <label for="firstName">First Name:</label>
	  <input type="text" id="firstName" bind:value={selectedCandidate.firstName} on:input={(e) => handleFieldChange(e, 'firstName')} />
	  <label for="surname">Surname:</label>
	  <input type="text" id="surname" bind:value={selectedCandidate.surname} on:input={(e) => handleFieldChange(e, 'surname')} />
	  <label for="email">Email:</label>
	  <input type="text" id="email" bind:value={selectedCandidate.email} on:input={(e) => handleFieldChange(e, 'email')} />
	  <label for="mobile">Mobile:</label>
	  <input type="text" id="mobile" bind:value={selectedCandidate.mobile} on:input={(e) => handleFieldChange(e, 'mobile')} />
	  <button class="btn btn-primary" on:click={handleSaveClick}>Save</button>
	  <button class="btn btn-secondary" on:click={() => (selectedCandidate = null)}>Close</button>
	</div>
  </div>
  {/if}
  {#if cvPopupVisible}
  <div class="popup">
    <div class="popup-content">
      <h2>Submit CV</h2>
      <input type="file" id="cv" on:change={handleCVChange} accept=".pdf,.doc,.docx" />
      <button class="btn btn-primary" on:click={() => {
        handleCVSubmit();
        cvPopupVisible = false;
      }}>Submit</button>
      <button class="btn btn-secondary" on:click={() => (cvPopupVisible = false)}>Close</button>
    </div>
  </div>
{/if}