<script>
import axios from "axios";
import Select from "svelte-select";
import { users } from "../stores/users";
let newUsers;
let loading;
let message;
let selectedUser;
const user = JSON.parse(localStorage.getItem("$store"));
let wallet;
const config = {
  headers: {
    Authorization: "Bearer " + user.token,
    "Content-Type": "application/json",
  },
};

$: if ($users === null) {
  update();
}
function handleWalletAdd() {
  loading = true;
  console.log(selectedUser);
  axios
    .post(
      "https://recipetohome-api.herokuapp.com/api/v1/users/wallet",
      {
        userId: selectedUser.id,
        wallet: wallet,
      },
      config
    )
    .then((res) => {
      console.log(res.data);
      // message = `Wallet loaded to user ${res.data.user.name}`;
    })
    .catch((err) => {
      console.log(err);
      message = `Wallet failed to loaded to user ${selectedUser}`;
    })
    .finally(() => {
      loading = false;
    });
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
    newUsers = res.data.users.map((user) => {
      return {
        ...user,
        label: user.name,
        value: user.name,
      };
    });
    console.log(newUsers);
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

<h1>Add Money to users</h1>

<!--Creating a dropdown with all users -->
<form on:submit|preventDefault={handleWalletAdd}>
  {#if $users}
    <Select
      items={newUsers}
      bind:value={selectedUser}
      on:select={(event) => (selectedUser = event.detail)} />
    <input bind:value={wallet} type="number" required />

    <button disabled={loading} type="submit"> Submit </button>
    {#if message}
      <p>{message}</p>
    {/if}
  {/if}
</form>

<style>
</style>
