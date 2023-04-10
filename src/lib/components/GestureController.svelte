<script lang="ts">
  import { FilesetResolver, GestureRecognizer } from "@mediapipe/tasks-vision";
  import { drawConnectors, drawLandmarks } from "@mediapipe/drawing_utils";
  import { HAND_CONNECTIONS } from "@mediapipe/hands";
  import { onMount } from "svelte";
  import { gesture, handLocation } from "$lib/stores";

  let gestureRecognizer: GestureRecognizer;
  let gestureButtonText = "Enable Webcam";
  let detectedGesture: string;
  let detectedGestureConfidence: number;
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

    // getUsermedia parameters.
    const constraints = {
      video: true,
    };

    // Activate the webcam stream.
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
        detectedGesture = results.gestures[0][0].categoryName;
        $gesture = detectedGesture;
        $handLocation = results.landmarks[0][0].x;
        console.log(results.landmarks);
        detectedGestureConfidence = Math.round(results.gestures[0][0].score * 100);
      }

      if (webcamRunning === true) {
        window.requestAnimationFrame(predictWebcam);
      }
    }
  };
</script>

<div class="absolute flex flex-col bottom-0 right-0 items-end m-2 space-y-2">
  <div class="w-[200px] h-[150px]">
    <video
      bind:this={video}
      autoplay
      playsinline
      class="absolute w-[200px] h-[150px] scale-x-[-1]"
    />
    <canvas bind:this={canvas} class="absolute scale-x-[-1] w-[200px] h-[150px]" />
  </div>
  <div class="flex space-x-2">
    {#if detectedGesture}
      <div class="flex flex-col">
        <p class="dark:text-white">{detectedGesture}</p>
        <p class="dark:text-white">{detectedGestureConfidence}%</p>
      </div>
    {/if}
    <button on:click={enableCamera} class="p-2 border dark:text-white">
      {gestureButtonText}
    </button>
  </div>
</div>
