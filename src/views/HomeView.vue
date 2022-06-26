<template>
  <v-container>
    <v-card>
      <v-card-header>
        <h1>Поиск изображений по тегам</h1>
      </v-card-header>
      <v-card-text>
        <div
          style="
            display: flex;
            flex-direction: row;
            gap: 10px;
            align-items: center;
          "
        >
          <v-select
            :items="['Строгий (И)', 'Широкий (ИЛИ)']"
            v-model="mode"
            hide-details
            style="max-width: 200px"
            label="Режим поиска"
          ></v-select>
          <v-autocomplete
            label="Теги"
            :items="tags"
            v-model="reqTags"
            small-chips
            title="Введите запрос"
            multiple
            hide-details
          ></v-autocomplete>
          <v-btn xl append-icon="mdi-magnify" @click="loadImages()"
            >поиск</v-btn
          >
        </div>
      </v-card-text>
      <v-card-text v-if="images.length != 0">
        <v-row>
          <v-col
            v-for="n in images"
            :key="n"
            class="d-flex child-flex"
            cols="4"
          >
            <v-img
              :src="n.url"
              :lazy-src="`https://picsum.photos/10/6?image=${n * 5 + 10}`"
              aspect-ratio="1"
              class="bg-grey-lighten-2"
            >
              <template v-slot:placeholder>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular
                    indeterminate
                    color="grey-lighten-5"
                  ></v-progress-circular>
                </v-row>
              </template>
            </v-img>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-text v-if="notfound">
        <h2>По вашему запросу ничего не нашлось :(</h2>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
  import axios from "axios";
  export default {
    data() {
      return {
        tags: [],
        reqTags: [],
        images: [],
        mode: "Широкий (ИЛИ)",
        notfound: false,
      };
    },
    async mounted() {
      const response = await axios.get(`${import.meta.env.VITE_URL}/tags/`);
      if (response.status == 200) {
        this.tags = response.data;
      }
    },
    methods: {
      async loadImages() {
        this.images = await (
          await axios.get(`${import.meta.env.VITE_URL}/image`, {
            params: { tags: this.reqTags.toString(), mode: this.whatMode() },
          })
        ).data.images;
        if (this.images.length == 0) {
          this.notfound = true;
        } else {
          this.notfound = false;
        }
      },
      whatMode() {
        if (this.mode == "Широкий (ИЛИ)") {
          return "or";
        } else {
          return "and";
        }
      },
    },
  };
</script>
