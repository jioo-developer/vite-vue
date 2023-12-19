<template>
  <div id="container">
    <div class="wrap">
      <div class="box" v-for="(items, boxIndex) in imgArr" :key="boxIndex">
        <img
          v-for="(item, index) in items"
          :key="index"
          :src="`https://picsum.photos/id/${item.id}/200/300`"
          :alt="item.author"
          class="item"
          @load="handleImageLoad"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imgArr: [],
      imgwidth: [],
    };
  },
  mounted() {
    this.loadImage();
  },
  methods: {
    async loadImage() {
      try {
        const response = await fetch(
          "https://picsum.photos/v2/list?page=2&limit=32"
        );
        const data = await response.json();
        this.splitImageGroup(data);
        this.$nextTick(() => this.diagonalSlide());
      } catch (error) {
        console.error("Failed to fetch images:", error);
      }
    },
    splitImageGroup(value) {
      // 이미지 배열의 길이가 0이 아닌 경우에만 그룹화 로직 수행
      if (value.length > 0) {
        const result = [];
        for (let i = 0; i < value.length; i += 8) {
          result.push(value.slice(i, i + 8));
        }
        this.imgArr = result;
      }
    },
    handleImageLoad(event) {
      const imgElement = event.target;
      this.imgwidth.push(imgElement.clientWidth);
    },
    diagonalSlide() {
      const el = Array.from(document.querySelectorAll(".box .item"));
      let itemLeft = 0;
      let itemSize = 0;
      const fontSize = parseFloat(
        getComputedStyle(document.documentElement).fontSize
      );
      const itemGap = 0.43 * fontSize;
    },
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
