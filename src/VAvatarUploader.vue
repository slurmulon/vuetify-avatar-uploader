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
        <img
          v-if="url"
          :src="url"
        />

        <slot
          v-else
          name="none"
        >
          <v-icon large>account_circle</v-icon>
        </slot>
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
      const [ image ] = file
      const { maxSize } = this

      if (file.length > 0) {
        const size = image.size / 1024

        if (!image.type.match('image.*')) {
          this.$emit('error-type')
        } else if (size > maxSize) {
          this.$emit('error-size')
        } else {
          this.upload(image)
        }
      } else {
        this.$emit('error-empty')
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
            const progress = (event.loaded / event.total) * 100

            this.status.progress = progress

            this.$emit('progress', progress)
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
