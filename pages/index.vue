<script setup>
const jwt = ref('')
const token = ref('')
const books = ref([])

async function getJwtToken() {
  const response = await $fetch('http://localhost/api/user/token-auth/', {
    method: 'POST',
    body: {
      username: 'baserowtest@gmail.com',
      password: 'securepassword',
    },
  })
  jwt.value = response.token
}

async function groups() {
  const response = await $fetch('http://localhost/api/groups/', {
    headers: {
      Authorization: `JWT ${jwt.value}`,
    },
  })
  console.log('current group to use', response[0].users[0].group)
}

async function getToken() {
  const response = await $fetch('http://localhost/api/database/tokens/', {
    method: 'POST',
    body: {
      name: 'testing-purposes', // ? not really that important
      group: 74, // ? fetched from above
    },
    headers: {
      Authorization: `JWT ${jwt.value}`,
    },
  })
  token.value = response.key
}

async function fetchBooks() {
  const response = await $fetch('http://localhost/api/database/rows/table/386/?user_field_names=true', {
    headers: {
      Authorization: `Token ${token.value}`,
    },
  })
  books.value = response
}

await getJwtToken()
await groups()
await getToken()
await fetchBooks()
</script>

<template>
  <main class="max-w-3xl mx-auto pt-6">
    <h1 class="text-center text-4xl bg-clip-text bg-gradient-to-r text-transparent from-emerald-600 to-sky-500 mb-6">
      My book collection
    </h1>
    <section v-if="Object.values(books).length" class="grid grid-cols-1 gap-6 md:grid-cols-2 grid-flow-dense">
      <div
        v-for="book in books.results"
        :key="book.id"
        class="relative flex justify-between flex-col bg-slate-800 rounded-xl py-6"
      >
        <div v-show="book['In possession?']" class="absolute right-4 top-4 text-emerald-500 flex justify-between">
          <span class="text-md mr-2">Owned</span>
          <icon class="text-xl" name="pajamas-check-circle-filled" />
        </div>
        <img :src="book['Cover photo'][0].url" class="h-48 object-contain w-full" />
        <p class="text-center text-slate-200 text-xl mt-2">{{ book['Book title'] }}</p>
      </div>
    </section>
  </main>
</template>
