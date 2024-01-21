<template>
  <div v-if="city" class="h-screen relative overflow-hidden">
    <img :src="background" class="w-full h-full"/>
    <div class="absolute w-full h-full top-0 overlay" />
    <div class="absolute w-full h-full top-0 p-48">
      <div class="flex justify-between">
        <div class="text-7xl text-white">
          <div class="flex">
            <h1>{{ city?.name }}</h1>
            <p class="text-xl mt-auto">,{{ city?.sys?.country }}</p>
          </div>
          <p class="font-extralight text-2xl mt-2 text-white">
            {{ today }}
          </p>
          <img v-if="city?.weather" :src="`https://openweathermap.org/img/wn/${city?.weather[0]?.icon}@4x.png`" class="w-56 icon" />
        </div>
        <div>
          <p class="text-9xl text-white font-extralight">{{ city?.main?.temp }}°C</p>
        </div>
      </div>
      <div class="mt-40 flex justify-center">
        <input v-model="input" type="text" class="w-1/2 h-[3rem] rounded-lg" placeholder="Search a City...." />
        <button @click="handleClick" class="bg-sky-400 text-white rounded-lg mx-3 w-[10rem]">
          Search
        </button>
      </div>
    </div>
  </div>
  <div v-else class="h-screen flex">
    <div class="mx-auto my-auto">
      <h1 class="text-7xl">
        Oops!... We can't find that city
      </h1>
      <div class="flex justify-center mt-[5rem]">
        <button @click="goBack" class="bg-sky-900 text-[#fff] my-auto text-3xl rounded py-2 px-[5rem] hover:bg-sky-700">
          Go Back
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const cookie = useCookie('city')
const search = ref(cookie.value ?? 'cairo')
const input = ref('')
const background = ref('')
const config = useRuntimeConfig()
// const { data: city, error} = useFetch(() => `https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=metric&appid=e866c5f2eaf872354975c24af5e6243a`)

const { data:city, error, refresh } = useAsyncData('city', async () => {
  let response

  try {
    response = await $fetch(`https://api.openweathermap.org/data/2.5/weather`, {
      params: {
        q: search.value,
        units: 'metric',
        appid: config.public.WEATHER_APP_SECRET
      }
    })
    cookie.value = search.value
    const temp = response?.main?.temp
    if(temp <= -10) {
      background.value = 'https://i.pinimg.com/736x/ba/ff/e3/baffe34f42af0aea6ab134768eebe79f.jpg'
    } else if(temp <= 0) {
      background.value = 'https://timelinecovers.pro/facebook-cover/download/orange-foliage-autumn-in-the-park-facebook-cover.jpg'
    } else if(temp <= 10) {
      background.value = 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQXWyxwda9FgTPQTuCpeBWqYbnGKhv0W8kY4Q&usqp=CAU'
    } else {
      background.value = 'https://timelinecovers.pro/facebook-cover/download/spring-facebook-cover.jpg'
    }
  } catch (e) {

  } finally {
    return response
  }
}, {
  watch: [search]
})

const handleClick = () => {
  const formattedSearch = input.value.trim().split(' ').join('+')
  search.value = formattedSearch
  input.value = ''
  //refresh()
}

const goBack = () => {
  search.value = cookie.value ?? 'cairo'
}

const today = new Date().toLocaleDateString("en-US", {
  weekday: "short",
  year: "numeric",
  month: "short",
  day: 'numeric'
})
</script>

<style scoped>
.overlay {
  background-color: rgb(0, 0, 0, 0.5);
}
.icon {
  margin-left: -2.75rem;
  margin-top: -2.75rem;
}
</style>
