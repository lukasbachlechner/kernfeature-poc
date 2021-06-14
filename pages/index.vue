<template>
  <div class="container">
    <div>
      <Logo />
      <h1 class="text-3xl my-8">Offline Audio Storage – Proof of Concept</h1>
      <p>
        Making audio files (podcast episodes) available when the user is
        offline.
      </p>
      <nuxt-link
        to="/episodes"
        class="px-6 py-2 border rounded mt-8 inline-block"
        >To episodes</nuxt-link
      >
    </div>
  </div>
</template>

<script>
export default {}
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
@apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
