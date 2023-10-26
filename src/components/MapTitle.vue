<template>
  <div class="content">
    <div class="flex-1">
      <h2>MAP USING MAPBOX GL</h2>
      <h2>MAP USING MAPBOX GL</h2>
    </div>
    <div class="flex-1 description">Maps showing the churches in California & Oregon in USA</div>
  </div>
</template>

<style scoped>
.content {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 14vh;
}

.flex-1 {
  text-align: center;
  width: 100vw;
}

.description {
  color: #fff;
}


.content h2 {
  text-align: center;
  color: #fff;
  font-size: 5em;
  position: absolute;
  top: 40px;
  left: calc(56% - 100px);
  transform: translate(-50%, -50%);
}

.content h2:nth-child(1) {
  color: white;
  -webkit-text-stroke: 2px #03a9f4;
}

.content h2:nth-child(2) {
  color: #03a9f4;
  animation: animate 4s ease-in-out infinite;
}

@keyframes animate {
  0%,
  100% {
    clip-path: polygon(
      0% 45%,
      16% 44%,
      33% 50%,
      54% 60%,
      70% 61%,
      84% 59%,
      100% 52%,
      100% 100%,
      0% 100%
    );
  }

  50% {
    clip-path: polygon(
      0% 60%,
      15% 65%,
      34% 66%,
      51% 62%,
      67% 50%,
      84% 45%,
      100% 46%,
      100% 100%,
      0% 100%
    );
  }
}

</style>