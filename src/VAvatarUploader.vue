<template>
  <div>
    <div v-if="clickable">
      <v-avatar
        v-bind="avatar"
        @click="launch"
        style="cursor: pointer"
      >
        <img
          v-if="url && !status.loading"
          :src="url"
        />

        <slot
          v-if="!url && !status.loading"
          name="none"
        >
          <v-icon large>account_circle</v-icon>
        </slot>

        <slot
          v-if="status.loading"
          name="loading"
          :status="status"
        >
          <v-progress-circular
            :value="status.progress"
            :size="32"
            :width="2"
            color="primary"
          />
        </slot>
      </v-avatar>

      <input
        type="file"
        ref="file"
        name="file"
        @change="change($event.target.name, $event.target.files)"
        style="display: none"
      />
    </div>

    <div v-else>
      <v-avatar v-bind="avatar">
        <img :src="url" />
      </v-avatar>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    url: {
      type: String,
      required: true
    },

    request: {
      type: Function,
      required: true
    },

    rename: {
      type: Function,
      required: false,
      default: file => file.name
    },

    field: {
      type: String,
      required: false,
      default: 'file'
    },

    clickable: {
      type: Boolean,
      required: false,
      default: true
    },

    maxSize: {
      type: Number,
      required: false,
      default: 2048
    },

    // TODO: Consider just using $attrs instead
    avatar: {
      type: Object,
      required: false,
      default: () => ({})
    }
  },

  data: () => ({
    status: {
      loading: false,
      progress: 0
    }
  }),

  methods: {
    launch () {
      if (!this.status.loading) {
        this.$refs.file.click()
      } else {
        // TODO: Cancel
      }
    },

    change (field, file) {
      const { maxSize } = this
      const [ image ] = file

      if (file.length > 0) {
        const size = image.size / maxSize / maxSize

        if (!image.type.match('image.*')) {
          this.$emit('error-type')
        } else if (size > 1) {
          this.$emit('error-size')
        } else {
          this.upload(image)
        }
      }
    },

    async upload (file) {
      this.status.loading = true

      const form = new FormData()

      const config = {
        headers: {
          'Content-Type': 'multipart/form-data'
        },

        onUploadProgress: function (event) {
          if (event.lengthComputable) {
            this.status.progress = (event.loaded / event.total) * 100
          }
        }.bind(this)
      }

      form.append(this.field, file, this.rename(file))

      try {
        const upload = await this.request(form, config)

        this.$emit('success', upload)
      } catch (e) {
        this.$emit('failed', e)
      } finally {
        this.status.loading = false
        this.status.progress = 0
      }
    }
  }
}
</script>
