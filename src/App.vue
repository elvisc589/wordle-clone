<script setup>
import { reactive, onMounted, computed } from "vue";
import WordRow from "./components/WordRow.vue";
import SimpleKeyboard from "./components/SimpleKeyboard.vue";
import { wordsList } from './data.js'

var ranInt = Math.floor(Math.random() * wordsList.length);
var word = new String(wordsList[ranInt]);

const state = reactive({
  solution: word,
  guesses: ["", "", "", "", "", ""],
  currentGuessIndex: 0,
  guessedLetters: {
    miss: [],
    found: [],
    hint: [],
  },
});

const wonGame = computed(
  () =>
  state.guesses[state.currentGuessIndex - 1] == state.solution
);

const lostGame = computed(() => !wonGame.value
  && state.currentGuessIndex >= 6
);

const handleInput = (key) => {
  if(state.currentGuessIndex>=6 || wonGame.value){
    return;
  }
  const currentGuess=state.guesses[state.currentGuessIndex];
  if(key=="{enter}"){
    //Send guess
    if(currentGuess.length==5){
      state.currentGuessIndex++;
      for(var i=0; i<currentGuess.length; i++){
        let c = currentGuess.charAt(i);
        if(c==state.solution.charAt(i)){
          state.guessedLetters.found.push(c);
          //index f returns -1 if value is not found in solution
        } else if(state.solution.indexOf(c)!= -1){
          state.guessedLetters.hint.push(c);
        } else{
          state.guessedLetters.miss.push(c);
        }
      }
    }
  }else if(key=="{bksp}"){
     state.guesses[state.currentGuessIndex]=
    currentGuess.slice(0,-1);
  }else if(currentGuess.length<5){
    //Add letter if alphabetical
      const alphaRegex = /[a-zA-Z]/;
      if(alphaRegex.test(key)){
      state.guesses[state.currentGuessIndex] += key;
      }
  }
  
};

onMounted(()=> {
  window.addEventListener("keyup", (e)=>{
    e.preventDefault();
    let key = 
      e.key == 'Enter'
       ? "{enter}"
       : e.key == 'Backspace'
       ? "{bksp}"
       : (e.key).toLowerCase();
    handleInput(key);
    console.log(key);
  });
});

</script>

<template>
  <div class="flex flex-col h-screen max-w-md mx-auto justify-evenly">
    <div>
      <word-row
        v-for="(guess, i) in state.guesses"
        :key="i"
        :value="guess"
        :solution="state.solution"
        :submitted="i<state.currentGuessIndex"
      />
    </div>
    
    <p v-if="wonGame" class="text-center">
      🏆 Congrats, you solved it! 
    </p>
    <p v-else-if="lostGame" class="text-center">
      
      😔 Out of tries. 
    </p>
    <simple-keyboard @onKeyPress="handleInput"
    :guessedLetters="state.guessedLetters"
    />
  </div>
</template>


<style></style>
