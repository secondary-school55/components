<template>
  <div>
    <div
      id="gallery" 
      :style="style">
      <div 
        class="video"
        v-for="video in items" 
        :key="video.thumbnail"
        @click="click(video)"
        ref="video"
      >
        <img 
          class="thumbnail" 
          :src="video.thumbnail">
        <div 
          class="title" 
        >{{ video.title }}</div>
      </div>
    </div>
    <div 
      v-if="current"
      id="player" 
      @click="current = null">
      <iframe
        :src="current"
        frameborder="0"/>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "YoutubePlaylist",
  props: {
    id: {
      type: String,
      required: true
    },
    columns: {
      type: String,
      required: true
    },
    gap: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      items: [],
      current: null
    };
  },
  computed: {
    style() {
      return {
        gridTemplateColumns: `repeat(${this.columns}, 1fr)`,
        gridGap: this.gap,
        paddingBottom: this.gap
      };
    }
  },
  methods: {
    click(v) {
      this.current = `https://www.youtube.com/embed/${v.id}?rel=0`;
    },
  },
  async created() {
    this.items = (await axios.get(`https://api.school55.pp.ua/api/youtube/playlist/${this.id}`)).data;
  }
};
</script>

<style scoped>
#gallery {
  width: 100%;
  display: grid;
}

.video {
  width: 100%;
}

.video:hover {
  cursor: pointer;
}

.thumbnail {
  width: 100%;
}

.title {
  font-family: "Georgia", serif;
  text-align: center;
  font-size: 1.5vw;
}

#player {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  background-color: rgba(0, 0, 0, 0.9);
  justify-content: center;
  align-items: center;
}

#player iframe {
  width: 75%;
  height: 75%;
}
</style>
