<script setup>
import { ref, computed, watch, reactive } from 'vue';
import weatherLocations from '@/weatherLocations';

const {firingTimes, firingProgramDocumentUrl, activeLayers} = defineProps({
  firingTimes: {
      type: [null, Array],
      required: true
  },
  firingProgramDocumentUrl: {
      type: [null, String],
  },
  activeLayers: {
    type: Object,
  }
});

const emit = defineEmits(['selectedFiringTimeChanged', 'activeLayersChanged']);

const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
const months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

const todaysDate = new Date(new Date().setHours(0,0,0,0));
const currentSelectedFiringTime = ref(null);
let currentSelectedFiringTimeIndex = null;


const currentSelectedFiringTimeDateFormatted = computed( () => {
  if (currentSelectedFiringTime.value === null) {
    return 'Loading';
  }

  if (currentSelectedFiringTime.value.date.getTime() === todaysDate.getTime()) {
    return 'Today';
  }

  return  days[currentSelectedFiringTime.value.date.getDay()]
        + ' ' + currentSelectedFiringTime.value.date.getDate()
        + ' ' + months[currentSelectedFiringTime.value.date.getMonth()];
});

watch(
  () => firingTimes,
  () => {

    if (currentSelectedFiringTime.value !== null) {
      console.warn('Selected date is already set');
      return;
    }

    const foundIndex = findFiringTimesIndexFromDate(todaysDate);

    if (foundIndex === -1) {
      console.error('Unable to find today\'s date within the firing times')
      return;
    }

    currentSelectedFiringTimeIndex = foundIndex;
    setCurrentSelectedFiringTimeByIndex(currentSelectedFiringTimeIndex);
  }
);

/**
 * Firing Areas
 */

function findFiringTimesIndexFromDate(date) {
  return firingTimes.findIndex((firingTime) => {
    return new Date(firingTime.date).getTime() === date.getTime();
  })
}

function handleTimelineMinusButtonClick() {
  if (currentSelectedFiringTimeIndex == null) {
    console.warn('Cannot time travel without knowing where to start');
    return;
  }

  if (currentSelectedFiringTimeIndex - 1 < 0) {
    console.warn('reached the start, cannot move backwards any more');
    return;
  }

  setCurrentSelectedFiringTimeByIndex(--currentSelectedFiringTimeIndex);
}

function handleTimelinePlusButtonClick() {
  if (currentSelectedFiringTimeIndex == null) {
    console.warn('Cannot time travel without knowing where to start');
    return;
  }

  if (currentSelectedFiringTimeIndex + 1 == firingTimes.length) {
    console.warn('reached the end, there is nothing past this point');
    return;
  }

  setCurrentSelectedFiringTimeByIndex(++currentSelectedFiringTimeIndex);
}

function setCurrentSelectedFiringTimeByIndex(index) {
  currentSelectedFiringTime.value = firingTimes[index];
  emit('selectedFiringTimeChanged', currentSelectedFiringTime);
}

</script>

<template>
  <div>
    <h1 class="text-3xl font-extrabold p-1 text-center mb-6">Dartmoor Hiking and Camping Information</h1>

    <h3 class="text-xl font-weight-bold mb-2">Firing Range Times</h3>

    <div class="grid grid-cols-3 justify-center text-center mb-4 h-16">
      <button class="bg-blue-200 hover:cursor-pointer" @click="handleTimelineMinusButtonClick"><</button>
      <span class="inline-block align-middle h-full">{{ currentSelectedFiringTimeDateFormatted }}</span>
      <button class="bg-blue-200 hover:cursor-pointer" @click="handleTimelinePlusButtonClick">></button>
    </div>

    <!-- Range Firing -->
    <div class="p-2 flex flex-col mb-4" v-if="currentSelectedFiringTime">
      <div v-for="(firings, range, index) in currentSelectedFiringTime.ranges" :key="range"
        class="mb-4 p-2"
        :class="{'border-red-500 border-2': (firings.day || firings.night)}"
      >
        <div>
          {{ range.charAt(0).toUpperCase()+range.slice(1) }}
        </div>
        <div class="mt-1 flex flex-row">
          <div class="flex-1 text-sm">Day: {{ firings.day? 'Firing' : 'No Firing' }}</div>
          <div class="flex-1 text-sm">Night: {{ firings.night? 'Firing' : 'No Firing' }}</div>
        </div>
      </div>
    </div>

    <div class="p-2 flex flex-col mb-2" v-if="firingProgramDocumentUrl">
      <p>
        <a :href="'https://www.gov.uk'+firingProgramDocumentUrl" target="_blank" class="underline hover:no-underline">Latest Dartmoor Firing Programme</a>
      </p>
    </div>

    <!-- Layers -->
    <div class="p-2 flex flex-col mb-4">
      <h3 class="text-xl font-weight-bold mb-2">Layers</h3>
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-bind="activeLayers.campingArea"
              v-model="activeLayers.campingArea"
              />
            Camping Zones
          </summary>
          <p class="mt-5">Permitted backpacking camping areas</p>
          <p>Source: <a href="https://www.dartmoor.gov.uk/about-us/about-us-maps/camping-map" target="_blank" class="underline hover:no-underline">dartmoor.gov.uk</a></p>
        </details>
      </div>
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-bind="activeLayers.accessAreas"
              v-model="activeLayers.accessAreas"
              />
            Access Land
          </summary>
          <p class="mt-5">Free to roam areas</p>
        </details>
      </div>
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-bind="activeLayers.nestingBirdsArea"
              v-model="activeLayers.nestingBirdsArea"
            />
            Ground Nesting Birds
          </summary>
          <p class="mt-5">Ground nesting birds have been identified in these areas, avoid when possible between March and July</p>
          <p>Source: <a href="https://www.dartmoor.gov.uk/wildlife-and-heritage/wildlife/birds/birds-nesting" target="_blank" class="underline hover:no-underline">dartmoor.gov.uk</a></p>
        </details>
      </div>
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-bind="activeLayers.weatherPoints"
              v-model="activeLayers.weatherPoints"
            />
            Weather
          </summary>
          <p class="mt-5">Weather forecast locations</p>
        </details>
      </div>
    </div>

    <!-- Weather Forecasts-->
    <div
      class="p-2 flex flex-col mb-4"
    >
      <h3 class="text-xl font-weight-bold mb-2">Weather Forecasts</h3>
      <div class="flex flex-col">
        <div
          v-for="(weather, index) in weatherLocations"
          :key="index"
        >
          <p>{{ weather.locationName }}</p>
          <div class="flex flex-row">
            <a
              v-for="(site, index) in weather.sites"
              :key="index"
              :href="site.url" target="_blank"
              class="underline hover:no-underline p-3"
            >
              {{ site.name }}
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
