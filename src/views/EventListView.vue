<script setup>
import EventCard from '@/components/EventCard.vue'
import { ref, onMounted, computed, watchEffect } from 'vue'
import EventService from '@/services/EventService.js'
import { useRouter } from 'vue-router'

const props = defineProps(['page'])
const events = ref(null)
const page = computed(() => props.page)
const router = useRouter()

const totalEvents = ref(0) // Store the total events
const isNextPage = computed(() => {
  // Calculate totalPages, based on 2 per page
  const totalPages = Math.ceil(totalEvents.value / 2)

  // If current page is less than total pages, return true
  return page.value < totalPages
})

onMounted(() => {
  watchEffect(() => {
    events.value = null
    EventService.getEvents(2, page.value)
      .then((response) => {
        events.value = response.data
        // our response has total stored in the header.
        totalEvents.value = response.headers['x-total-count']
      })
      .catch((error) => {
        router.push({ name: 'NetworkError' })
      })
  })
})
</script>

<template>
  <h1>Events for Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        id="page-prev"
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
        >&#60; Previous</router-link
      >

      <router-link
        id="page-next"
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-if="isNextPage"
        >Next &#62;</router-link
      >
    </div>
  </div>
</template>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pagination {
  display: flex;
  width: 290px;
}
.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
