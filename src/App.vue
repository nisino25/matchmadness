<template>
  <div class="container mx-auto p-2 bg-gray-100 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">Match Madness</h1>
    <p class="mb-4">Match the English words with their Italian counterparts, Nozo!</p>

    <!-- group looping -->
    <div v-if="groups.length > 0 && !readyToPlay" class="row gx-2 gy-2 mb-3">
      <div
        v-for="(group, index) in computedGroups"
        :key="index"
        class="col-6"
      >
        <div 
          :style="{
              backgroundColor: group.name === 'unchecked' ? 'LightSlateGray' : '',
              opacity: group.counter === 0 ? 0.5 : 1
          }"
          class="card shadow" 
          @click="startMatchmadness(group)" 
          style="cursor: pointer;"
        >
          <div class="card-body">
            <h5 class="card-title text-center">{{ group.name }}</h5>
            <hr>
            <section class="flex-container" style="width: 80%; margin: auto;">
              <small><i class="fa-solid fa-list"></i> <strong>{{ group.words.length }}</strong></small>
              <small v-if="group.name !== 'unchecked'  && group.name !== 'Marked Words'"><i class="fa-solid fa-bookmark"></i> <strong>{{ group.words.filter(word => word.marked)?.length }}</strong></small>
              <small v-if="group.name !== 'unchecked' && group.name !== 'Marked Words'"><i class="fa-regular fa-clock"></i> <strong>{{ group.counter }}</strong></small>
            </section>
          </div>
        </div>
      </div>
    </div>

    <!-- loading animation  -->
    <div v-if="groups?.length == 0" class="text-center fixed left-1/2 top-1/4 transform -translate-x-1/2 -translate-y-1/2">
      <div class="relative w-[22.5vw] aspect-square m-8">
          <div class="absolute w-[80%] h-[80%] border-4 border-indigo-300 rounded-full animate-[ripple_1.5s_infinite] left-1/2 top-1/2 text-center fixed left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
          <div class="absolute w-[80%] h-[80%] border-4 border-indigo-300 rounded-full animate-[ripple_1.5s_infinite] [animation-delay:0.5s] left-1/2 top-1/2 text-center fixed left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
      </div>
    </div>


    <!-- match madness -->
    <div v-if="readyToPlay">
      <hr>
      <h2>No.{{ currentIndex }} / {{ shuffledList.length }}</h2>
      <hr>
      <div class="grid grid-cols-2 gap-4" >
        <div>
          <h2 class="text-xl font-semibold mb-2">English</h2>
          <ul>
            <li
              v-for="(word, index) in englishBoard"
              :key="'eng-' + index"
              @click="selectEnglish(index)"
              :class="[
                'h-12 p-2 mb-2 bg-white cursor-pointer rounded',
                selectedEnglish.index === index ? 'border-2 border-blue-500' : ''
              ]"
            >
              {{ word?.english || '' }}
            </li>
          </ul>
        </div>

        <div>
          <h2 class="text-xl font-semibold mb-2">Italian</h2>
          <ul>
            <li
              v-for="(word, index) in italianBoard"
              :key="'ita-' + index"
              @click="selectItalian(index)"
              :class="[
                'h-12 p-2 mb-2 bg-white cursor-pointer rounded',
                selectedItalian.index === index ? 'border-2 border-blue-500' : ''
              ]"
            >
              {{ word?.italian || '' }}
            </li>
          </ul>
        </div>
      </div> 
    </div>

    <div class="mt-4">
      <p class="text-green-500">{{ message }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

// const originalList = [
//   { italian: "zucchero", english: "sugar" },
//   { italian: "arrivederci", english: "goodbye" },
//   { italian: "cappuccino", english: "cappuccino" },
//   { italian: "gelato", english: "ice cream" },
//   { italian: "no", english: "no" },
//   { italian: "ma", english: "but" },
//   { italian: "latte", english: "milk" },
//   { italian: "sì", english: "yes" },
//   { italian: "grazie", english: "thank you" },
//   { italian: "con", english: "with | to" },
//   { italian: "caffè", english: "coffee" },
//   { italian: "lo", english: "the | him | it | that" },
//   { italian: "un", english: "a | an" },
//   { italian: "salve", english: "hello" },
//   { italian: "tè", english: "tea" },
//   { italian: "per favore", english: "please" },
//   { italian: "cornetto", english: "croissant" },
// ]
const baseUrl = ref('https://script.google.com/macros/s/AKfycbzFbw5BxXC51tZZTz4QWekcSWIfx0eidErj8JviL_KWKf5bJdOyaPtP6CxJeWC_Y_GgrQ/exec')
const groups = ref([])

const shuffledList = ref([])
const englishBoard = ref(Array(5).fill({}))
const italianBoard = ref(Array(5).fill({}))
const selectedEnglish = ref({ index: null, word: null })
const selectedItalian = ref({ index: null, word: null })
const message = ref("")
const readyToPlay = ref(false);

const currentIndex = ref(0);

onMounted(() => {
  console.clear();
  fetchData();
  
})

const fetchData = () => {
    console.log(`getting reload`);
    groups.value = [];

    const url = `${baseUrl.value}?callback=jsonpCallback&action=fetchData`;

    // Define the callback function globally
    window.jsonpCallback = (data) => {
        console.log("API Response (fetchData):", data);
        if (data.success) {
            processData(data.data); // Process and store data
        } else {
            console.error("Error fetching data:", data.message);
        }
    };

    // Dynamically add a <script> tag to call the JSONP API
    const script = document.createElement("script");
    script.src = url; // Set the API URL
    script.async = true; // Load asynchronously
    document.body.appendChild(script);

    // Clean up the <script> tag after the request
    script.onload = () => {
      document.body.removeChild(script); // Remove the script tag
    };
}

const processData =  (values) => {
        let tempGroups = [];
        let tempGroup = { name: "", words: [] };
        let unchecked = { name: "unchecked", words: [] }; // Group for rows with `uncheckedFlag` == 1

        // Helper function to process a word and add it to the appropriate group
        const addWordToGroup = (row, targetGroup) => {
            if (row.it && row.en) {
                targetGroup.words.push({
                  italian: row.it,
                  english: row.en,
                    marked: row.marked == 1,  // Ensure boolean conversion
                    counter: row.counter ? Number(row.counter) : 0,
                });
            }
        };

        for (let row of values) {
            // Handle "unchecked" words
            if (row.uncheckedFlag == 1) {
                addWordToGroup(row, unchecked);
                continue;
            } 
            // Handle group headers (groupName)
            else if (row.en === "groupName") {
                if (tempGroup.name) tempGroups.push(tempGroup);
                tempGroup = { name: row.it, words: [], counter: row.counter ? Number(row.counter) : 0 };
                continue;
            } 
            // Handle regular words
            else {
                addWordToGroup(row, tempGroup);
            }
        }

        if (tempGroup.name) tempGroups.push(tempGroup);
        if (unchecked.words.length > 0) tempGroups.push(unchecked); // Add "unchecked" group if it has words

        groups.value = null;
        groups.value = tempGroups;
}

const filteredWords = computed(() => {
  return groups.value.flatMap(group => group.words).filter(word => word.marked);
})

const mistakesInThePast = computed(() =>{ 
  return groups.value.flatMap(group => group.words).filter(word => word.counter !== 0 );
})

const computedGroups = computed(() => {
  const markedWordsOnly = {
        name: "Marked Only",
        words: filteredWords.value,
        counter: '?', // No counter needed
  };

  const mistakesWordsOnly = {
        name: "Mistakes Only",
        words: mistakesInThePast.value,
        counter: '?', // No counter needed
  };

  return [mistakesWordsOnly, markedWordsOnly, ...groups.value];
});

const startMatchmadness = (group) =>{
  currentIndex.value = 0

  console.log(group)
  const listWithIds = group.words.map((item, index) => ({
    id: index + 1,
    ...item,
  }))

  shuffledList.value = shuffle([...listWithIds, ...listWithIds])
  for (let i = 0; i < 5; i++) {
    randomRefill()
  }

  readyToPlay.value = true;
}


function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}


function randomRefill() {
  if (shuffledList.value.length === 0) return

  const emptyEnglishIndexes = englishBoard.value
    .map((item, index) => (Object.keys(item).length === 0 ? index : -1))
    .filter((index) => index !== -1)

  const emptyItalianIndexes = italianBoard.value
    .map((item, index) => (Object.keys(item).length === 0 ? index : -1))
    .filter((index) => index !== -1)

  if (emptyEnglishIndexes.length === 0 || emptyItalianIndexes.length === 0) return

  const randomEnglishIndex =
    emptyEnglishIndexes[Math.floor(Math.random() * emptyEnglishIndexes.length)]
  const randomItalianIndex =
    emptyItalianIndexes[Math.floor(Math.random() * emptyItalianIndexes.length)]

  const selectedItem = shuffledList.value.shift()

  englishBoard.value[randomEnglishIndex] = { ...selectedItem }
  italianBoard.value[randomItalianIndex] = { ...selectedItem }
}

function selectEnglish(index) {
  if (!englishBoard.value[index]) return
  selectedEnglish.value = { index, word: englishBoard.value[index] }
  if (selectedEnglish.value.word && selectedItalian.value.word) {
    checkMatch()
  }
}

function selectItalian(index) {
  if (!italianBoard.value[index]) return
  selectedItalian.value = { index, word: italianBoard.value[index] }
  if (selectedEnglish.value.word && selectedItalian.value.word) {
    checkMatch()
  }
}

function checkMatch() {
  if (selectedEnglish.value.word.id === selectedItalian.value.word.id) {
    currentIndex.value = currentIndex.value + 1
    message.value = "Correct match, Nozo!"
    englishBoard.value[selectedEnglish.value.index] = {}
    italianBoard.value[selectedItalian.value.index] = {}
    selectedEnglish.value = { index: null, word: null }
    selectedItalian.value = { index: null, word: null }

    const emptyCount = englishBoard.value.filter((item) => Object.keys(item || {}).length === 0).length
    if (emptyCount >= 2) {
      randomRefill()
    }
  } else {
    message.value = "Try again, Nozo!"
    setTimeout(() => {
      selectedEnglish.value = { index: null, word: null }
      selectedItalian.value = { index: null, word: null }
      message.value = ""
    }, 1000)
  }
}
</script>

<style>
#app {
  height: 100vh; /* Ensures the app takes the full viewport height */
  display: flex; /* Optional: For centering or flex layouts */
  flex-direction: column; /* Optional: Ensures children stack vertically */
}

.toggle-container{
  display: flex;
  justify-content: space-around !important;
  align-items: center;
}

.flag-container{
  display: flex !important;
  align-items: center;
  gap: 5px;
  
}

.form-check-inline{
  margin-right: unset !important;
}

.flag{
  font-size: 2em;
}

.flex-container{
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 10px;
}

.text-center{
  justify-content: center;
}

.flex-container span{
  font-size: 1em;
}

.flex-container h4{
  margin-bottom: unset !important;
}

.card-title{
  margin-bottom: unset !important;
}

.card-body{
  padding: 10px !important;
}

.card-body hr{
  margin: .5em 0;
}

.flashcard{
  border: 1px solid lightslategray;
}


@keyframes ripple {
    from {
        transform: translate(-50%,-50%) scale(0);
        opacity: 1;
    }
    to {
        transform: translate(-50%,-50%) scale(1.5);
        opacity: 0;
    }
}
</style>
