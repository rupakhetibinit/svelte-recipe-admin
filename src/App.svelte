<script>
import Login from "./components/Login.svelte";
import { store } from "./stores/auth.js";
import MainApp from "./components/MainApp.svelte";
</script>

<main>
  {#if $store == null && localStorage.getItem("$store") == null}
    <Login />
  {:else}
    <MainApp />
  {/if}
</main>

<style>
main {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
</style>
