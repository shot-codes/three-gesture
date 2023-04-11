<script>
  import { Canvas } from "@threlte/core";
  import Scene from "$lib/components/Scene.svelte";
  import GestureController from "$lib/components/GestureController.svelte";
</script>

<GestureController />

<div class="h-full w-full">
  <Canvas>
    <Scene />
  </Canvas>
</div>
