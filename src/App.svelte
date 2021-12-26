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
  text-align: center;
  padding: 1em;
  max-width: 500;
  margin: 0 auto;
}
</style>
