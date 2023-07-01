<script setup>
const user = useSupabaseUser()
const client = useSupabaseAuthClient()


async function Logout() {
  try {
    const { error } = await client.auth.signOut()
    if(error) throw error
  } catch (error) {
    console.log(error);
  }
}

</script>

<template>
    <div>
      <header class="shadow-sm bg-white ">
        <nav class="container mx-auto p-4 flex justify-between">
        <NuxtLink to="/" class="font-bold">Home</NuxtLink>
          <ul class="flex items-center gap-4">
            <li><NuxtLink to="/">Home</NuxtLink></li>
            <li><NuxtLink to="/todos">Todos</NuxtLink></li>
            <li v-if="!user"><NuxtLink to="/login" class="btn">Login</NuxtLink></li>
            <li> <button @click="Logout" v-if="user" class="px-4 py-2 text-white bg-cyan-800 rounded-md">Log out </button></li>
          </ul>
        </nav>
      </header>
      <div class="container mx-auto p-4">
        <slot />
      </div>
    </div>
  </template>
  <style>
  .router-link-exact-active{
      color:#12b488;
  }
  </style>