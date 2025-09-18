<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>WebGL Test</title>
</head>
<body>
  <canvas id="c"></canvas>
  <script>
    const canvas = document.getElementById("c");
    const gl = canvas.getContext("webgl");
    if (gl) {
      document.body.style.background = "lightgreen";
      document.body.innerHTML = "<h1>✅ WebGL 動作OK</h1>";
    } else {
      document.body.style.background = "pink";
      document.body.innerHTML = "<h1>❌ WebGL 非対応</h1>";
    }
  </script>
</body>
</html>
