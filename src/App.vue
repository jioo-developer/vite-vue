<template>
  <div id="container">
    <div class="wrap" v-if="loadData">
      <div class="box" v-for="(items, boxIndex) in imgArr" :key="boxIndex">
        <img
          v-for="(item, index) in items"
          :key="index"
          :src="`https://picsum.photos/id/${item.id}/200/300`"
          :alt="item.author"
          class="item"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, nextTick } from "vue";

export default {
  setup() {
    const imgArr = ref([]);
    const imgwidth = ref([]);
    const loadData = ref(false);

    onMounted(async () => {
      await loadImage();
      nextTick(() => handleImageLoad());
    });

    async function loadImage() {
      try {
        const response = await fetch(
          "https://picsum.photos/v2/list?page=2&limit=32"
        );
        const data = await response.json();
        for (let i = 0; i < data.length; i += 8) {
          imgArr.value.push(data.slice(i, i + 8));
        }
        if (imgArr.value.length > 0) {
          loadData.value = true;
        }
      } catch (error) {
        console.error("Failed to fetch images:", error);
      }
    }

    function handleImageLoad() {
      const el = Array.from(document.querySelectorAll(".box .item"));
      const width = el.map((item) => item.offsetWidth);
      imgPositionProperty(width, el);
    }
    function imgPositionProperty(value, arr) {
      let itemLeft = 0;
      let imgSize = 0;
      const rememberLeft = [];
      const fontSize = parseFloat(
        getComputedStyle(document.documentElement).fontSize
      );
      const itemGap = 0.43 * fontSize;
      arr.forEach((item, index) => {
        rememberLeft.push(Math.floor(itemLeft));
        item.style.left = `${Math.floor(itemLeft)}px`;
        itemLeft = itemLeft + value[index] + itemGap;
        imgSize = value[index];
      });
      slideMoveLogic(arr, rememberLeft, imgSize, itemGap);
    }

    function slideMoveLogic(el, rememberLeft, imgSize, itemGap) {
      let first = 1;
      let last = el.length;
      let count = 0;
      console.log(rememberLeft);
      console.log(-imgSize);
      el.forEach((item, index) => {
        if (rememberLeft[index] < -imgSize) {
          console.log("성공");
          const newLeft = rememberLeft[last - 1] + imgSize + itemGap;
          item.style.left = `${Math.floor(newLeft)}px`;
          console.log(rememberLeft[index]);
          rememberLeft.splice(index, 1, newLeft);
          console.log(rememberLeft[index]);
          first++;
          last++;
          if (last > el.length) last = 1;
          if (first > el.length) first = 1;
        } else {
          console.log("실패");
          item.style.left = `${Math.floor(--rememberLeft[index])}px`;
        }
      });
    }

    return {
      imgArr,
      imgwidth,
      loadData,
      loadImage,
      handleImageLoad,
      imgPositionProperty,
    };
  },
};
</script>
<style>
* {
  margin: 0;
  padding: 0;
}
#container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

#container > .wrap {
  position: absolute;
  /* container 보다 크기가 50% 더 크도록 설정 */
  width: 150%;
  height: 150%;
  /* 각도 조정 */
  transform: rotate(-114deg);
  -webkit-transform: rotate(-114deg);
  -moz-transform: rotate(-114deg);
  -ms-transform: rotate(-114deg);
  -o-transform: rotate(-114deg);
  /* 위치 조정 (선택사항) */
  top: -8rem;
  left: -2.5rem;
}

#container > .wrap > .box {
  position: relative;
  width: 100%;
  height: 7.5rem; /* item 의 높이와 동일하게 설정 */
  overflow: hidden;
}

#container > .wrap > .box + .box {
  margin-top: 0.63rem;
}

#container > .wrap > div.box > .item {
  position: absolute;
  width: 7.5rem;
  height: 7.5rem;
  transform: rotate(90deg); /* 이미지 세로로 보이게 변경 */
}
</style>
