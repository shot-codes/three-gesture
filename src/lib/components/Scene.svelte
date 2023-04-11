<script>
  import { T, useFrame } from "@threlte/core";
  import { Environment, GLTF } from "@threlte/extras";
  import { gesture, handLocation } from "$lib/stores";

  let rotation = 0;
  let previousRotation = 0;
  let initialDif = 0;
  let rotationDif = 0;
  let panning = false;
  let firstPan = true;

  $: {
    rotationDif = $handLocation * 3;
  }

  useFrame(() => {
    if ($gesture == "Closed_Fist") {
      if (firstPan) {
        firstPan = false;
        initialDif = rotationDif;
      }
      panning = true;
      rotation = previousRotation + (initialDif - rotationDif);
    } else if (panning == true) {
      firstPan = true;
      panning = false;
      previousRotation = rotation;
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
