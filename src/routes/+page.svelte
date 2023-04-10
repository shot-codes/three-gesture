<script>
  import { Canvas } from "@threlte/core";
  import { OrbitControls } from "@threlte/extras";
  import Scene from "$lib/components/Scene.svelte";
  import GestureController from "$lib/components/GestureController.svelte";
</script>

<GestureController />

<div class="w-full h-full">
  <Canvas>
    <Scene />
  </Canvas>
</div>
