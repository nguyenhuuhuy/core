<template>
  <section id="songsWrapper">
    <h1 class="heading">
      <span>All Songs
        <controls-toggler :showing-controls="showingControls" @toggleControls="toggleControls"/>

        <span class="meta" v-show="meta.songCount">
          {{ meta.songCount | pluralize('song') }}
          •
          {{ meta.totalLength }}
        </span>
      </span>

      <song-list-controls
        v-show="state.songs.length && (!isPhone || showingControls)"
        @shuffleAll="shuffleAll"
        @shuffleSelected="shuffleSelected"
        :songs="state.songs"
        :config="songListControlConfig"
        :selectedSongs="selectedSongs"
      />
    </h1>

    <song-list :items="state.songs" type="all-songs"/>
  </section>
</template>

<script>
import { pluralize } from '@/utils'
import { songStore } from '@/stores'
import hasSongList from '@/mixins/has-song-list'

export default {
  mixins: [hasSongList],
  filters: { pluralize },

  data: () => ({
    state: songStore.state
  })
}
</script>
