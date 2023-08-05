<template>
  <section>
    <div class="scene">
      <div class="cube" :style="{ transform: `rotateX(${-beta}deg) rotateY(${gamma}deg) rotateZ(${-alpha}deg)` }">
        <div class="cube__face cube__face--front">front</div>
        <div class="cube__face cube__face--back">back</div>
        <div class="cube__face cube__face--right">right</div>
        <div class="cube__face cube__face--left">left</div>
        <div class="cube__face cube__face--top">top</div>
        <div class="cube__face cube__face--bottom">bottom</div>
      </div>
    </div>
  </section>
</template>

<script setup>
let alpha = ref(0)
let beta = ref(0)
let gamma = ref(0)

function websocket(isMobile) {

  const url = 'ws://192.168.1.105:8055/websocket';
  const access_token = 'pT8BXfityh2jhD2XByrfFFvRcqfv1kRg';
  const collection = 'coordinates';

  const connection = new WebSocket(url);

  connection.addEventListener('open', function () {
    console.log("websocket connection open");
    // auth
    connection.send(
      JSON.stringify({
        type: 'auth',
        access_token,
      })
    );
  });

  connection.addEventListener('close', function () {
    console.log("websocket connection closed");
  });

  connection.addEventListener('error', function (error) {
    console.log("websocket connection error", error);
  });

  if (isMobile === false) {
    connection.addEventListener('message', function (message) {
      const data = JSON.parse(message.data);
      if (data.type === 'auth' && data.status === 'ok') {
        console.log("websocket auth success");
        // subscribe
        connection.send(
          JSON.stringify({
            type: 'subscribe',
            collection,
            query: { fields: ['x, y, z'] }
          })
        );
      }
      if (data.type === 'auth' && data.status !== 'ok') {
        console.log("websocket auth failed");
      }
      if (data.type === "subscription") {
        beta.value = data.data[0].x
        gamma.value = data.data[0].y
        alpha.value = data.data[0].z
      }
    });

  } else {

    setTimeout(() => {
      function updateCoordinates(x, y, z) {
        connection.send(
          JSON.stringify({
            type: 'items',
            collection: collection,
            action: 'update',
            data: { id: 1, x: x, y: y, z: z },
          })
        );
      }
      let lastCall = 0
      window.addEventListener('deviceorientation', e => {
        const now = Date.now();
        if (now - lastCall >= 16) {
          lastCall = now;
          updateCoordinates(e.beta, e.gamma, e.alpha);
        }
      });
    }, 2000);

  }

}

function isMobile() {
  return /Mobi|Android/i.test(navigator.userAgent);
}

onMounted(() => {
  websocket(isMobile());
})
</script>

<style scoped>
.scene {
  width: 200px;
  height: 200px;
  margin: 80px;
  perspective: 400px;
}

.cube {
  width: 200px;
  height: 200px;
  position: relative;
  transform-style: preserve-3d;
}

.cube__face {
  position: absolute;
  width: 200px;
  height: 200px;
  border: 2px solid black;
  line-height: 200px;
  font-size: 40px;
  font-weight: bold;
  color: white;
  text-align: center;
  background: hsla(0, 100%, 50%, 0.7);
}

.cube__face--front {
  transform: rotateY(0deg) translateZ(100px);
}

.cube__face--right {
  background: hsla(60, 100%, 50%, 0.7);
  transform: rotateY(90deg) translateZ(100px);
}

.cube__face--back {
  background: hsla(120, 100%, 50%, 0.7);
  transform: rotateY(180deg) translateZ(100px);
}

.cube__face--left {
  background: hsla(180, 100%, 50%, 0.7);
  transform: rotateY(-90deg) translateZ(100px);
}

.cube__face--top {
  background: hsla(240, 100%, 50%, 0.7);
  transform: rotateX(90deg) translateZ(100px);
}

.cube__face--bottom {
  background: hsla(300, 100%, 50%, 0.7);
  transform: rotateX(-90deg) translateZ(100px);
}
</style>
