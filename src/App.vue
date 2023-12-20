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
      nextTick(() => imgPositionProperty());
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

    function imgPositionProperty() {
      let itemLeft = 0;
      let itemSize = 0;
      const rememberLeft = [];
      const el = Array.from(document.querySelectorAll(".box .item"));
      const fontSize = parseFloat(
        getComputedStyle(document.documentElement).fontSize
      );
      const itemGap = 0.43 * fontSize;
      el.forEach((item) => {
        rememberLeft.push(Math.floor(itemLeft));
        console.log(itemLeft);
        item.style.left = Math.floor(itemLeft) + "px";
        itemLeft += item.offsetWidth + itemGap;
        itemSize = item.offsetWidth;
      });
      slideMoveLogic(el, rememberLeft, itemSize, itemGap);
    }

    function slideMoveLogic(el, rememberLeft, itemSize, itemGap) {
      let first = 1;
      let last = el.length;
      setInterval(() => {
        el.forEach((item, index) => {
          if (rememberLeft[index] < -itemSize) {
            const newLeft = rememberLeft[last - 1] + itemSize + itemGap;
            item.style.left = `${Math.floor(newLeft)}px`;
            rememberLeft.splice(index, 1, newLeft);
            first++;
            last++;
            if (last > el.length) last = 1;
            if (first > el.length) first = 1;
          } else {
            item.style.left = `${Math.floor(--rememberLeft[index])}px`;
          }
        });
      }, 10);
    }

    return {
      imgArr,
      imgwidth,
      loadData,
      loadImage,
      slideMoveLogic,
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
  height: 25rem;
  overflow: hidden;
}

#container > .wrap {
  position: absolute;
  /* container 보다 크기가 50% 더 크도록 설정 */
  width: 150%;
  height: 150%;
  /* 각도 조정 */
  transform: rotate(-114deg);
  /* 위치 조정 (선택사항) */
  top: -8rem;
  left: -5.5rem;
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
}
</style>
