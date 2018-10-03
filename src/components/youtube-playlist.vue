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
    async getPlaylist(playlistId) {
      const result = [];

      async function request(playlistId, pageToken) {
        let base =
          "https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&maxResults=50&playlistId=" +
          playlistId +
          "&key=AIzaSyB-_brk224t6vZMfast60I0brymIdiU2q4";

        if (pageToken !== undefined) base += `&pageToken=${pageToken}`;

        const data = (await axios.get(base)).data;
        const items = data.items;
        const nextPage = data.nextPageToken;

        for (const i of items) {
          if (i.snippet.thumbnails === undefined) continue;

          const reDate = /(\d\d\d\d)-(\d\d)-(\d\d)/g.exec(
            i.snippet.publishedAt
          );

          const id = i.snippet.resourceId.videoId;
          const year = reDate[1];
          const month = reDate[2];
          const day = reDate[3];
          const formatDot = `${day}.${month}.${year}`;
          const formatSlash = `${day}/${month}/${year}`;

          const url = "https://www.youtube.com/watch?v=" + id;
          const title = i.snippet.title.replace(/"/g, '\\"');
          const description = i.snippet.description.split("\n\n")[0];
          const thumbnail = i.snippet.thumbnails.high.url;

          result.push({
            id,
            url,
            title,
            year,
            month,
            day,
            formatDot,
            formatSlash,
            description,
            thumbnail
          });
        }

        if (nextPage !== undefined) await request(playlistId, nextPage);
      }
      await request(playlistId);
      return result;
    }
  },
  async created() {
    this.items = await this.getPlaylist(this.id);
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
