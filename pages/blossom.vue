<script setup>
import { NDKEvent } from "@nostr-dev-kit/ndk"
import { BlossomClient } from "blossom-client-sdk"
import { useNdkStore } from '~/stores/Ndk'
import { useUserStore } from '~/stores/User'

const NdkStore = useNdkStore()
const UserStore = useUserStore()

const Blobs = ref()
const BlossomClientRef = ref()

/**
 * Init BlossomClient.
 * @returns {Promise<void>}
 */
const initBlossomClient = async () => {
  BlossomClientRef.value = new BlossomClient('https://nstore.nostrver.se', UserStore.pubkey)
}

/**
 * List all blobs.
 *
 * @returns {Promise<BlobDescriptor[]>}
 */
const listBlobs = async () => {
  try {
    if (!UserStore.signedIn) {
      throw 'You need to login first'
    }
    return BlossomClientRef.value.listBlobs(UserStore.pubkey)
  } catch (e) {
    console.log(e)
  }
}

/**
 * Upload a selected file as a blob.
 *
 * @returns {Promise<void>}
 */
const uploadBlob = async () => {
  try {
    // Trigger input file element
    const inputFile = document.createElement('input')
    inputFile.type = 'file'
    inputFile.click()
    inputFile.onchange = async e => {
      const file = e.target.files[0]
      // Create and sign an upload event with NDK
      const uploadEvent = new NDKEvent(NdkStore.ndk)
      uploadEvent.kind = 24242
      uploadEvent.created_at = Math.floor(Date.now() / 1000)
      uploadEvent.pubkey = UserStore.pubkey
      uploadEvent.content = 'Upload ' + file.name
      uploadEvent.tags = [
        ['t', 'upload'],
        ['name', file.name],
        ['size', file.size]
      ]
      console.log(uploadEvent)
      const signed = await uploadEvent.sign()
      console.log(signed)
      // @todo work in progress uploading a blob
      //const uploadAuthEvent = await BlossomClientRef.value.getUploadAuth(file, 'https://nstore.nostrver.se', 'Upload ' + file.name)
      //console.log(uploadAuthEvent)
      // encode it using base64
      //const encodedAuthHeader = BlossomClientRef.value.encodeAuthorizationHeader(uploadAuthEvent);
      //await BlossomClientRef.value.uploadBlob(BlossomClientRef.value.server, file, auth)
      //await BlossomClientRef.value.uploadBlob(file)
      const res = await fetch(new URL("/upload", 'https://nstore.nostrver.se'), {
        method: "PUT",
        body: file,
        headers: { authorization: signed },
      });
      console.log(res)
      if (res.ok) {
        // @todo reload list with blobs after successful upload
      }
    }
  } catch (e) {
    console.log(e)
    alert(e)
  }
}

/**
 * Init function.
 */
onMounted(async () => {
  await NdkStore.initNdk()
  await NdkStore.ndk.connect()
  await initBlossomClient()
  Blobs.value = await listBlobs()
})

</script>

<template>
  <div>
    <p class="mb-4">
      This is a quick demo how to get and list files with <NuxtLink to="/event/naddr1qqxkymr0wdek7mfdv3exjan9qgszv6q4uryjzr06xfxxew34wwc5hmjfmfpqn229d72gfegsdn2q3fgrqsqqqa28e4v8zy">Blossom</NuxtLink>.
      <br />
      Using the node package <a href="https://github.com/hzrd149/blossom-client-sdk" target="_blank">https://github.com/hzrd149/blossom-client-sdk which</a> you can install with <code>npm install blosson-client-sdk</code>
      <br />
      Documentation: <a href="https://hzrd149.github.io/blossom-client-sdk/index.html" target="_blank">https://hzrd149.github.io/blossom-client-sdk/index.html</a>
    </p>
    <p v-if="BlossomClientRef">
      Showing blobs stored at <a :href="`${BlossomClientRef.server}`" target="_blank">{{ BlossomClientRef.server }}</a> from your pubkey <code>{{ BlossomClientRef.signer }}</code>:
    </p>
    <div v-if="Blobs">
      <div class="grid grid-cols-3">
        <div v-for="(blob) in Blobs">
          <a :href="`${ blob.url}`" target="_blank" v-if="blob.type.startsWith('image/')">
            <img :src="`${blob.url}`" class="w-64" :alt="`${blob.sha256}`" />
          </a>
          <a href="#" target="_blank" v-else>{{ blob.sha256 }}</a>
        </div>
      </div>
      <button @click="uploadBlob" class="block text-purple-500 mt-6 p-2 mx-auto bg-purple-100 hover:bg-purple-200">Click here to upload a file (works not yet)</button>
    </div>
    <div v-else>
      <NuxtLink to="/login" class="block text-purple-500 mt-6 p-2 mx-auto bg-purple-100 hover:bg-purple-200">
        You need to login first
      </NuxtLink>
    </div>
  </div>
</template>

<style scoped>

</style>