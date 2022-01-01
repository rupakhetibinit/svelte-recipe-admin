<script>
import { store } from "../stores/auth.js";
import { orders } from "../stores/orders.js";
import Spinner from "./Spinner.svelte";

let message = document.getElementById("message");

let loading;
let colNames;

import axios from "axios";
const user = JSON.parse(localStorage.getItem("$store"));

// console.log(user, "user");
$: if ($orders === null) {
  update();
}
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
  if (res && res.data.orders) {
    if (res.data.orders === null) {
      message = "failed to load orders";
    } else {
      // console.log(res.data.orders);
      $orders = res.data.orders;
    }
    // colNames = Object.keys(res.data.orders[0]);
  } else if (res) {
    // console.log("error");
    message = "";
  } else {
    message = "error fetching data";
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
    .then((res) => {
      // console.log(res);
      if (res.status === 200) {
        // message = "Order Delivered";
        console.log("Order Delivered");
      } else {
        // message = "Error Delivering Order";
        console.log("Error Delivering Order");
      }
    })
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

function generateDateTime(date) {
  return new Date(date).toISOString().replace(/T/, " ").replace(/\..+/, "");
}
</script>

{#if message}
  <p id="message">{message}</p>
{/if}

{#if loading}
  <Spinner />
{/if}

{#if user.isAdmin == true}
  {#if $orders}
    <p>Showing {$orders.length} orders</p>
    <table class="content-table">
      <thead>
        <tr>
          <th> S.N. </th>
          <th> Order ID </th>
          <th> Order Delivered </th>
          <th> Order Date </th>
          <th> Order Total </th>
          <th> Order By </th>
          <th> Deliver Now </th>
          <th> Cancel Order </th>
        </tr>
      </thead>
      <tbody>
        {#each $orders as order, i (order.id)}
          <tr class="active-row">
            <td>{i + 1}.</td>
            <td>{order.id.split("-")[0].toUpperCase()}</td>
            <td style="text-transform: capitalize;letter-spacing:1px"
              >{order.delivered}</td>
            <td>{generateDateTime(order.createdAt)}</td>
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
