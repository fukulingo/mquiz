<template>
  <v-container class="pb-0 container">
    <v-layout>
      <v-text-field
        v-model="term"
        v-on:keyup.enter="search(term)"
        @change="search(term)"
        placeholder="スピッツ"
        outlined />
      <v-btn
        depressed
        large
        @click="search(term)"
      >
        検索
      </v-btn>
    </v-layout>
    <div
      v-if="count > 0"
      key="results-count-exists"
      class="results"
    >
      <li
        v-for="n in results"
        :key="n.artistId"
        class="results__list"
      >
        <nuxt-link
          :to="`/artists/${n.artistId}`"
          class="results__list_link"
        >
          {{ n.artistName}}
        </nuxt-link>
      </li>
    </div>
    <p
      v-if="count === 0"
      key="results-count-zero"
    >
      "{{ term }}"に該当するアーティストが見つかりませんでした
    </p>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      term: '',
      results: '',
      count: false
    };
  },
  methods: {
    search(term) {
      axios
        .get(
          `http://itunes.apple.com/search?lang=ja_jp&country=JP&term=${term}&media=music&entity=musicArtist&attribute=artistTerm&attribute=artistTerm&limit=20`
        )
        .then(response => {
          this.results = response.data.results
          this.count = response.data.resultCount
        })
    }
  }
};
</script>

<style scoped>
.container {
  position: relative;
}
.results {
  position: absolute;
  padding: 10px;
  width: 80%;
  background-color: white;
}
.results__list {
  list-style: none;
}
.results__list a {
  text-decoration: none;
}
</style>
