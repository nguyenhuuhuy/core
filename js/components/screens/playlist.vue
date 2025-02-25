<template>
  <section id="playlistWrapper">
    <template v-if="playlist.populated">
      <h1 class="heading">
        <span>{{ playlist.name }}
          <controls-toggler :showing-controls="showingControls" @toggleControls="toggleControls"/>

          <span class="meta" v-show="meta.songCount">
            {{ meta.songCount | pluralize('song') }}
            •
            {{ meta.totalLength }}
            <template v-if="sharedState.allowDownload && playlist.songs.length">
              •
              <a href @click.prevent="download" title="Download all songs in playlist" role="button">
                Download All
              </a>
            </template>
          </span>
        </span>

        <song-list-controls
          v-show="!isPhone || showingControls"
          @shuffleAll="shuffleAll"
          @shuffleSelected="shuffleSelected"
          @deletePlaylist="destroy"
          :songs = "playlist.songs"
          :config="songListControlConfig"
          :selectedSongs="selectedSongs"
        />
      </h1>

      <song-list v-show="playlist.songs.length"
        :items="playlist.songs"
        :playlist="playlist"
        type="playlist"
        ref="songList"
      />

      <div v-if="!playlist.songs.length" class="none">
        The playlist is currently empty. You can fill it up by dragging songs into its name in the sidebar,
        or use the &quot;Add To…&quot; button.
      </div>
    </template>
  </section>
</template>

<script>
import { pluralize, event } from '@/utils'
import { playlistStore, sharedStore } from '@/stores'
import { playback, download } from '@/services'
import { views } from '@/config'
import hasSongList from '@/mixins/has-song-list'

export default {
  mixins: [hasSongList],
  filters: { pluralize },

  data: () => ({
    playlist: playlistStore.stub,
    sharedState: sharedStore.state,
    songListControlConfig: {
      deletePlaylist: true
    }
  }),

  created () {
    /**
     * Listen to 'main-content-view:load' event to load the requested
     * playlist into view if applicable.
     */
    event.on(event.$names.LOAD_MAIN_CONTENT, (view, playlist) => {
      if (view !== views.PLAYLIST) {
        return
      }

      playlist.populated ? (this.playlist = playlist) : this.populate(playlist)
    })
  },

  methods: {
    /**
     * Shuffle the songs in the current playlist.
     * Overriding the mixin.
     */
    shuffleAll () {
      playback.queueAndPlay(this.playlist.songs, true /* shuffled */)
    },

    destroy () {
      event.emit(event.$names.PLAYLIST_DELETE, this.playlist)
    },

    download () {
      return download.fromPlaylist(this.playlist)
    },

    /**
     * Fetch a playlist's content from the server, populate it, and use it afterwards.
     */
    async populate (playlist) {
      await playlistStore.fetchSongs(playlist)
      playlist.populated = true
      this.playlist = playlist
      this.$nextTick(() => this.$refs.songList && this.$refs.songList.sort())
    }
  }
}
</script>

<style lang="scss">
@import "~#/partials/_vars.scss";

#playlistWrapper {
  button.play-shuffle, button.del {
    i {
      margin-right: 0 !important;
    }
  }

  .none {
    color: $color2ndText;
    padding: 16px 24px;

    a {
      color: $colorHighlight;
    }
  }
}
</style>
