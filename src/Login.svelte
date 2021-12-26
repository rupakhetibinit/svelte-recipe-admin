<script>
// Get user from svelte store using $store
import { store } from "./auth.js";
let password = "";
let email = "";
let loading = null;
import axios from "axios";

const onLogin = () => {
  if (password.length < 6) {
    alert("Password must be at least 6 characters long");
    return;
  }
  if (!email.includes("@")) {
    alert("Enter valid Email");
    return;
  }
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
      console.log(err);
    })
    .finally(() => {
      loading = false;
    });
};
</script>

<h1>Login Screen</h1>
<body>
  <form action={onLogin} method="post">
    <input bind:value={email} type="text" name="email" placeholder="email" />
    <input
      bind:value={password}
      type="password"
      name="password"
      placeholder="Password" />
    <button on:click|preventDefault={onLogin}
      >{loading ? "Loading" : "Login"}</button>
  </form>
</body>
