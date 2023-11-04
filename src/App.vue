<script setup lang="ts">
import { onMounted, ref } from 'vue';
// @ts-ignore
import { BASE_URL, API_KEY, notFound } from './constants';
// @ts-ignore
import Card from './components/Card.vue'

interface GifInfo {
  data: Array<any>;
}

const gifInfo = ref<GifInfo>({
  data: []
});

const searchPhrase = ref<string>("");
const page = ref<number>(1);
const selectedCard = ref<object[]>([]);
const hasNextPage = ref<boolean>(true);

const numberGifsOnPage = 10;

const getServerInfo = async () => {
  const response = await fetch(`${BASE_URL}?api_key=${API_KEY}&q=${searchPhrase.value}&limit=50&offset=0&rating=pg&lang=en&bundle=clips_grid_picker`);
  if (response.ok) {
    gifInfo.value = await response.json();
    page.value = 1;
    paginationInfo();

    if (!gifInfo?.value?.data?.length && searchPhrase.value) {
      gifInfo.value = notFound; // default gifs "Not found"
    }
  } else {
    alert(`Error!!!: ${response.status}`);
  }
}

const paginationInfo = () => {
  const start = (page.value - 1) * numberGifsOnPage;
  const end = page.value * numberGifsOnPage;

  hasNextPage.value = gifInfo.value.data.length > end;

  return gifInfo.value.data.slice(start, end);
}

const mountedStart = async () => {
  getStartPhrase();

  await getServerInfo();
};

const getStartPhrase = () => {
  const maxYear = 2024;
  const minYear = 1980;

  searchPhrase.value = `best in ${Math.floor(Math.random() * (maxYear - minYear)) + minYear}`;
};

const updateSelectCardInfo = (gif: object) => {
  selectedCard.value = [];
  selectedCard.value.push(gif);
};

onMounted(mountedStart)
</script>

<template>
  <section class="top-section">
    <img src="/logo.png" alt="Logo">
    <div>
      <input id="input" type="text" placeholder=" Search here" v-model="searchPhrase" @input="getServerInfo" />
    </div>
    <div v-if="!searchPhrase" class="type-text">Type something</div>
  </section>


  <div class="buttons-wrapper" v-if="!selectedCard?.length">
    <button v-if="page > 1" class="button leftBtn" @click="page--">Previus</button>
    <button v-if="hasNextPage" class="button rightBtn" @click="page++">Next</button>
  </div>

  <div class="gifs-wrapper">
    <TransitionGroup v-if="gifInfo?.data?.length && !selectedCard?.length" name="list" tag="ul" class="listContainer">
      <li v-for="gif in paginationInfo()" :key="gif" @click="() => { updateSelectCardInfo(gif); }" class="gifItem">

        <div class="gifImg" :style="`background-image: url(${gif.images?.fixed_width.webp});`">
        </div>

        <div v-if="gif.title" class="gifTitle">
          {{ gif.title }}
        </div>
        <div v-else>
          GIF without title
        </div>
      </li>
    </TransitionGroup>

    <div v-if="selectedCard?.length" class="cover-div"></div>
    <div v-if="selectedCard?.length" class="wrapper-card sticky">
      <button class="close-button " @click="selectedCard.length = 0">X</button>

      <Card :cardData="selectedCard[0]" :images="gifInfo?.data" />
    </div>


  </div>
</template>

<style scoped>
.stycky {
  position: sticky;
  top: 0;
}

.top-section {
  position: absolute;
  top: 0;
  left: 0;
  width: 400px;
}

.top-section>img {
  position: absolute;
  top: 0;
  left: 0;
  margin: 10px;
  width: 180px;
  border-radius: 5px;
}

.type-text {
  position: absolute;
  top: 50px;
  left: 200px;
}

.wrapper-card {
  position: absolute;
  top: 60px;
  left: 50%;
  width: 400px;
  height: 90vh;
  transform: translate(-50%, 0);
  z-index: 10;
}

.gifs-wrapper {
  position: absolute;
  top: 120px;
  left: 50%;
  width: 95%;
  height: 90vh;
  transform: translate(-50%, 0);
}

.buttons-wrapper {
  position: absolute;
  top: 0;
  left: 50%;
  width: 250px;
  height: 70px;
  transform: translate(-50%, 100%);
}

.leftBtn {
  position: absolute;
  top: 0;
  left: 0;
}

.rightBtn {
  position: absolute;
  top: 0;
  right: 0;
}

.cover-div {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9;
  width: 100%;
  height: 100%;
  background-color: rgb(0, 0, 0);
  opacity: 0.8;
  transform: scale(1.5);
}

.list-enter-active,
.list-leave-active {
  transition: all 0.25s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: scale(0.1);
}

#input {
  position: absolute;
  top: 20px;
  left: 200px;
  width: 170px;
  height: 24px;
  font-size: 16px;
}

.gifItem {
  position: relative;
  width: 200px;
}

.gifImg {
  position: relative;
  height: 200px;
  background-repeat: no-repeat;
  object-fit: cover;
}

.listContainer {
  display: flex;
  flex-wrap: wrap;
  position: relative;
  justify-content: center;
  gap: 20px;
  width: 100%;
}

.button {
  margin: 10px;
}

.close-button {
  position: fixed;
  top: 20px;
  left: 345px;
  background-color: #800d0d;
  z-index: 100;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 2.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

@media screen and (max-width: 480px) {
  .gifImg {
    width: 150px;
    height: 150px;
  }

  .gifItem {
    width: 150px;
  }

  .gifTitle {
    font-size: 12px;
  }
}
</style>

