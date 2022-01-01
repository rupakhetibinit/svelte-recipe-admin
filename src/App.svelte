<script>
import Login from "./components/Login.svelte";
import { store } from "./stores/auth.js";

import Tabs from "./components/Tabs.svelte";
import Orders from "./components/Orders.svelte";
import Users from "./components/Users.svelte";
import Recipe from "./components/Recipe.svelte";
import Wallet from "./components/Wallet.svelte";
let items = [
  {
    label: "Orders",
    value: 1,
    component: Orders,
  },
  {
    label: "Users",
    value: 2,
    component: Users,
  },
  {
    label: "Add Recipes",
    value: 3,
    component: Recipe,
  },
  {
    label: "Add Money",
    value: 4,
    component: Wallet,
  },
];
const onLogout = async () => {
  $store = null;
  localStorage.clear();
  location.reload();
};
const user = JSON.parse(localStorage.getItem("$store"));
</script>

<main>
  {#if $store == null && user == null}
    <Login />
  {:else}
    <h2>Recipe to Home Admin Panel</h2>
    <Tabs items={items} />
    <button class="logout" type="submit" on:click={onLogout}>LogOut</button>
  {/if}
</main>

<style>
main {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
.logout {
  margin-top: 20px;
}
</style>
