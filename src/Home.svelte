<script>
  import { onMount } from "svelte"
  import { getDefaultHeaders } from "./utils.js"
  let images = []
  let id = 0
  let uri = ""
  let hits = 1
  let owner = ""
  let alertMessage = ""
  let limit = 5

  onMount(async () => {
    await refreshImages()
  })
  export async function refreshImages() {
    images = await fetch("/api/v1/images/all", {
      headers: getDefaultHeaders(),
    }).then((r) => r.json())
    images.reverse()
  }
  export async function getImages() {
    await fetch(`/api/v1/images?limit=${limit}`, {
      headers: getDefaultHeaders(),
    })
      .then((r) => r.json())
      .then(() => refreshImages())
  }
  export async function addImage() {
    await fetch("/api/v1/images", {
      method: "POST",
      headers: getDefaultHeaders(),
      body: JSON.stringify({
        owner,
        uri,
      }),
    }).then(() => {
      refreshImages()
    })
  }
  export async function updateImage(selectedID) {
    await fetch(`/api/v1/images/${selectedID}`, {
      method: "PATCH",
      headers: getDefaultHeaders(),
      body: JSON.stringify({
        id,
        hits,
        uri,
      }),
    }).then(() => {
      refreshImages()
    })
  }
  export async function deleteImage(selectedID) {
    await fetch(`/api/v1/images/${selectedID}`, {
      method: "DELETE",
      headers: getDefaultHeaders(),
    }).then(() => {
      refreshImages()
    })
  }
</script>

<div class="card">
  <div class="card-body">
    <div class="m-sm-4">
      <form>
        {#if id != 0}
          <div class="mb-3">
            <label for="id" class="form-label">ID</label>
            <input
              bind:value={id}
              id="id"
              type="text"
              class="form-control form-control-lg"
              placeholder="ID"
              readonly
            />
          </div>
        {/if}
        <div class="mb-3">
          <label for="owner" class="form-label">Owner</label>
          <input
            bind:value={owner}
            id="owner"
            type="text"
            class="form-control form-control-lg"
            placeholder="Owner"
            required="required"
          />
        </div>
        <div class="mb-3">
          {#if uri != ""}
            <img alt="200" height="200" class="responsive-img" src={uri} />
          {/if}
          <label for="uri" class="form-label">Image URI</label>
          <input
            bind:value={uri}
            id="uri"
            type="text"
            class="form-control form-control-lg"
            placeholder="Image URI"
            required="required"
          />
        </div>
        {#if id != 0}
          <div class="mb-3">
            <label for="hits" class="form-label">Hits</label>
            <input
              bind:value={hits}
              id="hits"
              type="number"
              class="form-control form-control-lg"
              placeholder="Hits"
              required="required"
            />
          </div>
        {/if}
        <div class="text-center mt-3">
          {#if id != 0}
            <button
              class="btn btn-lg btn-primary"
              on:click={() => {
                id = 0
                uri = ""
                hits = 1
                owner = ""
              }}>Cancel</button
            >
          {/if}
          <button
            on:click={async (e) => {
              e.preventDefault()
              id == 0 ? await addImage() : await updateImage(id)
            }}
            class="btn btn-lg btn-primary"
            >{id == 0 ? "Create new Image" : `Update #${id} Image`}</button
          >
        </div>
        {#if alertMessage != ""}
          <div
            class="text-center mt-3 alert alert-danger alert-dismissible"
            role="alert"
          >
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="alert"
              aria-label="Close"
            />
            <div class="alert-message">
              <strong>{alertMessage}</strong>
            </div>
          </div>
        {/if}
      </form>
      <div class="mb-3">
        <label for="limit" class="form-label">Quantity</label>
        <input
          bind:value={limit}
          id="limit"
          type="number"
          min="1"
          max="10"
          step="1"
          class="form-control form-control-lg"
          placeholder="Quantity"
          required="required"
        />
        <button
          class="btn btn-secondary"
          on:click={async () => await getImages()}
          >Get Random Image{limit > 1 ? "s" : ""}</button
        >
      </div>
    </div>
  </div>
</div>

{#if images.length > 0}
  <table class="table table-striped">
    <thead>
      <tr>
        <td>Owner</td>
        <td>Image</td>
        <td>Hits</td>
        <td>Action</td>
      </tr>
    </thead>
    <tbody>
      {#each images as image}
        <tr>
          <td>{image.username}</td>
          <td>
            <img
              alt="200"
              height="200"
              class="responsive-img"
              src={image.uri}
            /></td
          >
          <td>{image.hits}</td>
          <td
            ><button
              class="btn btn-lg btn-primary"
              on:click={() => {
                id = image.id
                uri = image.uri
                hits = image.hits
                owner = image.username
              }}>Select</button
            ><button
              class="btn btn-lg btn-primary"
              on:click={async () => await deleteImage(image.id)}>Delete</button
            ></td
          >
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

<style>
  .alert-message {
    box-sizing: border-box;
    padding: 0.95rem;
    width: 100%;
  }

  .alert-dismissible .btn-close {
    padding: 1.1875rem 0.95rem;
    position: absolute;
    right: 0;
    top: 0;
    z-index: 2;
  }

  .alert {
    display: flex;
    padding: 0;
  }

  .alert-danger {
    background-color: #f8d7da;
    border-color: #f5c2c7;
    color: #842029;
  }
  .alert-dismissible {
    padding-right: 2.85rem;
  }
  .alert {
    border: 0 solid transparent;
    border-radius: 0.2rem;
    margin-bottom: 1rem;
    padding: 0.95rem;
    position: relative;
  }
</style>
