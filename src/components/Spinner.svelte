<script>
export let size = 60;
export let color = "#009879";
export let unit = "px";
</script>

<div class="circle" style="--size: {size}{unit}; --color: {color};" />

<style>
.circle {
  height: var(--size);
  width: var(--size);
  border-color: var(--color) transparent var(--color) var(--color);
  border-width: calc(var(--size) / 15);
  border-style: solid;
  border-image: initial;
  border-radius: 50%;
  animation: 0.75s linear 0s infinite normal none running rotate;
  margin: auto;
}
@keyframes rotate {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
