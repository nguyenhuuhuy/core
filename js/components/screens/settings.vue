<template>
  <section id="settingsWrapper">
    <h1 class="heading">
      <span>Settings</span>
    </h1>

    <form @submit.prevent="confirmThenSave" class="main-scroll-wrap">
      <div class="form-row">
        <label for="inputSettingsPath">Media Path</label>

        <p class="help" id="mediaPathHelp">
          The <em>absolute</em> path to the server directory containing your media.
          Koel will scan this directory for songs and extract any available information.<br>
          Scanning may take a while, especially if you have a lot of songs, so be patient.
        </p>

        <input
          type="text"
          v-model="state.settings.media_path"
          id="inputSettingsPath"
          aria-describedby="mediaPathHelp"
        >
      </div>

      <div class="form-row">
        <button type="submit">Scan</button>
      </div>
    </form>
  </section>
</template>

<script>
import { settingStore, sharedStore } from '@/stores'
import { parseValidationError, forceReloadWindow, showOverlay, hideOverlay, alerts } from '@/utils'
import router from '@/router'

export default {
  data () {
    return {
      state: settingStore.state,
      sharedState: sharedStore.state
    }
  },

  computed: {
    shouldWarn () {
      // Warn the user if the media path is not empty and about to change.
      return this.sharedState.originalMediaPath &&
        this.sharedState.originalMediaPath !== this.state.settings.media_path.trim()
    }
  },

  methods: {
    confirmThenSave () {
      if (this.shouldWarn) {
        alerts.confirm('Warning: Changing the media path will essentially remove all existing data – songs, artists, \
          albums, favorites, everything – and empty your playlists! Sure you want to proceed?', this.save)
      } else {
        this.save()
      }
    },

    save: async () => {
      showOverlay()

      try {
        await settingStore.update()
        // Make sure we're back to home first.
        router.go('home')
        forceReloadWindow()
      } catch (err) {
        let msg = 'Unknown error.'

        if (err.response.status === 422) {
          msg = parseValidationError(err.response.data)[0]
        }

        hideOverlay()
        alerts.error(msg)
      }
    }
  }
}
</script>

<style lang="scss">
@import "~#/partials/_vars.scss";

#settingsWrapper {
  input[type="text"] {
    width: 384px;
    margin-top: 12px;
  }

  @media only screen and (max-width : 667px) {
    input[type="text"] {
      width: 100%;
    }
  }
}
</style>
