<template>
  <div id="previewer">
    <div class="bar">
      <button @click="back" class="action" :title="$t('files.closePreview')" :aria-label="$t('files.closePreview')" id="close">
        <i class="material-icons">close</i>
      </button>

      <div class="title">
        <span>{{ this.name }}</span>
      </div>

      <rename-button :disabled="loading" v-if="user.perm.rename"></rename-button>
      <delete-button :disabled="loading" v-if="user.perm.delete"></delete-button>
      <download-button :disabled="loading" v-if="user.perm.download"></download-button>
      <info-button :disabled="loading"></info-button>
    </div>

    <div class="loading" v-if="loading">
      <div class="spinner">
        <div class="bounce1"></div>
        <div class="bounce2"></div>
        <div class="bounce3"></div>
      </div>
    </div>

    <button class="action" @click="prev" v-show="hasPrevious" :aria-label="$t('buttons.previous')" :title="$t('buttons.previous')">
      <i class="material-icons">chevron_left</i>
    </button>
    <button class="action" @click="next" v-show="hasNext" :aria-label="$t('buttons.next')" :title="$t('buttons.next')">
      <i class="material-icons">chevron_right</i>
    </button>

    <template v-if="!loading">
      <div class="preview">
        <ExtendedImage v-if="req.type == 'image'" :src="raw"></ExtendedImage>
        <audio v-else-if="req.type == 'audio'" :src="raw" autoplay controls></audio>
        <video v-else-if="req.type == 'video'" :src="raw" autoplay controls>
          <track
            kind="captions"
            v-for="(sub, index) in subtitles"
            :key="index"
            :src="sub"
            :label="'Subtitle ' + index" :default="index === 0">
          Sorry, your browser doesn't support embedded videos,
          but don't worry, you can <a :href="download">download it</a>
          and watch it with your favorite video player!
        </video>
        <object v-else-if="req.extension == '.pdf'" class="pdf" :data="raw"></object>
        <a v-else-if="req.type == 'blob'" :href="download">
          <h2 class="message">{{ $t('buttons.download') }} <i class="material-icons">file_download</i></h2>
        </a>
      </div>
    </template>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import url from '@/utils/url'
import { baseURL } from '@/utils/constants'
import { files as api } from '@/api'
import InfoButton from '@/components/buttons/Info'
import DeleteButton from '@/components/buttons/Delete'
import RenameButton from '@/components/buttons/Rename'
import DownloadButton from '@/components/buttons/Download'
import ExtendedImage from './ExtendedImage'

const mediaTypes = [
  "image",
  "video",
  "audio",
  "blob"
]

export default {
  name: 'preview',
  components: {
    InfoButton,
    DeleteButton,
    RenameButton,
    DownloadButton,
    ExtendedImage
  },
  data: function () {
    return {
      previousLink: '',
      nextLink: '',
      listing: null,
      name: '',
      subtitles: []
    }
  },
  computed: {
    ...mapState(['req', 'user', 'oldReq', 'jwt', 'loading']),
    hasPrevious () {
      return (this.previousLink !== '')
    },
    hasNext () {
      return (this.nextLink !== '')
    },
    download () {
      return `${baseURL}/api/raw${this.req.path}?auth=${this.jwt}`
    },
    previewUrl () {
      if (this.req.type === 'image') {
        return `${baseURL}/api/preview/big${this.req.path}?auth=${this.jwt}`
      }
      return `${baseURL}/api/raw${this.req.path}?auth=${this.jwt}`
    },
    raw () {
      return `${this.previewUrl}&inline=true`
    }
  },
  watch: {
    $route: function () {
      this.updatePreview()
    }
  },
  async mounted () {
    window.addEventListener('keyup', this.key)
    this.$store.commit('setPreviewMode', true)
    this.listing = this.oldReq.items
    this.updatePreview()
  },
  beforeDestroy () {
    window.removeEventListener('keyup', this.key)
    this.$store.commit('setPreviewMode', false)
  },
  methods: {
    back () {
      this.$store.commit('setPreviewMode', false)
      let uri = url.removeLastDir(this.$route.path) + '/'
      this.$router.push({ path: uri })
    },
    prev () {
      this.$router.push({ path: this.previousLink })
    },
    next () {
      this.$router.push({ path: this.nextLink })
    },
    key (event) {
      event.preventDefault()

      if (event.which === 13 || event.which === 39) { // right arrow
        if (this.hasNext) this.next()
      } else if (event.which === 37) { // left arrow
        if (this.hasPrevious) this.prev()
      }
    },
    async updatePreview () {
      if (this.req.subtitles) {
        this.subtitles = this.req.subtitles.map(sub => `${baseURL}/api/raw${sub}?auth=${this.jwt}&inline=true`)
      }

      let dirs = this.$route.fullPath.split("/")
      this.name = decodeURIComponent(dirs[dirs.length - 1])

      if (!this.listing) {
        try {
          const path = url.removeLastDir(this.$route.path)
          const res = await api.fetch(path)
          this.listing = res.items
        } catch (e) {
          this.$showError(e)
        }
      }

      this.previousLink = ''
      this.nextLink = ''

      for (let i = 0; i < this.listing.length; i++) {
        if (this.listing[i].name !== this.name) {
          continue
        }

        for (let j = i - 1; j >= 0; j--) {
          if (mediaTypes.includes(this.listing[j].type)) {
            this.previousLink = this.listing[j].url
            break
          }
        }

        for (let j = i + 1; j < this.listing.length; j++) {
          if (mediaTypes.includes(this.listing[j].type)) {
            this.nextLink = this.listing[j].url
            break
          }
        }

        return
      }
    }
  }
}
</script>
