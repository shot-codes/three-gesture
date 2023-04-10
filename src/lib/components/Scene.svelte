<script>
  import { T, useFrame } from "@threlte/core";
  import { Environment, GLTF } from "@threlte/extras";
  import { gesture, handLocation } from "$lib/stores";

  let rotation = 0;
  let rotationDif = 0;
  let previousRotation = 0;
  let lastRotation = 0;
  let panning = false;

  $: {
    rotationDif = previousRotation - $handLocation * 3;
    previousRotation = $handLocation;
  }

  useFrame(() => {
    if ($gesture == "Closed_Fist") {
      panning = true;
      rotation = rotationDif;
    } else if (panning == true) {
      panning = false;
      rotation = previousRotation;
    }
  });
</script>

<Environment path="/hdr/" files="shanghai_riverside_1k.hdr" />

<T.PerspectiveCamera
  makeDefault
  position={[3, 1, 3]}
  on:create={({ ref }) => {
    ref.lookAt(0, 0, 0);
  }}
/>

<T.Group rotation.y={rotation}>
  <GLTF url="/gltf/DamagedHelmet.gltf" />
</T.Group>
