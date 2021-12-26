<script>
import { store } from "./auth.js";
import { orders } from "./orders.js";
import Spinner from "./Spinner.svelte";
let message = document.getElementById("message");
const onLogout = async () => {
  $store = null;
  localStorage.clear();
  location.reload();
};
let loading;
let colNames;
import axios from "axios";
const user = JSON.parse(localStorage.getItem("$store"));
console.log(user, "user");
import { onMount } from "svelte";
onMount(() => update());
async function update() {
  loading = true;
  const res = await axios.get(
    "https://recipetohome-api.herokuapp.com/api/v1/orders",
    {
      headers: {
        Authorization: "Bearer " + user.token,
        "Content-Type": "application/json",
      },
    }
  );
  if (res) {
    // console.log(res.data.orders);
    $orders = res.data.orders;
    colNames = Object.keys(res.data.orders[0]);
  } else {
    // console.log("error");
    message = "Error fetching data";
  }
  loading = false;
}
const handleDeliver = async (id) => {
  loading = true;
  axios({
    method: "patch",
    url: "https://recipetohome-api.herokuapp.com/api/v1/order/" + id,
    headers: {
      Authorization: "Bearer " + user.token,
      "Content-Type": "application/json",
    },
  })
    .then((res) => console.log(res.data))
    .catch((err) => console.log(err))
    .finally(() => {
      update();
      loading = false;
    });
};

const handleCancel = async (id) => {
  axios({
    method: "delete",
    url: "https://recipetohome-api.herokuapp.com/api/v1/order/" + id,
    headers: {
      Authorization: "Bearer " + user.token,
      "Content-Type": "application/json",
    },
  })
    .then((res) => {
      $orders.filter((order) => order.id !== id);
      update();
    })
    .catch((err) => console.log(err));
};

// let sortBy = { col: "name", ascending: true };

// $: sort = (column) => {
//   if (sortBy.col == column) {
//     sortBy.ascending = !sortBy.ascending;
//   } else {
//     sortBy.col = column;
//     sortBy.ascending = true;
//   }
//   let sortModifier = sortBy.ascending ? 1 : -1;
//   let sort = (a, b) => {
//     a[column] < b[column]
//       ? -1 * sortModifier
//       : a[column] > b[column]
//       ? 1 * sortModifier
//       : 0;
//     $orders = $orders.sort(sort);
//   };
// };
</script>

<h1>You are currently logged in as {user.name}</h1>

{#if user.isAdmin == true}
  <h2>You are an admin</h2>
{:else}
  <h2>You do not have permission to edit</h2>
{/if}

{#if message}
  <p id="message">{message}</p>
{/if}

{#if loading}
  <Spinner />
{/if}

{#if $orders}
  <table>
    <thead>
      <tr>
        <th> Order ID </th>
        <th> Order Delivered </th>
        <th>Order Date</th>
        <th> Order Total </th>
        <th> Order By</th>
        <th>Deliver Now</th>
        <th>Cancel Order</th>
      </tr>
    </thead>
    <tbody>
      {#each $orders as order}
        <tr>
          <td>{order.id.split('-')[0].toUpperCase()}</td>
          <td>{order.delivered}</td>
          <td>{order.createdAt}</td>
          <td>{order.total}</td>
          <td>{order.user.name}</td>
          <td>
            {#if order.delivered == false}
              <button on:click={() => handleDeliver(order.id)}>
                Deliver Now
              </button>
            {:else if order.delivered == true}
              <button disabled>Delivered</button>
            {/if}
          </td>
          <td>
            {#if order.delivered == false}
              <button on:click={() => handleCancel(order.id)}>Cancel</button>
            {:else if order.delivered == true}
              <button disabled>Already Delivered</button>
            {/if}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

<button type="submit" on:click={onLogout}>LogOut</button>

<style>
table {
  max-width: 100%;
  border: 0;
  white-space: nowrap;
  border-collapse: collapse;
  border: 1px solid rgba(0, 0, 0, 0.12);
  border-radius: 4px;
  table-layout: fixed;
}

th {
  height: 56px;
  font-family: Roboto, sans-serif;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  font-size: 0.875rem;
  line-height: 1.375rem;
  font-weight: 500;
  letter-spacing: 0.00714em;
  text-decoration: inherit;
  text-transform: inherit;
  text-align: left;
  padding-right: 16px;
  padding-left: 16px;
}

tbody {
  font-family: Roboto, sans-serif;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  font-size: 0.875rem;
  line-height: 1.25rem;
  font-weight: 400;
  letter-spacing: 0.01786em;
  text-decoration: inherit;
  text-transform: inherit;
}
tr {
  height: 52px;
  border-top-color: rgba(0, 0, 0, 0.12);
  border-top-width: 1px;
  border-top-style: solid;
}
tr:hover {
  background-color: rgba(0, 0, 0, 0.04);
}

td {
  padding-right: 16px;
  padding-left: 16px;
}

button {
  margin-top: 20px;
}
</style>
