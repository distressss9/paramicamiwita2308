<!DOCTYPE html>
<html>
  <head>
    <title>para mi camiwita</title>
    <link href="https://fonts.googleapis.com/css2?family=Petit+Formal+Script&display=swap" rel="stylesheet">
    <style>
      canvas {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, .2);
      }
      #lover-name {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        color: white;
        font-family: 'Petit Formal Script', cursive;
        font-size: 3em;
        text-shadow: 0 0 10px rgba(255,255,255,0.5);
        z-index: 1;
        pointer-events: none;
        animation: glow 2s ease-in-out infinite alternate;
        letter-spacing: 1px;
      }
      @keyframes glow {
        from {
          text-shadow: 0 0 5px #fff, 
                       0 0 10px #fff, 
                       0 0 15px #ff69b4, 
                       0 0 20px #ff69b4, 
                       0 0 25px #ff69b4;
        }
        to {
          text-shadow: 0 0 10px #fff, 
                       0 0 20px #fff, 
                       0 0 30px #ff1493, 
                       0 0 40px #ff1493, 
                       0 0 50px #ff1493;
        }
      }
    </style>
  </head>
  <body>
    <div id="lover-name">Camila</div>
    <canvas id="heart" width="1920" height="947"></canvas>
    
    <script>
    window.requestAnimationFrame =
    window.__requestAnimationFrame ||
    window.requestAnimationFrame ||
    window.webkitRequestAnimationFrame ||
    window.mozRequestAnimationFrame ||
    window.oRequestAnimationFrame ||
    window.msRequestAnimationFrame ||
    (function () {
      return function (callback, element) {
        let lastTime = element.__lastTime;
        if (lastTime === undefined) {
          lastTime = 0;
        }
        let currTime = Date.now();
        let timeToCall = Math.max(1, 33 - (currTime - lastTime));
        window.setTimeout(callback, timeToCall);
        element.__lastTime = currTime + timeToCall;
      };
    })();
    window.isDevice = (/android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(((navigator.userAgent || navigator.vendor || window.opera)).toLowerCase()));
    let loaded = false;
    let init = function () {
    if (loaded) return;
    loaded = true;
    let mobile = window.isDevice;
    let koef = mobile ? 0.5 : 1;
    let canvas = document.getElementById('heart');
    let ctx = canvas.getContext('2d');
    let width = canvas.width = koef * innerWidth;
    let height = canvas.height = koef * innerHeight;
    let rand = Math.random;
    ctx.fillStyle = "rgba(0,0,0,1)";
    ctx.fillRect(0, 0, width, height);

    // Crear campo de repulsión alrededor del texto
    let nameElement = document.getElementById('lover-name');
    let nameRect = nameElement.getBoundingClientRect();
    let nameCenterX = nameRect.left + nameRect.width / 2;
    let nameCenterY = nameRect.top + nameRect.height / 2;
    let nameRadius = Math.max(nameRect.width, nameRect.height) * 0.7;

    let heartPosition = function (rad) {
      return [Math.pow(Math.sin(rad), 3), -(15 * Math.cos(rad) - 5 * Math.cos(2 * rad) - 2 * Math.cos(3 * rad) - Math.cos(4 * rad))];
    };
    let scaleAndTranslate = function (pos, sx, sy, dx, dy) {
      return [dx + pos[0] * sx, dy + pos[1] * sy];
    };

    window.addEventListener('resize', function () {
      width = canvas.width = koef * innerWidth;
      height = canvas.height = koef * innerHeight;
      ctx.fillStyle = "rgba(0,0,0,1)";
      ctx.fillRect(0, 0, width, height);
      
      // Actualizar posición del texto en resize
      nameRect = nameElement.getBoundingClientRect();
      nameCenterX = nameRect.left + nameRect.width / 2;
      nameCenterY = nameRect.top + nameRect.height / 2;
      nameRadius = Math.max(nameRect.width, nameRect.height) * 0.7;
    });

    let traceCount = mobile ? 20 : 50;
    let pointsOrigin = [];
    let i;
    let dr = mobile ? 0.3 : 0.1;
    for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 210, 13, 0, 0));
    for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 150, 9, 0, 0));
    for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 90, 5, 0, 0));
    let heartPointsCount = pointsOrigin.length;

    let targetPoints = [];
    let pulse = function (kx, ky) {
      for (i = 0; i < pointsOrigin.length; i++) {
        targetPoints[i] = [];
        targetPoints[i][0] = kx * pointsOrigin[i][0] + width / 2;
        targetPoints[i][1] = ky * pointsOrigin[i][1] + height / 2;
      }
    };

    let e = [];
    for (i = 0; i < heartPointsCount; i++) {
      let x = rand() * width;
      let y = rand() * height;
      e[i] = {
        vx: 0,
        vy: 0,
        R: 2,
        speed: rand() + 5,
        q: ~~(rand() * heartPointsCount),
        D: 2 * (i % 2) - 1,
        force: 0.2 * rand() + 0.7,
        f: "hsla(" + (rand() * 30 + 330) + ",100%," + (rand() * 40 + 40) + "%,.3)",
        trace: []
      };
      for (let k = 0; k < traceCount; k++) e[i].trace[k] = { x: x, y: y };
    }
    let config = {
      traceK: 0.4,
      timeDelta: 0.01
    };

    let time = 0;
    let loop = function () {
      let n = -Math.cos(time);
      pulse((1 + n) * .5, (1 + n) * .5);
      time += ((Math.sin(time)) < 0 ? 9 : (n > 0.8) ? .2 : 1) * config.timeDelta;
      ctx.fillStyle = "rgba(0,0,0,.1)";
      ctx.fillRect(0, 0, width, height);
      for (i = e.length; i--;) {
        var u = e[i];
        var q = targetPoints[u.q];
        var dx = u.trace[0].x - q[0];
        var dy = u.trace[0].y - q[1];
        var length = Math.sqrt(dx * dx + dy * dy);

        // Calcular distancia al texto
        let toTextDX = u.trace[0].x - nameCenterX;
        let toTextDY = u.trace[0].y - nameCenterY;
        let distanceToText = Math.sqrt(toTextDX * toTextDX + toTextDY * toTextDY);

        // Aplicar fuerza de repulsión si está cerca del texto
        if (distanceToText < nameRadius) {
          let repulsionForce = (1 - distanceToText / nameRadius) * 2;
          u.vx += (toTextDX / distanceToText) * repulsionForce;
          u.vy += (toTextDY / distanceToText) * repulsionForce;
        }

        if (10 > length) {
          if (0.95 < rand()) {
            u.q = ~~(rand() * heartPointsCount);
          } else {
            if (0.99 < rand()) {
              u.D *= -1;
            }
            u.q += u.D;
            u.q %= heartPointsCount;
            if (0 > u.q) {
              u.q += heartPointsCount;
            }
          }
        }

        u.vx += -dx / length * u.speed;
        u.vy += -dy / length * u.speed;
        u.trace[0].x += u.vx;
        u.trace[0].y += u.vy;
        u.vx *= u.force;
        u.vy *= u.force;
        for (k = 0; k < u.trace.length - 1;) {
          let T = u.trace[k];
          let N = u.trace[++k];
          N.x -= config.traceK * (N.x - T.x);
          N.y -= config.traceK * (N.y - T.y);
        }
        ctx.fillStyle = u.f;
        for (k = 0; k < u.trace.length; k++) {
          ctx.fillRect(u.trace[k].x, u.trace[k].y, 1, 1);
        }
      }
      ctx.fillStyle = "rgba(255,255,255,1)";
      for (i = u.trace.length + 13; i--;) ctx.fillRect(targetPoints[i][0], targetPoints[i][1], 2, 2);

      window.requestAnimationFrame(loop, canvas);
    };
    loop();
    };

    let s = document.readyState;
    if (s === 'complete' || s === 'loaded' || s === 'interactive') init();
    else document.addEventListener('DOMContentLoaded', init, false);
    </script>
  </body>
</html>
