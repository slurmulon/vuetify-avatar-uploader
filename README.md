# vuetify-avatar-uploader
> :cloud: `v-avatar` + file uploads
---

Wraps Vuetify's [`v-avatar`](https://vuetifyjs.com/en/components/avatars) component with file upload magic.

## Installation

```sh
npm i vuetify-avatar-uploader
```

### Module

```js

import VAvatarUploader from 'vuetify-avatar-uploader'

export default {
  components: {
    'v-avatar-uploader': AvatarUploader
  }
}
```

### Browser

```html
<script type="text/javascript" src="node_modules/vuejs/dist/vue.min.js"></script>
<script type="text/javascript" src="node_modules/vuetify-upload-button/dist/upload-button.min.js"></script>
<script type="text/javascript">
  Vue.use(VAvatarUploader);
</script>
```

## Usage

At a minimum, you must provide `:url` and `:request` properties:

```vue
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

It's also recommended to specify the `:clickable` property, as this determines when users should be able to click on the avatar to perform file uploads.

More details on the supported properties follows.


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
`error-type`|File upload exceeds maximum size

### Slots

 - `none`
 - `loading`

### Future

 - [ ] Provide configuration options for `v-progress-bar`
 - [ ] Handle upload cancellations
 - [ ] Allow custom supported MIME types

## License

MIT
