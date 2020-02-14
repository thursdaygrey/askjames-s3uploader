<template>
  <div class="hello">
    <h1>S3 Uploader Test</h1>

    <div v-if="!image">
      <h2>Select an image</h2>
      <input type="file" @change="onFileChange">
    </div>
    <div v-else>
      <img :src="image" />
      <button v-if="!uploadURL" @click="removeImage">Remove file</button>
      <button v-if="!uploadURL" @click="uploadImage">Upload file</button>
    </div>
    <h2 v-if="uploadURL">Success! File uploaded to:</h2>
    <a :href="uploadURL">{{ uploadURL }}</a>
  </div>
</template>

<script>

import axios from 'axios'
const MAX_IMAGE_SIZE = 100000000

export default {
  name: 's3uploader',
  data () {
    return {
      image: '',
      uploadURL: ''
    }
  },
  methods: {
    onFileChange (e) {
      let files = e.target.files || e.dataTransfer.files
      if (!files.length) return
      this.createImage(files[0])
    },
    createImage (file) {
      // var image = new Image()
      let reader = new FileReader()

      reader.onload = (e) => {
        console.log('length: ', e.target.result.includes('data:image/jpeg'))
        if (e.target.result.includes('data:image/jpeg')) {
          return alert('Wrong file type - JPG only.')
        }
        if (e.target.result.length > MAX_IMAGE_SIZE) {
          return alert('File is loo large - 100Mb maximum')
        }

        this.image = e.target.result
      }
      reader.readAsDataURL(file)
    },
    removeImage: function (e) {
      console.log('Remove clicked')
      this.image = ''
    },
    uploadImage: async function (e) {
      console.log('Upload clicked')

      // Get the presigned URL
      const response = await axios({
        method: 'GET',
        url: `https://tr49ij83hd.execute-api.us-east-1.amazonaws.com/Prod/'
      })

      console.log('Response: ', response.data)
      console.log('Uploading: ', this.image)

      let binary = atob(this.image.split(',')[1])
      let array = []
      for (var i = 0; i < binary.length; i++) {
        array.push(binary.charCodeAt(i))
      }
      let blobData = new Blob([new Uint8Array(array)], {type: 'image/jpeg'})
      console.log('Uploading to: ', response.data.uploadURL)

      const result = await fetch(response.data.uploadURL, {
        method: 'PUT',
        body: blobData
      })

      console.log('Result: ', result)

      // Final URL for the user doesn't need the query string params
      this.uploadURL = response.data.uploadURL.split('?')[0]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#app {
  text-align: center;
}
img {
  width: 30%;
  margin: auto;
  display: block;
  margin-bottom: 10px;
}
button {
}
</style>
