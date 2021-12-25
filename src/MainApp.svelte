<script>
  import {store} from './auth.js'
  import {orders} from'./orders.js'
  let response;
  let message = document.getElementById('message')
  const onLogout =async ()=>{
    localStorage.removeItem('$store')
    $store=null
  }
  import axios from 'axios';
  const user = JSON.parse(localStorage.getItem('$store'))
  console.log(user,'user')
  import { onMount } from 'svelte';
  onMount(async () =>{
   response = await axios.get('https://recipetohome-api.herokuapp.com/api/v1/orders',{
      headers:{
        'Authorization':'Bearer '+user.token,
        'Content-Type':'application/json'
      }
    })
    if(response){
      console.log(response.data.orders)
      $orders = response.data.orders
    }
    else{
      console.log('error')
      $orders = []
      message="Error fetching data"
    }})

    const handleDeliver = (id) =>{
      axios({
        method:'patch',
        url:'https://recipetohome-api.herokuapp.com/api/v1/order/'+id,
        headers:{
          'Authorization':'Bearer '+user.token,
          'Content-Type':'application/json'
        }
      }).then(res=>console.log(res.data)).catch(err=>console.log(err))
    }

    const handleCancel = (id) =>{
      axios({
        method:'delete',
        url:'https://recipetohome-api.herokuapp.com/api/v1/order/'+id,
        headers:{
          'Authorization':'Bearer '+user.token,
          'Content-Type':'application/json'
        }
      }).then(res=>console.log(res.data)).catch(err=>console.log(err))
    }
</script>

<style>
  th,td{
    text-align:center;
    padding-right: 30px;
    justify-content: center;
  }
</style>

<h1>You are currently logged in as {user.name}</h1>

{#if user.isAdmin==true}
<h2>You are an admin</h2>
{:else}
<h2>You do not have permission to edit</h2>
{/if}

{#if message}
<p id="message">{message}</p>
{/if}

{#if $orders}
<table style="display:flex;align-items: center;justify-content:center;flex-direction:column;max-width:none">
<thead style="padding: 25px;">
<tr>
<th>Order ID</th>
<th>Order Delivered</th>
<th>Order Date</th>
<th>Order Total</th>
<th>Order By</th>
<th>Deliver Now</th>
<th>Cancel Order</th>
</tr>
</thead>
<tbody>
{#each $orders as order}
<tr>
<td>{order.id.split('-')[0].toUpperCase()}</td>
<td>
  {order.delivered}
</td>
<td>{order.createdAt}</td>
<td>{order.total}</td>
<td>{order.user.name}</td>
<td>
  {#if order.delivered==false}
  <button on:click={handleDeliver(order.id)}>
    Deliver
  </button>
  {:else}
  <p>Delivered</p>
  {/if}
</td>
<td>
  {#if order.delivered==false}
  <button on:click={handleCancel(order.id)}>Cancel</button>
  {:else}
  <p>Already Delivered</p>
  {/if}
</td>
</tr>
{/each}
</table>
{/if}

<button type="submit" on:click|preventDefault={onLogout}>Logout</button>
