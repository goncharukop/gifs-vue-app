<script lang="ts" setup>
import { defineProps, ref, onMounted, computed, PropType } from 'vue';
const props = defineProps({
  cardData: {
    type: Object,
    required: true,
  },
  images: {
    type: Array as PropType<Image[]>,
    required: true,
  }
});

interface Image {
  id: string;
  images: {
    fixed_width: {
      webp: string;
    };
  };
}

const selectedGif = ref(props.cardData)

const currentIndex = ref<number>(0);

const timer = ref<number | null>(null);

const startSlide = () => {
  timer.value = setInterval(next, 3000);
};

const next = () => {
  currentIndex.value = (currentIndex.value + 1) % props.images.length;
};

const prev = () => {
  currentIndex.value = (currentIndex.value - 1 + props.images.length) % props.images.length;
};

const visibleImagesArray = ref<Image[]>([]);

const visibleImages = computed(() => {
  visibleImagesArray.value = [props.images[currentIndex.value]];

  for (let i = 1; i < 5; i++) {
    visibleImagesArray.value.push(props.images[(currentIndex.value + i) % props.images.length])
  }

  return visibleImagesArray.value;
});

const isAuthorCard = ref<boolean>(false);

const showAuthorCard = () => {
  isAuthorCard.value = !isAuthorCard.value;
}

const shareContent = async () => {
  try {
    if (navigator.share) {
      await navigator.share({
        text: props.cardData.title,
        url: props.cardData.url
      });
      console.log('Succes!');
    } else {
      alert('Web Share API not support');
    }
  } catch (error) {
    alert(`Error!: ${error}`);
  }
};

onMounted(() => {
  startSlide();
});
</script>

<template>
  <div v-if="!isAuthorCard">
    <div class="product-card card">
      <h3>{{ selectedGif?.title }}</h3>
      <img :src="selectedGif.images.original.webp" alt="Product Image" class="product-image">
      <div class="product-info">

        <p class="product-text"><b>Author:</b> {{ selectedGif.user?.display_name }}</p>
        <p class="product-text"><b>Description:</b> {{ selectedGif.user?.description }}</p>
      </div>

      <div v-if="images.length > 6" class="buttons-container ">
        <button v-if="selectedGif.user?.display_name" @click="showAuthorCard" class="button">
          About Author
        </button>
        <button class="button btn-share" @click="shareContent">Share</button>
      </div>

      <div v-if="images.length > 6" class="image-slider">
        <div class="slider-images">
          <transition-group name="fade" tag="div" class="image-container">
            <div v-for="image in visibleImages" :key="image.id">
              <img class="img" :src="image?.images?.fixed_width?.webp" alt="Slide Image" @click="selectedGif = image" />
            </div>
          </transition-group>
        </div>
      </div>
      <div v-if="images.length > 6" class="buttons-container cover-buttons">
        <button @click="prev" class="slider-button">&lt;</button>
        <button @click="next" class="slider-button">&gt;</button>
      </div>
    </div>
  </div>

  <div v-if="isAuthorCard" class="author-container">
    <div class="cover-div"></div>
    <button class="close-button " @click="showAuthorCard">X</button>
    <div class="product-card card">
      <img src="/logo.png" alt="Logo" class="logo">
      <h3>About Author</h3>
      <img :src="selectedGif.user.avatar_url" alt="Avatar" class="author-image">
      <div class="product-info">
        <p class="product-text"><b>User name:</b> {{ selectedGif.user.username.toUpperCase() }}</p>
        <p class="product-text">
          <b>Author on Gifhy:</b>
          <a :href="selectedGif.user.profile_url" class="product-text" target="_blank" rel="noopener noreferrer">
            Link
          </a>
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.product-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: absolute;
  max-width: 360px;
  top: 20px;
  left: 50%;
  margin-left: -180px;
  padding: 5px;
  border: 2px solid #ffffff;
  background: linear-gradient(to top, #7a7a7a, #e5e6e6);
  border-radius: 10px;
}

.product-image {
  width: 360px;
  border-radius: 3px;
  object-fit: cover;
}

.product-info {
  margin-top: 0.5rem;
}

.product-text {
  margin-left: 10px;
  margin-right: 10px;
  text-align: justify;
  font-size: 14px;
  line-height: 1.2;
  margin-block-start: 0.5em;
}

h3 {
  margin-top: 35px;
  line-height: 1.2;
  color: #000;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.slider-images {
  height: 50px;
  margin-top: 10px;
  margin-bottom: 10px;
}

.image-slider {
  display: flex;
  flex-direction: row;
  height: 50px;
}

.image-container {
  display: flex;
  flex-direction: row;
  height: 50px;

}

.img {
  width: 50px;
  height: 50px;
  margin-left: 2px;
  border-radius: 3px;
  cursor: pointer;
}

.buttons-container {
  display: flex;
  flex-direction: row;
  margin-top: 10px;
}

.cover-buttons {
  gap: 270px;
}

.btn-share {
  margin-left: 10px;
}

.slider-button {
  padding: 5px;
  border-radius: 3px;
}

.close-button {
  position: fixed;
  top: 20px;
  left: 345px;
  z-index: 100;
  background-color: #800d0d;
}

a {
  color: #3d40e4;
  cursor: pointer;
}

.logo {
  border-radius: 5px;
}

.author-image {
  width: 360px;
  border-radius: 3px;
  object-fit: cover;
}
</style>