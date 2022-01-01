<script>
import axios from "axios";
import { store } from "../stores/auth";

let input;
let container;
let image;
let placeholder;
let showImage = false;
const user = JSON.parse(localStorage.getItem("$store"));
let ingredientError = "";
let stepError = "";
let newError = "";
let success = "";
$: setInterval(() => {
  newError = "";
}, 5000);
$: setInterval(() => {
  success: "";
}, 5000);

let recipeObject = {
  name: "",
  ingredients: [],
  steps: [],
  imageUrl: "",
  description: "",
  servings: 0,
};
let step;
let tableIngredient = {
  name: "",
  measurement: "",
  amount: 0,
  price: "",
};
let tableStep = "";
function onChange() {
  const file = input.files[0];

  if (file) {
    showImage = true;

    const reader = new FileReader();
    reader.addEventListener("load", function () {
      image.setAttribute("src", reader.result);
    });
    reader.readAsDataURL(file);

    return;
  }
  showImage = false;
}

const addIngredient = (ingredient) => {
  if (ingredient.name === "") {
    ingredientError = "Ingredient name is required";
    return;
  }
  if (ingredient.measurement === "") {
    ingredientError = "Ingredient measurement is required";
    return;
  }
  if (ingredient.amount === 0) {
    ingredientError = "Ingredient amount is required";
    return;
  }
  if (ingredient.price === "") {
    ingredientError = "Ingredient price is required";
    return;
  }
  const ingredientObject = {
    name: ingredient.name,
    measurement: ingredient.measurement,
    amount: ingredient.amount,
    price: ingredient.price,
  };
  recipeObject.ingredients.push(ingredientObject);
  recipeObject = recipeObject;
  tableIngredient = {
    name: "",
    measurement: "",
    amount: 0,
    price: 0,
  };
  console.log(recipeObject);
};

const addSteps = (step) => {
  if (step === "") {
    stepError = "Step is required";
    return;
  }
  recipeObject.steps.push(step);
  recipeObject = recipeObject;
  console.log(recipeObject);
  tableStep = "";
};

const handleUpload = () => {
  const file = input.files[0];

  if (file) {
    const formData = new FormData();
    formData.append("recipe-file", file);

    axios
      .post(
        "https://recipetohome-api.herokuapp.com/image-upload-single",
        formData,
        {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        }
      )
      .then((res) => {
        console.log(res);
        if (res.statusText === "OK") {
          recipeObject.imageUrl = res.data.path;
          recipeObject = recipeObject;
          console.log(recipeObject.imageUrl);
        } else {
          console.log("failed");
        }
      })
      .catch((err) => {
        console.log(err);
      });
  }
};

const handleAddToDatabase = () => {
  if (recipeObject.name === "") {
    newError = "Recipe name is required";
    return;
  }
  if (recipeObject.description === "") {
    newError = "Recipe description is required";
    return;
  }
  if (recipeObject.servings === 0) {
    newError = "Recipe servings is required";
    return;
  }
  if (recipeObject.ingredients.length === 0) {
    newError = "Recipe ingredients is required";
    return;
  }
  if (recipeObject.steps.length === 0) {
    newError = "Recipe steps is required";
    return;
  }
  if (recipeObject.imageUrl === "") {
    newError = "Upload Image first";
    return;
  }

  axios
    .post(
      "https://recipetohome-api.herokuapp.com/api/v1/recipes",
      recipeObject,
      {
        headers: {
          Authorization: `Bearer ${user.token}`,
          "Content-Type": "application/json",
        },
      }
    )
    .then((res) => {
      console.log(res);
      if (res.statusText === "OK") {
        console.log("success");
        recipeObject = {
          name: "",
          ingredients: [],
          steps: [],
          imageUrl: "",
          description: "",
          servings: 0,
        };
        success = "Recipe added successfully";
      } else {
        console.log("failed");
      }
    })
    .catch((err) => {
      console.log(err);
    });
};
</script>

<h1>Add Recipe</h1>

<input bind:value={recipeObject.name} placeholder="Recipe Name" type="text" />

<input
  bind:value={recipeObject.description}
  placeholder="Description"
  type="text" />

<input
  bind:value={recipeObject.servings}
  placeholder="Servings"
  type="number" />

<p>steps</p>

<!-- array steps field -->
<div style="display: flex;flex-direction:column">
  {#each recipeObject.steps as step, i}
    <input bind:value={step} placeholder="Step" type="text" />
  {/each}
  <input bind:value={tableStep} placeholder="Step" type="text" />
  <p style="color:red">{stepError}</p>
  <button on:click={() => addSteps(tableStep)} type="button">Add Step</button>
</div>

<div style="display: flex;flex-direction:column">
  {#each recipeObject.ingredients as ingredient}
    <input bind:value={ingredient.name} placeholder="Ingredient" type="text" />
    <input
      bind:value={ingredient.measurement}
      placeholder="Measurement"
      type="text" />
    <input bind:value={ingredient.amount} placeholder="Amount" type="number" />
    <input bind:value={ingredient.price} placeholder="Price" type="number" />
    <p>****************************************</p>
  {/each}

  <input
    bind:value={tableIngredient.name}
    on:focus={(ingredientError = "")}
    placeholder="Ingredient"
    type="text" />
  <input
    bind:value={tableIngredient.measurement}
    on:focus={(ingredientError = "")}
    placeholder="Measurement"
    type="text" />
  <input
    bind:value={tableIngredient.amount}
    on:focus={(ingredientError = "")}
    placeholder="Amount"
    type="number" />
  <input
    bind:value={tableIngredient.price}
    placeholder="Price"
    on:focus={(ingredientError = "")}
    type="number" />

  <p style="color:red">{ingredientError}</p>
  <button on:click={() => addIngredient(tableIngredient)} type="button"
    >Add Ingredient</button>
</div>

<h2>upload image of recipe</h2>
<input bind:this={input} on:change={onChange} type="file" />
<div bind:this={container}>
  {#if showImage}
    <img bind:this={image} src="" alt="Preview" />
  {:else}
    <span bind:this={placeholder}>Image Preview</span>
  {/if}
</div>
<button
  disabled={recipeObject.imageUrl.length > 0 ? true : false}
  on:click|preventDefault={handleUpload}
  >{recipeObject.imageUrl.length > 0 ? "Uploaded" : "Upload"}</button>

<p style="color:red">{newError ? newError : success ? success : ""}</p>
<button on:click|preventDefault={handleAddToDatabase}>Add to Database</button>

<style>
div {
  width: 300px;
  min-height: 100px;
  border: 2px solid #ddd;
  margin-top: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: #ccc;
}
img {
  width: 100%;
}
</style>
