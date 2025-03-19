<template>
  <div class="flex flex-col items-center bg-black min-h-screen text-white">
    <h1 class="text-3xl font-bold my-4 text-pink-500 neon-glow">🎲 픽셀 핀볼 챌린지 🎲</h1>

    <button
      @click="dropBall"
      class="mb-2 px-4 py-2 bg-purple-500 text-white rounded"
    >
      공 떨어뜨리기!
    </button>

    <div
      ref="gameContainer"
      class="border border-purple-500 shadow-lg bg-black"
      style="width: 500px; height: 10000px;"
    ></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { Engine, Render, Runner, Bodies, World, Events, Body } from 'matter-js';
import confetti from 'canvas-confetti';

const gameContainer = ref(null);
const engine = Engine.create();

onMounted(() => {
  const render = Render.create({
    element: gameContainer.value,
    engine,
    options: {
      width: 500,
      height: 10000,
      wireframes: false,
      background: "#000000",
    },
  });

  // 벽
  const leftWall = Bodies.rectangle(5, 5000, 10, 10000, { isStatic: true, render: { fillStyle: '#7e22ce' } });
  const rightWall = Bodies.rectangle(495, 5000, 10, 10000, { isStatic: true, render: { fillStyle: '#7e22ce' } });

  // 장애물들
  const obstacles = [];

  // 점 장애물 구간 (랜덤 배치)
  for (let y = 500; y <= 1500; y += 100) {
    for (let i = 0; i < 4; i++) {
      const x = Math.random() * 400 + 50;
      obstacles.push(Bodies.circle(x, y, 10, {
        isStatic: true,
        render: { fillStyle: "#f472b6" },
      }));
    }
  }

  // 랜덤 위치 회전하는 풍차 장애물 구간
  const windmillPositions = [
    { x: 100, y: 2000 },
    { x: 400, y: 2300 },
    { x: 250, y: 2600 },
    { x: 120, y: 2900 },
    { x: 380, y: 3200 },
  ];

  windmillPositions.forEach(({ x, y }) => {
    const windmill = Bodies.rectangle(x, y, 250, 15, { isStatic: true, render: { fillStyle: "#fbbf24" } });
    setInterval(() => {
      Body.rotate(windmill, 0.08);
    }, 20);
    obstacles.push(windmill);
  });

  // 큰 원형 장애물 구간
  for (let y = 3500; y <= 4500; y += 200) {
    obstacles.push(Bodies.circle(Math.random() * 400 + 50, y, 30, {
      isStatic: true,
      render: { fillStyle: "#06b6d4" },
    }));
  }

  // 사각 장애물 랜덤 각도 구간
  for (let y = 5000; y <= 6000; y += 250) {
    obstacles.push(Bodies.rectangle(Math.random() * 400 + 50, y, 150, 20, {
      isStatic: true,
      angle: Math.random() * Math.PI,
      render: { fillStyle: "#ef4444" },
    }));
  }

  // 지그재그 형태 장애물
  let direction = 1;
  for (let y = 6500; y <= 7500; y += 200) {
    obstacles.push(Bodies.rectangle(direction === 1 ? 100 : 400, y, 200, 10, {
      isStatic: true,
      angle: direction * 0.3,
      render: { fillStyle: "#a855f7" },
    }));
    direction *= -1;
  }

  // 새로운 장애물 (랜덤 위치 삼각형 장애물)
  for (let y = 8000; y <= 8800; y += 200) {
    obstacles.push(Bodies.polygon(Math.random() * 400 + 50, y, 3, 30, {
      isStatic: true,
      render: { fillStyle: "#14b8a6" },
    }));
  }

  // 최종 도착지점 (꼬깔콘 형태 경사로)
  const finalRampLeft = Bodies.rectangle(150, 9900, 400, 20, { isStatic: true, angle: -0.5, render: { fillStyle: "#10b981" } });
  const finalRampRight = Bodies.rectangle(350, 9900, 400, 20, { isStatic: true, angle: 0.5, render: { fillStyle: "#10b981" } });

  World.add(engine.world, [
    leftWall,
    rightWall,
    finalRampLeft,
    finalRampRight,
    ...obstacles,
  ]);

  Render.run(render);
  Runner.run(Runner.create(), engine);
});

// 공 생성 및 떨어뜨리기
function dropBall() {
  const randomX = Math.floor(Math.random() * 400) + 50;
  const ball = Bodies.circle(randomX, 10, 12, {
    restitution: 0.8,
    render: { fillStyle: "#34d399" },
  });
  World.add(engine.world, ball);
}
</script>
