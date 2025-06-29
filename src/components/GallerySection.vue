<template>
  <div class="gallery-section" :id="sectionId">
    <h2 class="section-title">{{ title }}</h2>
    <div class="gallery" id="gallery-{{ sectionId }}" ref="gallery">
      <a 
        v-for="(photo, index) in photos" 
        :key="`${sectionId}-${index}`"
        :href="`${publicPath}${photo.src}`" 
        :data-pswp-width="photo.width" 
        :data-pswp-height="photo.height"
        :data-pswp-index="`${sectionId}-${index}`"
        :class="['photo-container', photo.width > photo.height ? 'horizontal' : 'vertical']"
      >
        <img :src="`${publicPath}${photo.src}`" :alt="photo.alt" />
        <span v-if="showNames" class="photo-filename">{{ photo.src.split('/').pop() }}</span>
        <span class="photo-author">{{ photo.author }}</span>
      </a>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';

export default {
  name: 'GallerySection',
  props: {
    title: {
      type: String,
      required: true
    },
    sectionId: {
      type: String,
      required: true
    },
    photoDirectory: {
      type: String,
      required: true
    },
    photos: {
      type: Array,
      required: true
    },
    initPhotoSwipe: {
      type: Boolean,
      default: true
    }
  },
  setup(props) {
    const publicPath = process.env.BASE_URL || '/';
    const showNames = ref(false);

    onMounted(async () => {
      // Check if show-names parameter exists in URL
      const urlParams = new URLSearchParams(window.location.search);
      showNames.value = urlParams.has('show-names');

      // Only initialize PhotoSwipe if initPhotoSwipe prop is true
      if (props.initPhotoSwipe) {
        const PhotoSwipeLightbox = (await import('photoswipe/lightbox')).default;
        const lightbox = new PhotoSwipeLightbox({
          gallery: `#gallery-${props.sectionId}`,
          children: 'a',
          pswpModule: () => import('photoswipe')
        });
        lightbox.init();
      }
    });

    return {
      publicPath,
      showNames,
    };
  }
}
</script>

<style scoped>
.gallery-section {
  margin-bottom: 50px;
}

.section-title {
  color: #ffffff;
  margin-bottom: 20px;
  font-weight: 300;
  letter-spacing: 1px;
  text-align: center;
}

.gallery {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: auto;
  gap: 10px;
  padding: 0;
  max-width: 1200px;
  margin: 0 auto;
  background-color: transparent;
}

.gallery a {
  overflow: hidden;
  border-radius: 4px;
  transition: all 0.3s ease;
  position: relative;
  padding: 0;
  margin: 0;
  border: 3px solid white;
}

.gallery a:hover {
  border: 3px solid #fff;
  transform: scale(1.02);
}

.gallery a img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.5s ease;
}

.gallery a:hover img {
  transform: scale(1.05);
}

.photo-filename {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  font-weight: bold;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 10;
}

.gallery a:hover .photo-filename,
.gallery a:hover .photo-author {
  opacity: 1;
}

.photo-author {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  font-weight: bold;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 10;
}

.photo-container {
  display: flex;
  flex-direction: column;
  position: relative;
  transition: transform 0.3s ease; /* Smooth transition for hover effects */
  will-change: transform; /* Optimize for animations */
}

.horizontal {
  grid-column: span 2;
}

.vertical {
  grid-column: span 1;
  /* Make vertical images taller to maintain proportions */
  grid-row: span 2;
}

/* Center horizontal images that are alone on their line */
.horizontal:last-child:nth-child(4n-1),
.horizontal:last-child:nth-child(4n-2) {
  grid-column: 2 / span 2; /* Center in the middle 2 columns */
  justify-self: center;
}

/* Responsive adjustments */
@media (max-width: 992px) {
  .gallery {
    grid-template-columns: repeat(3, 1fr);
  }

  .horizontal {
    grid-column: span 2;
  }

  .vertical {
    grid-column: span 1;
  }

  /* Center horizontal images that are alone on their line in 3-column grid */
  .horizontal:last-child:nth-child(3n-1) {
    grid-column: 1 / span 2; /* Center in the first 2 columns */
    justify-self: center;
  }
}



@media (max-width: 768px) {
  .gallery {
    grid-template-columns: 1fr;
    /* Force flex layout for all sections on mobile to ensure single column */
    display: flex !important;
    flex-direction: column !important;
  }

  .horizontal, .vertical {
    grid-column: span 1 !important;
    grid-row: auto !important;
    width: 100% !important;
  }

  .gallery a img {
    width: 100% !important;
    height: auto !important;
    max-width: 100% !important;
  }

  /* Ensure no images are displayed side by side */
  .photo-container {
    width: 100% !important;
    max-width: 100% !important;
  }

  /* Add padding at the bottom to ensure last photos are visible above the side menu */
  .gallery-section:last-child {
    padding-bottom: 80px; /* Ensure enough space for the side menu */
  }
}
</style>
