<script>
import axios from "axios";
import { users } from "../stores/users";
import Spinner from "./Spinner.svelte";
const user = JSON.parse(localStorage.getItem("$store"));
let loading = "";
let message = "";
$: if ($users === null) {
  update();
}

async function update() {
  loading = true;
  const res = await axios.get(
    "https://recipetohome-api.herokuapp.com/api/v1/users",
    {
      headers: {
        Authorization: "Bearer " + user.token,
        "Content-Type": "application/json",
      },
    }
  );
  if (res && res.data.users) {
    console.log(res.data.users);
    $users = res.data.users;
    // colNames = Object.keys(res.data.users[0]);
  } else if (res) {
    // console.log("error");
    message = "";
  } else {
    message = "error fetching data";
  }
  loading = false;
}
</script>

{#if message}
  <p id="message">{message}</p>
{/if}

{#if loading}
  <Spinner />
{/if}

{#if user.isAdmin == true}
  {#if $users}
    <p>Showing {$users.length} users</p>
    <table class="content-table">
      <thead>
        <th>User ID</th>
        <th>Name</th>
        <th>Email</th>
        <th>Total Orders</th>
      </thead>
      <tbody>
        {#each $users as user (user.id)}
          <tr class="active-row">
            <td>{user.id}</td>
            <td>{user.name}</td>
            <td>{user.email}</td>
            <td>{user._count.orders}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
{:else}
  <p>You are not authorized to view this page</p>
{/if}

<style>
* {
  font-family: sans-serif; /* Change your font family */
}

.content-table {
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 0.9em;
  min-width: 400px;
  border-radius: 5px 5px 0 0;
  overflow: hidden;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
}

.content-table thead tr {
  background-color: #009879;
  color: #ffffff;
  text-align: left;
  font-weight: bold;
}

.content-table th,
.content-table td {
  padding: 12px 15px;
}

.content-table tbody tr {
  border-bottom: 1px solid #dddddd;
}

.content-table tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

.content-table tbody tr:last-of-type {
  border-bottom: 2px solid #009879;
}

.content-table tbody tr.active-row {
  font-weight: bold;
  color: #009879;
}
</style>
