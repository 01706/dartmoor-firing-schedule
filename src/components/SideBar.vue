<script setup>
import { ref, computed, watch, reactive } from 'vue';
import weatherLocations from '@/weatherLocations';

const {firingTimes, firingProgramDocumentUrl, activeLayers, peaks} = defineProps({
  firingTimes: {
    type: [null, Array],
    required: true
  },
  firingProgramDocumentUrl: {
    type: [null, String],
  },
  activeLayers: {
    type: Object,
  },
  peaks: {
    type: [null, Array],
  }
});

const emit = defineEmits(['selectedFiringTimeChanged', 'activeLayersChanged', 'selectedPeaksChanged']);

const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
const months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

const todaysDate = new Date(new Date().setHours(0,0,0,0));
const currentSelectedFiringTime = ref(null);
let currentSelectedFiringTimeIndex = null;

const peakSearchQuery = ref(null);

const peaksQuery = computed(() => {

  if (peakSearchQuery.value === null
    || peakSearchQuery.value.length < 3
  ) {
    return peaks;
  }

  return peaks.filter((item) => {
    return peakSearchQuery.value.toLowerCase().split(' ').every(v => item.name.toLowerCase().includes(v));
  });
});

const selectedPeaks = ref([]);

watch(
  selectedPeaks,
  (updated, previous) => {
    console.debug('sidebar: selected peaks changed');
    emit('selectedPeaksChanged', updated)
  },
  {
    immediate: true,
  }
);

/**
 * Firing Areas
 */

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

/**
 * Peaks & Tors
 */

function clearPeakSearchQuery() {
  peakSearchQuery.value = null;
}

function selectAllPeaks() {
  if (peaks === null) {
    console.warn('peaks have not been loaded, not able to select all');
    return;
  }
  selectedPeaks.value = peaks.map((e) => e.id);
}

function deselectAllPeaks() {
  selectedPeaks.value = [];
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
      <!-- Camping zones -->
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-model="activeLayers.campingArea"
              />
            Camping Zones
          </summary>
          <p class="mt-5">Permitted backpacking camping areas</p>
          <p>Source: <a href="https://www.dartmoor.gov.uk/about-us/about-us-maps/camping-map" target="_blank" class="underline hover:no-underline">dartmoor.gov.uk</a></p>
        </details>
      </div>
      <!-- Access land -->
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-model="activeLayers.accessAreas"
              />
            Access Land
          </summary>
          <p class="mt-5">Free to roam areas</p>
        </details>
      </div>
      <!-- Ground Nesting Birds -->
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-model="activeLayers.nestingBirdsArea"
            />
            Ground Nesting Birds
          </summary>
          <p class="mt-5">Ground nesting birds have been identified in these areas, avoid when possible between March and July</p>
          <p>Source: <a href="https://www.dartmoor.gov.uk/wildlife-and-heritage/wildlife/birds/birds-nesting" target="_blank" class="underline hover:no-underline">dartmoor.gov.uk</a></p>
        </details>
      </div>
      <!-- Weather -->
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3">
          <summary>
            <input type="checkbox" class="mr-2 ml-2"
              v-model="activeLayers.weatherPoints"
            />
            Weather
          </summary>
          <p class="mt-5">Weather forecast locations</p>
        </details>
      </div>
      <!-- Peaks & Tors -->
      <div class="rounded-md bg-gray-100 m-2">
        <details class="p-3" open>
          <summary>
            Peaks & Tors
          </summary>
          <div class="mt-3 grid grid-cols-2 gap-3">
            <button type="button" class="bg-green-200 rounded-md cursor-pointer p-1" @click.prevent="selectAllPeaks">Show All</button>
            <button type="buttom" class="bg-red-200 rounded-md cursor-pointer p-1" @click.prevent="deselectAllPeaks">Hide All</button>
            <button type="buttom" class="bg-orange-200 col-span-2 rounded-md cursor-pointer p-1" @click.prevent="clearPeakSearchQuery">Clear</button>
          </div>
          <div class="mt-2 mb-3">
            <input type="text" name="tor-search" class="block w-full bg-white p-1 border-1 border-gray-500" placeholder="Search ..." v-model="peakSearchQuery"/>
          </div>
          <!-- items -->
          <div class="flex flex-col">
            <div v-if="peaks" v-for="peak in peaksQuery" :key="peak.id" class="mb-4">
              <label class="inline-block w-full border-t-1 border-gray-500"><input type="checkbox" class="mr-2 ml-2" :value="peak.id" v-model="selectedPeaks"/>{{ peak.name }}</label><br>
              <div class="ml-9 mt-2 grid grid-cols-2 gap-3 justify-between">
                <div><abbr title="Elevation">Ele</abbr>: {{ peak.ele? `${peak.ele}m` : 'Unknown' }}</div>
                <div><abbr title="Camping Zone">CZ</abbr>: {{ peak.cz===true? 'Within' : 'Outside' }}</div>
                <div><abbr title="Firing Zone">FZ</abbr>: {{ peak.fz!==null? peak.fz.charAt(0).toUpperCase() + peak.fz.slice(1) : 'Outside'  }}</div>
              </div>
            </div>
            <div v-else>
              Loading...
            </div>
          </div>
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
