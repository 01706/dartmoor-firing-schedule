<script setup>
import { ref, reactive } from 'vue';
import Map from './components/Map.vue';
import SideBar from './components/SideBar.vue';

const firingTimes = ref(null);
const firingProgramDocumentUrl = ref(null);
const currentSelectedFiringTime = ref(null);

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
          for(let i = 1; i < table.rows.length; i++) {
              const okehampton = {
                day: table.rows[i].cells[1].innerText.includes('day'),
                night: table.rows[i].cells[1].innerText.includes('night')
              };

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

</script>

<template>
  <header>
  </header>

  <div class="flex flex-col sm:flex-row grow h-screen">
    <aside class="h-64 w-full sm:h-full sm:w-64 overflow-x-scroll">
      <SideBar
        @selectedFiringTimeChanged="selectedFiringTimeChanged"
        :firingTimes="firingTimes"
        :firingProgramDocumentUrl="firingProgramDocumentUrl"
        :activeLayers="activeLayers"
      />
    </aside>
    <main class="grow">
      <Map
        :currentSelectedFiringTime="currentSelectedFiringTime"
        :activeLayers="activeLayers"
      />
    </main>
  </div>

</template>

<style scoped>

</style>
