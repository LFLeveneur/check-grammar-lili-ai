<script>
import axios from "axios";
import { ref } from "vue";

export default {
  // Sate
  data() {
    return {
      submit: false,
      msg: "",
      msgFault: null,
      msgTableWord: [],
      faultSuggestions: [],
      viewSuggestions: '',
    }
  },
  // Methods
  methods: {
    checkSend() {
      axios.request({
        method: 'POST',
        url: 'https://jspell-checker.p.rapidapi.com/check',
        headers: {
          'content-type': 'application/json',
          'X-RapidAPI-Key': 'd6eee89164msh1f1c1ff09dc6f4fp16d331jsnb9fe8a5b5dea',
          'X-RapidAPI-Host': 'jspell-checker.p.rapidapi.com'
        },
        data: {
          language: "enUS",
          fieldvalues: this.msg,
          config: {
            forceUpperCase: false,
            ignoreIrregularCaps: false,
            ignoreFirstCaps: true,
            ignoreNumbers: true,
            ignoreUpper: false,
            ignoreDouble: false,
            ignoreWordsWithNumbers: true
          }
        }
      })
      .then(response => {
        this.msgTableWord = this.msg.split(" ");
        this.msgFault = response.data.elements[0].errors;
        this.checkGrammar(this.msgTableWord, this.msgFault);
      })
      .catch(err => console.log(err));
    },
    // ----------------------------------------------------------------------------------------------
    checkSuggestion(msgWord, msgFault) {
      this.viewSuggestions = '';
      let msgFaultLength = msgFault.length;
      let faultSuggestions = [];
      for (let i = 0; i < msgFaultLength; i++) {
        if (msgWord == msgFault[i].word) {
          for (let j = 0; j < 3; j++) {
            if (msgFault[i].suggestions[j] != undefined) {
              faultSuggestions.push(msgFault[i].suggestions[j]);
            }
          }
        }
        else {
          this.viewSuggestions = '';
        }
      }
      this.viewSuggestions = `<ul class="view-suggestions"> ${faultSuggestions.map(suggestion => `<li>${suggestion}</li>`).join('')} </ul>`
      
    },
    // ----------------------------------------------------------------------------------------------
    checkGrammar(msgTableWord, msgFault) {
      console.log(msgTableWord);
      console.log(msgFault);
      let msgTableWordLength = msgTableWord.length;
      let msgFaultLength = msgFault.length;
      console.table(msgTableWord);
      
      for (let i = 0; i < msgTableWordLength; i++) {
        for (let j = 0; j < msgFaultLength; j++) {
          if (msgTableWord[i] == msgFault[j].word) {
            console.log(msgTableWord[i]);
            /*
            let msgWordId = 'word' + msgTableWord[i];
            let doc = this.$refs.word+msgTableWord[i];
            console.log(msgWordId, doc);
            */
          }
        }
      }
    }
  },
  // ----------------------------------------------------------------------------------------------
  updated() {
    console.log('updated');
    // if (this.submit) {
    //   this.checkGrammar(this.msgTableWord, this.msgFault);
    // }
  }
};


</script>

<template>
  <div class="field">
    <form action="submit">
      <label>
        <span>Grammar Checker: check spelling and grammar for English texts</span>
        <textarea class="text-area" v-if="!submit" v-model="msg" placeholder="Type your text here"></textarea>
        <div v-if="submit" class="text-area text-area-submit">
          <ul>
            <li v-for="msgWord in msgTableWord" :key="msgWord.id" @click="checkSuggestion(msgWord, msgFault)" :ref="'word' + msgWord">
              {{ msgWord }}&nbsp;
            </li>
          </ul>
        </div>
        <ul v-html="viewSuggestions" v-if="viewSuggestions !== ''"></ul>
      </label>
      <input type="submit" value="Check" @click.prevent="checkSend()" @click="submit = !submit">
    </form>
    <!-- if error msg !== '' display ul -->
    <ul class="fault" v-if="msgFault === []">
    <p>list :</p>
      <li v-for="error in msgFault" v-bind:key="error.id">{{ error }}</li>
    </ul>
  </div>
</template>
