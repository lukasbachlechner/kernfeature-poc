<template>
  <div class="w-1/3 mx-auto mb-64">
    <div class="flex justify-between items-center">
      <h1>Episodes</h1>
      <nuxt-link to="/">Back to home</nuxt-link>
    </div>
    <p>
      Only the green item will work, as the audio url is hardcoded (this
      particular podcast API doesn't allow direct access to audio files with
      axios)
    </p>
    <section>
      <h2>From API</h2>
      <div v-show="$nuxt.isOffline">
        <p>Episodes can't be fetched - you are offline.</p>
      </div>
      <ul v-show="$nuxt.isOnline" class="grid">
        <li
          v-for="episode in episodes"
          :key="episode.id"
          class="p-4 border flex justify-between items-center mb-2"
          :class="{ 'bg-green-200': episode.id === idForAudio }"
        >
          {{ episode.title }}

          <button
            class="bg-blue-700 text-white rounded px-6 py-2"
            @click="savePodcast(episode)"
          >
            Save
          </button>
        </li>
      </ul>
    </section>

    <section>
      <h2>Saved</h2>
      <ul v-show="savedEpisodes.length > 0" class="grid">
        <li
          v-for="episode in savedEpisodes"
          :key="episode.id"
          class="p-4 border flex justify-between items-center"
          :class="{ 'bg-green-200': episode.id === idForAudio }"
        >
          {{ episode.title }}

          <button
            class="bg-blue-700 text-white rounded px-6 py-2"
            @click="playSaved(episode.id)"
          >
            Play
          </button>
        </li>
      </ul>
      <div v-show="savedEpisodes.length === 0">
        <p>No saved episodes.</p>
      </div>
    </section>

    <div class="fixed left-0 right-0 bottom-0 bg-gray-400 p-4">
      <audio ref="audioPlayer" controls class="w-full"></audio>
    </div>
  </div>
</template>

<script>
import { Client } from 'podcast-api'

export default {
  data: () => ({
    episodes: [],
    savedEpisodes: [],
    idForAudio: 'ef567102a2fa4ac6940e24da7249468f',
  }),
  fetchOnServer: false,
  async fetch() {
    if (this.$nuxt.isOnline) {
      try {
        const client = Client({
          apiKey: this.$config.apiKey,
        })

        const { data } = await client.fetchPodcastById({
          id: 'd2f3d50198a044039d9c3bc16407154f',
        })

        this.episodes = data.episodes
      } catch (e) {
        console.log(e)
      }
    }

    const keys = await this.$localForage.savedEpisodeData.keys()
    const promisedItems = keys.map((key) =>
      this.$localForage.savedEpisodeData.getItem(key)
    )

    this.savedEpisodes = await Promise.all(promisedItems)
  },
  methods: {
    async savePodcast(episode) {
      await this.$localForage.savedEpisodeData.setItem(episode.id, episode)

      const audioFile =
        'https://adn.podigee.com/adswizz/media/podcast_16544_baywatch_berlin_episode_454641_kafigfight_der_murmelpromis.mp3?awCollectionId=svo_285a8b&awEpisodeId=454641&source=feed&v=1621538476'

      const audioBlob = await this.$axios.$get(audioFile, {
        responseType: 'blob',
      })

      await this.$localForage.savedAudioFiles.setItem(episode.id, audioBlob)

      this.savedEpisodes.push(episode)
    },

    async playSaved(episodeId) {
      const audioBlob = await this.$localForage.savedAudioFiles.getItem(
        episodeId
      )

      if (audioBlob) {
        const url = URL.createObjectURL(audioBlob)

        this.$refs.audioPlayer.src = url
        this.$refs.audioPlayer.play()
      } else {
        alert('No blob found for episode ' + episodeId)
      }
    },
  },
}
</script>

<style scoped>
h1 {
  @apply text-3xl font-bold my-4;
}
h2 {
  @apply text-xl font-bold my-4;
}
</style>
