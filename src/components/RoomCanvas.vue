<template>
  <div class="room-container">
    <h2>Random Room Generator</h2>
    <canvas ref="canvas" :width="600" :height="400"></canvas>
    <button @click="changeDimensions">Change Dimensions</button>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  setup() {
    const canvas = ref(null);
    const roomData = ref(null);
    const selectedDimensions = ref({ length: null, width: null });

    const roomFiles = ["simple.json", "triangle.json", "t_shape.json"];

    const loadRoomData = async () => {
      try {
        const randomFile = roomFiles[Math.floor(Math.random() * roomFiles.length)];
        const response = await fetch(`/${randomFile}`);
        if (!response.ok) throw new Error(`Failed to load ${randomFile}`);

        roomData.value = await response.json();
        drawRoom();
        selectRandomDimensions();
      } catch (error) {
        console.error("Error loading room data:", error);
      }
    };

    const drawRoom = () => {
      if (!roomData.value || !canvas.value) return;

      const ctx = canvas.value.getContext("2d");
      ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
      ctx.strokeStyle = "#333";
      ctx.lineWidth = 2;

      const corners = roomData.value.corners;
      if (!corners || corners.length < 2) return;

      let minX = Math.min(...corners.map(c => c.x));
      let maxX = Math.max(...corners.map(c => c.x));
      let minY = Math.min(...corners.map(c => c.y));
      let maxY = Math.max(...corners.map(c => c.y));

      const scaleX = canvas.value.width / (maxX - minX + 50);
      const scaleY = canvas.value.height / (maxY - minY + 50);
      const scale = Math.min(scaleX, scaleY);

      const transformX = (x) => (x - minX) * scale + 20;
      const transformY = (y) => (y - minY) * scale + 20;

      ctx.beginPath();
      corners.forEach((corner, index) => {
        const nextCorner = corners[(index + 1) % corners.length];
        ctx.moveTo(transformX(corner.x), transformY(corner.y));
        ctx.lineTo(transformX(nextCorner.x), transformY(nextCorner.y));
      });
      ctx.stroke();

      if (selectedDimensions.value.length && selectedDimensions.value.width) {
        ctx.strokeStyle = "#FF3B30";
        ctx.lineWidth = 2.5;

        const [p1, p2] = selectedDimensions.value.length;
        ctx.beginPath();
        ctx.moveTo(transformX(p1.x), transformY(p1.y));
        ctx.lineTo(transformX(p2.x), transformY(p2.y));
        ctx.stroke();

        const [p3, p4] = selectedDimensions.value.width;
        ctx.beginPath();
        ctx.moveTo(transformX(p3.x), transformY(p3.y));
        ctx.lineTo(transformX(p4.x), transformY(p4.y));
        ctx.stroke();
      }
    };

    const selectRandomDimensions = () => {
      if (!roomData.value) return;

      const corners = roomData.value.corners;
      if (corners.length < 3) return;

      let length = null;
      let width = null;

      if (corners.length >= 4) {
        for (let i = 0; i < 50; i++) {
          const randomIndexes = [...Array(corners.length).keys()].sort(() => 0.5 - Math.random());
          const p1 = corners[randomIndexes[0]];
          const p2 = corners[randomIndexes[1]];
          const p3 = corners[randomIndexes[2]];
          const p4 = corners[randomIndexes[3]];

          if (p1.x === p2.x || p1.y === p2.y) length = [p1, p2];
          if (p3.x === p4.x || p3.y === p4.y) width = [p3, p4];
          if (length && width) break;
        }
      }

      if (corners.length === 3 || !length || !width) {
        length = [corners[0], corners[1]];
        width = [corners[1], corners[2]];
      }

      selectedDimensions.value.length = length;
      selectedDimensions.value.width = width;

      drawRoom();
    };

    const changeDimensions = () => {
      selectRandomDimensions();
    };

    onMounted(loadRoomData);

    return {
      canvas,
      changeDimensions,
    };
  },
};
</script>

<style scoped>
.room-container {
  background: #f8f9fa;
  padding: 30px;
  max-width: 720px;
  margin: 40px auto;
  border-radius: 16px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
  text-align: center;
  transition: all 0.3s ease;
}
.room-container h2 {
  font-size: 24px;
  margin-bottom: 20px;
  color: #333;
}
canvas {
  border-radius: 10px;
  background-color: #fff;
  border: 1px solid #ddd;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}
button {
  margin-top: 20px;
  padding: 10px 24px;
  font-size: 16px;
  background-color: green;
  border: none;
  border-radius: 8px;
  color: #fff;
  border: 1px solid #fff;
  transition: all 0.5s ease;
}
button:hover {
  background-color: #fff;
  color: green;
  border: 1px solid green;
}
</style>
