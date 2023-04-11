<script lang="ts">
  import { FilesetResolver, GestureRecognizer } from "@mediapipe/tasks-vision";
  import { drawConnectors, drawLandmarks } from "@mediapipe/drawing_utils";
  import { HAND_CONNECTIONS } from "@mediapipe/hands";
  import { onMount } from "svelte";
  import { gesture, handLocation } from "$lib/stores";

  let gestureRecognizer: GestureRecognizer;
  let gestureButtonText = "Enable Webcam";
  let detectedGesture = "N/A";
  let detectedGestureConfidence = -1;
  let webcamRunning = false;
  let video: HTMLVideoElement;
  let canvas: HTMLCanvasElement;
  let canvasCtx: CanvasRenderingContext2D | null;

  onMount(async () => {
    canvasCtx = canvas.getContext("2d");
    const vision = await FilesetResolver.forVisionTasks("/wasm");
    gestureRecognizer = await GestureRecognizer.createFromOptions(vision, {
      baseOptions: {
        modelAssetPath: "/models/gesture_recognizer.task",
      },
      runningMode: "VIDEO",
      numHands: 1,
    });
  });

  const enableCamera = () => {
    if (!gestureRecognizer) {
      alert("Please wait for gestureRecognizer to load");
      return;
    }

    if (webcamRunning === true) {
      webcamRunning = false;
      gestureButtonText = "Enable Preditictions";
    } else {
      webcamRunning = true;
      gestureButtonText = "Disable Preditictions";
    }

    const constraints = {
      video: true,
    };

    navigator.mediaDevices.getUserMedia(constraints).then(function (stream) {
      video.srcObject = stream;
      video.addEventListener("loadeddata", predictWebcam);
    });
  };

  const predictWebcam = () => {
    if (canvasCtx) {
      let nowInMs = Date.now();
      const results = gestureRecognizer.recognizeForVideo(video, nowInMs);
      canvasCtx.save();
      canvasCtx.clearRect(0, 0, canvasCtx.canvas.width, canvasCtx.canvas.height);
      if (results.landmarks) {
        for (const landmarks of results.landmarks) {
          drawConnectors(canvasCtx, landmarks, HAND_CONNECTIONS, {
            color: "#00FF00",
            lineWidth: 5,
          });
          drawLandmarks(canvasCtx, landmarks, { color: "#FF0000", lineWidth: 2 });
        }
      }
      canvasCtx.restore();

      if (results.gestures.length > 0) {
        $gesture = detectedGesture;
        $handLocation = results.landmarks[0][0].x;
        detectedGesture = results.gestures[0][0].categoryName;
        detectedGestureConfidence = Math.round(results.gestures[0][0].score * 100);
      }

      if (webcamRunning === true) {
        window.requestAnimationFrame(predictWebcam);
      }
    }
  };
</script>

<div
  class="absolute bottom-0 right-0 m-2 flex flex-col items-end space-y-1 rounded bg-neutral-800 p-2"
>
  <div class="mb-2 h-[150px] w-[200px]">
    <img class="absolute" src="camera-placeholder.png" alt="camera placeholder" />
    <video
      bind:this={video}
      autoplay
      playsinline
      class="absolute h-[150px] w-[200px] scale-x-[-1]"
    />
    <canvas bind:this={canvas} class="absolute h-[150px] w-[200px] scale-x-[-1]" />
  </div>

  <div class="flex w-full justify-between px-1 text-sm">
    <p class="text-neutral-500">GESTURE</p>
    <p class="text-neutral-300">{detectedGesture}</p>
  </div>

  <div class="flex w-full justify-between px-1 text-sm">
    <p class="text-neutral-500">CONFIDENCE</p>
    <p class="text-neutral-300">{detectedGestureConfidence}%</p>
  </div>

  <button
    on:click={enableCamera}
    class="w-full rounded border border-neutral-500 bg-neutral-900 p-2 text-sm text-neutral-300"
  >
    {gestureButtonText}
  </button>
</div>
