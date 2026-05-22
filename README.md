<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Eye Tracking + Dwell Timer</title>

<!-- MediaPipe -->
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>

<style>

body{
  margin:0;
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
  background:#111827;
  font-family:Arial,sans-serif;
  overflow:hidden;
}

/* knop */
.dwell-btn{
  position:relative;
  overflow:hidden;

  width:320px;
  height:110px;

  border:none;
  border-radius:24px;

  background:#22c55e;
  color:white;

  font-size:1.5rem;
  font-weight:bold;

  cursor:pointer;
}

/* ring */
.dwell-ring{
  position:absolute;
  inset:0;
  border-radius:inherit;
  pointer-events:none;

  background:
    conic-gradient(
      rgba(255,255,255,.45) 0deg,
      transparent 0deg
    );
}

/* cursor */
#eyeCursor{
  position:fixed;
  width:24px;
  height:24px;
  border-radius:50%;

  background:#00e5ff;
  border:3px solid white;

  transform:translate(-50%,-50%);

  pointer-events:none;

  z-index:9999;
}

.text{
  position:relative;
  z-index:2;
}

</style>
</head>
<body>

<div id="eyeCursor"></div>

<button class="dwell-btn" id="myButton">

  <span class="text">
    Kijk 2 seconden naar de knop
  </span>

  <div class="dwell-ring"></div>

</button>

<script>

/* =========================
   INSTELLINGEN
========================= */

const dwellDuration = 2000;

const button =
  document.getElementById("myButton");

const ring =
  button.querySelector(".dwell-ring");

/* =========================
   EYE TRACKING
========================= */

let faceMesh;
let camera;

startTracking();

function startTracking(){

  const video =
    document.createElement("video");

  video.autoplay = true;
  video.playsInline = true;
  video.style.display = "none";

  document.body.appendChild(video);

  faceMesh = new FaceMesh({
    locateFile:(file)=>
      `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
  });

  faceMesh.setOptions({
    maxNumFaces:1,
    refineLandmarks:true,
    minDetectionConfidence:0.5,
    minTrackingConfidence:0.5
  });

  faceMesh.onResults(onFaceResults);

  camera = new Camera(video,{
    onFrame: async () => {
      await faceMesh.send({image:video});
    },
    width:640,
    height:480
  });

  camera.start();
}

/* =========================
   OOG POSITIE
========================= */

function onFaceResults(results){

  if(!results.multiFaceLandmarks?.length)
    return;

  const face =
    results.multiFaceLandmarks[0];

  const irisL = face[468];
  const irisR = face[473];

  let x =
    1 - ((irisL.x + irisR.x) / 2);

  let y =
    ((irisL.y + irisR.y) / 2);

  /* gevoeligheid */
  x = (x - 0.5) * 5 + 0.5;
  y = (y - 0.5) * 8 + 0.5;

  x = Math.max(0, Math.min(1, x));
  y = Math.max(0, Math.min(1, y));

  const px = x * window.innerWidth;
  const py = y * window.innerHeight;

  /* cursor bewegen */
  const cursor =
    document.getElementById("eyeCursor");

  cursor.style.left = px + "px";
  cursor.style.top = py + "px";

  checkDwell(px, py);
}

/* =========================
   DWELL TIMER
========================= */

let hoverStart = 0;
let hovering = false;

function checkDwell(x, y){

  const rect =
    button.getBoundingClientRect();

  const inside =
    x > rect.left &&
    x < rect.right &&
    y > rect.top &&
    y < rect.bottom;

  /* cursor komt op knop */
  if(inside && !hovering){

    hovering = true;

    hoverStart = Date.now();
  }

  /* cursor verlaat knop */
  if(!inside){

    hovering = false;

    ring.style.background = "";
    return;
  }

  /* progress */
  const progress =
    Math.min(
      (Date.now() - hoverStart)
      / dwellDuration,
      1
    );

  /* ring vullen */
  ring.style.background =
    `conic-gradient(
      rgba(255,255,255,.45)
      ${progress * 360}deg,
      transparent 0deg
    )`;

  /* dwell klaar */
  if(progress >= 1){

    hovering = false;

    ring.style.background = "";

    window.location.href = "pagina2.html";
  }
}

</script>

</body>
</html>
