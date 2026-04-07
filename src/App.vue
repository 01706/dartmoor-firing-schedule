<script setup>
import { ref, reactive } from 'vue';
import Map from './components/Map.vue';
import SideBar from './components/SideBar.vue';

const firingTimes = ref(null);
const firingProgramDocumentUrl = ref(null);
const currentSelectedFiringTime = ref(null);
const peaks = ref(null);
const currentSelectedPeaks = ref([]);
const peakToZoomTo = ref(null);

const activeLayers = reactive({
  firingArea: false,
  campingArea: false,
  nestingBirdsArea: false,
  accessAreas: false,
  weatherPoints: false,
});

getDartmoorFiringProgramDocumentUrl()
  .then((url) => {
      return getDartmoorFiringProgramDocument(url);
  })
  .then((data) => {
      firingTimes.value = data;
  });

async function getDartmoorFiringProgramDocumentUrl() {
  const endPoint = 'https://www.gov.uk/api/content/government/publications/dartmoor-firing-programme';

  try {
      const response = await fetch(endPoint);
      if (!response.ok) {
          throw new Error('Unable to get reach firing program');
      }

      const json = await response.json();
      if (json.details.attachments.length > 0) {
          return json.details.attachments[0].url;
      } else {
          throw new Error('Unable to get the latest firing program document')
      }
  } catch (error) {
      console.error("Error:", error);
  }
}

async function getDartmoorFiringProgramDocument(url) {
  const endPoint = `https://www.gov.uk/api/content/${url}`;

  let data = [];
  try {
      const response = await fetch(endPoint);
      if (!response.ok) {
          throw new Error('Unable to get reach firing program');
      }

      const json = await response.json();
      if (json.details.body.length == 0) {
          throw new Error('Unable to get the latest firing program content')
      }

      const body = new DOMParser().parseFromString(json.details.body, "text/html");

      [...body.getElementsByTagName('table')].forEach(function(table) {
          let start = 0;

          if (doesTableHaveHeader(table)) {
            start = 1;
          }

          for(let i = start; i < table.rows.length; i++) {
            data[data.length] = {
                  'date': new Date(table.rows[i].cells[0].innerText),
                  'dateText': table.rows[i].cells[0].innerText,
                  'ranges': {
                    'okehampton': {
                      day: table.rows[i].cells[1].innerText.includes('day'),
                      night: table.rows[i].cells[1].innerText.includes('night')
                    },
                    'willsworth': {
                      day: table.rows[i].cells[2].innerText.includes('day'),
                      night: table.rows[i].cells[2].innerText.includes('night')
                    },
                    'merrivale': {
                      day: table.rows[i].cells[3].innerText.includes('day'),
                      night: table.rows[i].cells[3].innerText.includes('night')
                    },
                }
              }
          }
      });

      firingProgramDocumentUrl.value = url;
  } catch (error) {
      console.error("Error:", error);
  }
  return data;
}

function selectedFiringTimeChanged(newFiringTime) {
  currentSelectedFiringTime.value = newFiringTime;
}

function doesTableHaveHeader(table) {
  return table.rows[0].cells[0].innerText.toLowerCase() === 'date';
}


// Peaks and tors

function peaksLoaded(loadedPeaks) {
  peaks.value = loadedPeaks
    // Remove peaks which dont have a name
    .filter((item) => {
      return item.name !== undefined && item.name !== null
    })
    // Sort by name ASC
    .sort((a, b) => (a.name > b.name)? 1 : ((b.name > a.name)? -1: 0));
}

function selectedPeaksChanged(selectedPeaks) {
  currentSelectedPeaks.value = selectedPeaks;
}

function zoomToPeak(peaKId) {
  peakToZoomTo.value = peaKId;
  setTimeout(() => peakToZoomTo.value = null, 100);
}
</script>

<template>
  <header>
  </header>

  <div class="flex flex-col sm:flex-row grow h-screen">
    <aside class="h-64 w-full sm:h-full sm:w-sm overflow-x-scroll">
      <SideBar
        @selectedFiringTimeChanged="selectedFiringTimeChanged"
        @selectedPeaksChanged="selectedPeaksChanged"
        @zoomToPeak="zoomToPeak"
        :firingTimes="firingTimes"
        :firingProgramDocumentUrl="firingProgramDocumentUrl"
        :activeLayers="activeLayers"
        :peaks="peaks"
      />
    </aside>
    <main class="grow">
      <Map
        @peaksLoaded="peaksLoaded"
        :currentSelectedFiringTime="currentSelectedFiringTime"
        :activeLayers="activeLayers"
        :currentSelectedPeaks="currentSelectedPeaks"
        :peakToZoomTo="peakToZoomTo"
      />
    </main>
  </div>
</template>

<style scoped>
</style>
