<script>
// Get user from svelte store using $store
import { store } from "../stores/auth.js";
let password = "";
let email = "";
let loading = null;
let error = { email: "", password: "" };
import axios from "axios";

const onLogin = () => {
  if (!email || !password) {
    error.email = "Email is required";
    error.password = "Password is required";
  }
  if (!email.includes("@")) {
    error.email = "Email is invalid";
  }
  if (password.length < 6) {
    error.password = "Password must be at least 8 characters";
  }
  if (error.email || error.password) {
    return;
  }

  loading = true;
  axios
    .post("https://recipetohome-api.herokuapp.com/api/auth/login", {
      email: email,
      password: password,
    })
    .then((res) => {
      // console.log(res.data);
      $store = {
        token: res.data.accessToken,
        name: res.data.name,
        email: res.data.email,
        isAdmin: res.data.isAdmin,
      };
      localStorage.setItem("$store", JSON.stringify($store));
    })
    .catch((err) => {
      // console.log(err);
      error.password = "Email or password is incorrect";
    })
    .finally(() => {
      loading = false;
    });
};
</script>

<h1>Login Screen</h1>

<form action={onLogin} method="post">
  <input
    bind:value={email}
    type="text"
    name="email"
    placeholder="email"
    on:focus={() => {
      error.email = "";
    }} />
  <p hidden={!error.email} style="color:red">
    {error.email ? error.email : " "}
  </p>
  <input
    bind:value={password}
    type="password"
    name="password"
    placeholder="Password"
    on:focus={() => {
      error.password = "";
    }} />
  <p style="color:red" hidden={!error.password}>
    {error.password ? error.password : " "}
  </p>
  <button on:click|preventDefault={onLogin} disabled={loading}
    >{loading ? "Loading" : "Login"}</button>
</form>

<style>
form {
  display: flex;
  flex-direction: column;
  align-items: center;
}
p {
  margin: 0;
  margin-bottom: 0.25em;
}
</style>
