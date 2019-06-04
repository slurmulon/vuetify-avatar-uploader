# vuetify-avatar-uploader
> :cloud: `v-avatar` + file uploads
---

Wraps Vuetify's [`v-avatar`](https://vuetifyjs.com/en/components/avatars) component with file upload magic.

1. User clicks on their avatar
2. User selects an image file to use as their new avatar
3. File is uploaded to your API
4. User's avatar is replaced :sparkles:

## Installation

```sh
npm i slurmulon/vuetify-avatar-uploader
```

### Module

```js

import VAvatarUploader from 'vuetify-avatar-uploader'

export default {
  components: {
    VAvatarUploader
  }
}
```

### Browser

```html
<script type="text/javascript" src="node_modules/vuejs/dist/vue.min.js"></script>
<script type="text/javascript" src="node_modules/vuetify/dist/vuetify.min.js"></script>
<script type="text/javascript" src="node_modules/vuetify-upload-button/dist/upload-button.min.js"></script>
<script type="text/javascript">
  Vue.use(VAvatarUploader);
</script>
```

## Usage

At a minimum, you must provide `:url` and `:request` properties:

```html
<template>
  <v-avatar-uploader
    :url="url"
    :request="request"
  />
</template>

<script>
import VAvatarUploader from 'vuetify-avatar-uploader'

export default {
  computed: {
    // Determine the URL to show, typically from an object representing a user
    url () {
      return 'https://randomuser.me/api/portraits/men/1.jpg'
    }
  },

  methods: {
    // Responsible for performing the upload request to your API
    request (form, config) {
      return this.$http.post('/v1/avatars', form, config)
    }
  },

  components: {
    VAvatarUploader
  }
}
</script>
```

## Example

The following is a real-world example of how the library can be used to support user avatar uploads:

```html
<template>
  <v-avatar-uploader
    :url="url"
    :request="request"
    :clickable="clickable"
    :rename="rename"
    @success="success"
    @failed="failed"
  />
</template>

<script>
import VAvatarUploader from 'vuetify-avatar-uploader'
import events from '@/util/events'

import uuid from 'uuid/v4'
import get from 'dlv'
import { mapState } from 'vuex'

export default {
  name: 'user-avatar-uploader',

  props: {
    user: {
      type: Object,
      required: true
    }
  },

  computed: {
    ...mapState('api/user', ['session']),

    url () {
      return get(this.user, 'avatar.thumb')
    },

    clickable () {
      return this.user.id === this.session.id
    }
  },

  methods: {
    request (form, config) {
      return this.$http.post('/v1/avatars', form, config)
    },

    rename (file) {
      const ext = file.name.split('.')[1]
      const name = `${uuid()}.${ext}`

      return name
    },

    success ({ data }) {
      // Update user avatar with the latest
      Object.assign(this.user.avatar, data)

      // Request the latest user object from the API to update all other references in the app
      this.$store.dispatch('api/user/get')
    },

    failed (error) {
      events.$emit('notify', { text: 'Failed to upload avatar', kind: 'error', icon: 'warning' })
    }
  },

  components: {
    VAvatarUploader
  }
}
</script>
```

## API

### Props

**Name**|**Description**|**Type**|**Required**|**Default**
-----|-----|-----|-----|-----
url|URL of the avatar|`String`|Yes| 
`request`|Performs the file upload|`Function`|Yes| 
`rename`|Renames the file before upload|`Function`|No|`file => file.name`
`clickable`|Determines if the user can click to upload|`Boolean`|No|`true`
`maxSize`|Maximum file upload size (in bytes)|`Number`|No|`2048`
`avatar`|Core `v-avatar` configuration object|`Object`|No|`{}`

### Events

**Name**|**Description**
-----|-----
`success`|File upload succeeded
`failed`|File upload failed
`error-type`|File upload MIME type is unsupported
`error-size`|File upload exceeds maximum size

### Slots

 - `none`
 - `loading`

### Future

 - [ ] Provide configuration options for `v-progress-bar`
 - [ ] Handle upload cancellations
 - [ ] Allow custom supported MIME types

## License

MIT
