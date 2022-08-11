<script>
  import { onMount } from "svelte"
  import { getDefaultHeaders } from "./utils.js"
  let users = []
  let id = 0
  let email = ""
  let password = ""
  let name = ""
  let role = "User"
  let contactNumber = ""
  let alertMessage = ""

  onMount(async () => {
    await refreshUsers()
  })
  export async function refreshUsers() {
    users = await fetch("/api/v1/user", {
      headers: getDefaultHeaders(),
    }).then((r) => r.json())
  }
  export async function addUser() {
    await fetch("/api/v1/user", {
      method: "POST",
      headers: getDefaultHeaders(),
      body: JSON.stringify({
        username: email,
        password,
        name,
        role,
        contactNumber,
        email,
      }),
    })
      .then((r) => r.json())
      .then((d) => {
        if (d.status == "success") {
          refreshUsers()
        } else {
          alertMessage = d.message
        }
      })
  }
  export async function updateUser() {
    await fetch("/api/v1/user", {
      method: "PUT",
      headers: getDefaultHeaders(),
      body: JSON.stringify({
        id,
        username: email,
        password,
        name,
        role,
        contactNumber,
        email,
      }),
    })
      .then((r) => r.json())
      .then((d) => {
        if (d.status == "success") {
          refreshUsers()
        } else {
          alertMessage = d.message
        }
      })
  }
  export async function deleteUser(selectedID) {
    await fetch(`/api/v1/user/${selectedID}`, {
      method: "DELETE",
      headers: getDefaultHeaders(),
    })
      .then((r) => r.json())
      .then((d) => {
        if (d.status == "success") {
          refreshUsers()
        } else {
          alertMessage = d.message
        }
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
          <label for="email" class="form-label">Email</label>
          <input
            bind:value={email}
            id="email"
            type="email"
            class="form-control form-control-lg validate"
            placeholder="Email"
            required="required"
          />
        </div>
        <div class="mb-3">
          <label for="password" class="form-label">Password</label>
          <input
            bind:value={password}
            id="password"
            type="password"
            class="form-control form-control-lg validate"
            placeholder="Password"
            required="required"
          />
        </div>
        <div class="mb-3">
          <label for="name" class="form-label">Name</label>
          <input
            bind:value={name}
            id="name"
            type="text"
            class="form-control form-control-lg validate"
            placeholder="Name"
            required="required"
          />
        </div>
        <div class="mb-3">
          <label for="role" class="form-label">Role</label>
          <select
            bind:value={role}
            id="role"
            class="form-control form-control-lg validate"
            placeholder="Role"
            required="required"
          >
            <option value="User" selected>User</option>
            <option value="Admin">Admin</option>
          </select>
        </div>
        <div class="mb-3">
          <label for="contactNumber" class="form-label">Contact number</label>
          <input
            bind:value={contactNumber}
            id="contactNumber"
            type="tel"
            class="form-control form-control-lg validate"
            placeholder="Contact number"
            required="required"
          />
        </div>
        <div class="text-center mt-3">
          {#if id != 0}
            <button
              class="btn btn-lg btn-primary"
              on:click={() => {
                id = 0
                email = ""
                password = ""
                name = ""
                role = "User"
                contactNumber = ""
              }}>Cancel</button
            >
          {/if}
          <button
            on:click={async (e) => {
              e.preventDefault()
              id == 0 ? await addUser() : await updateUser()
            }}
            class="btn btn-lg btn-primary"
            >{id == 0 ? "Create new User" : `Update #${id} User`}</button
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
    </div>
  </div>
</div>

{#if users.length > 0}
  <table class="table table-striped">
    <thead>
      <tr>
        <td>Email</td>
        <td>Name</td>
        <td>Role</td>
        <td>Contact number</td>
        <td>Action</td>
      </tr>
    </thead>
    <tbody>
      {#each users as user}
        <tr>
          <td>{user.email}</td>
          <td>{user.name}</td>
          <td>{user.role}</td>
          <td>{user.contactNumber}</td>
          <td
            ><button
              class="btn btn-lg btn-primary"
              on:click={() => {
                id = user.id
                email = user.username
                name = user.name
                role = user.role
                contactNumber = user.contactNumber
              }}>Select</button
            ><button
              class="btn btn-lg btn-primary"
              on:click={async () => await deleteUser(user.id)}>Delete</button
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
