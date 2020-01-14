<template>
  <div class="main">

    <button class="export" @click="exportJson" style="margin-bottom: 20px">Export Quiz</button>
    
      <label class="fileContainer">
          Upload File
          <input type="file" @change="loadBank($event)" accept=".json"/>
      </label>
      
    <catbox>

      <h5>Categories</h5>

      <div class="catholder">
        <div v-for="category in qdata.Categories" v-bind:key="category.value">
          <span class="catnum">{{category.value}}</span>: <input type="text" v-model="category.text" :style="{width: getWidth(category.text)}"> <button class="remove" @click="deleteCategory(category)">-</button>
        </div>
      </div>

      <input type="text" v-model="newCat" :style="{width: getWidth(newCat)}"> <button @click="addCategory" class="add" >+</button>
    </catbox>

    <h4>
      <transition name="smooth">
        <button @click="previousQuestion" v-show="previousQuestionExists">Previous</button>
      </transition>
      Question {{question.qid}} <span id="qnum"> of {{numQuestions}} </span>
      <transition name="smooth">
        <button @click="nextQuestion" v-if="nextQuestionExists">Next</button>
      </transition>
    </h4> 


    Question: <br/><textarea type="textbox" v-model="question.question" style="width:40%"/> 
    <br/>
    <button class="add" @click="addQuestion" style="margin-top: 50px">Add Question</button>
    <transition name="smooth">
      <button class="remove" v-if="notFirstQuestion" @click="deleteQuestion" style="margin-top: 50px">Delete Question</button>
    </transition>
    <div class="big-container">
      
    <br/>
    <sub>Answer: Text / Category / Points</sub>

    <div v-for="answer in question.answers" v-bind:key="answer.key"> 
      <input type="text" v-model="answer.text" :style="{width: getWidth(answer.text)}"> 
      <select v-model="answer.category">
        <option v-for="category in qdata.Categories" v-bind:key="category.value" v-bind:value="category.value">
          {{category.text}} 
        </option>
      </select>
      <input type="number" name="points" v-model="answer.points">

      <button class="add" @click="addAnswer(answer)">+</button>
      <button class="remove" @click="deleteAnswer(answer)">-</button>
    
    </div>

    
    </div>
  </div>
  
</template>

<script>
import qbank from "../assets/data.json"

export default {
  name: 'QView',
  data() { return {
        qdata: Object,
        question: Object,
        newCat: new String
      }
  },
  methods: {
    validateQData: function(data){

      let cats = data.Categories;
      let quests = data.Bank;

      //Check qbank structure

      //Categories
      if(cats == undefined || quests == undefined){
        alert("ERROR! Root element missing.");
        return false;
      }

      cats.forEach(function(cat){
        if(cat.value == undefined || cat.text == undefined){
          alert("ERROR! Corrupt category.");
          return false;
        }
      });

       

      quests.forEach(function(quest){

        if(quest.qid == undefined){
         alert("ERROR! Corrupt qid. ");
         return false;
        }
        if(quest.question == undefined){
         alert("ERROR! Corrupt question value. QID: " + quest.qid);
         return false;
        }
        quest.answers.forEach(function(ans){
          if(ans.key == undefined){
            alert("ERROR! Corrupt answer key. QID: " + quest.qid);
            return false;
          }
          if(ans.text == undefined){
            alert("ERROR! Corrupt answer text. QID: " + quest.qid);
            return false;
          }
          if(ans.points == undefined){
            alert("ERROR! Corrupt answer points. QID: " + quest.qid);
            return false;
          }
          if(ans.category == undefined){
            alert("ERROR! Corrupt answer category. QID: " + quest.qid);
            return false;
          }
        });
      });
    
      //Check categories match
      quests.forEach(function(quest){
        quest.answers.forEach(function(answer){

            var catIn = false;

            cats.forEach(function(cat){
              if (answer.category === cat.value){
                catIn = true;
              }
            });

            if(!catIn){
              alert("ERROR! Category mismatch");
              return false;
            }
        });
      });

      return true;
    },
    exportJson: function(){
      
      let bank = this.qdata.Bank;

      var uncat = false;

      let uncatQuests = []

      bank.forEach(function(quest){
        quest.answers.forEach(function(ans){
          if(ans.category == 0){
            uncat = true;
            uncatQuests.splice(uncatQuests.length, 0, quest.qid);
          }
        });
      });

      let uqs = " ";

      uncatQuests.forEach(function(uq){
        uqs = uqs + uq + " ";
      });

      var dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.qdata, null, 2));
      var downloadAnchorNode = document.createElement('a');

      if(uncat){
        if (confirm('Are you sure you want export the quiz with uncategorized answers? The following questions have answers that need categorizing: ' + uqs)) {
          downloadAnchorNode.setAttribute("href",     dataStr);
          downloadAnchorNode.setAttribute("download", "questionBank.json");
          document.body.appendChild(downloadAnchorNode); // required for firefox
          downloadAnchorNode.click();
          downloadAnchorNode.remove();
        }
      }
      else{
          downloadAnchorNode.setAttribute("href",     dataStr);
          downloadAnchorNode.setAttribute("download", "questionBank.json");
          document.body.appendChild(downloadAnchorNode); // required for firefox
          downloadAnchorNode.click();
          downloadAnchorNode.remove();
      }
      
    },
    loadBank: async function(event){

      let foreignQBank = event.target.files[0];
      var text = await foreignQBank.text();
      let newQBank = JSON.parse(text);
      
      if(this.validateQData(newQBank)){
        this.qdata = newQBank;
        this.question = newQBank.Bank[0];
      }
      else{
        alert("Fix the corrupt question bank or upload a different question bank.")
      }
    },
    addAnswer: function(ansClicked){

      let index = this.question.answers.indexOf(ansClicked) + 1;

      this.question.answers.splice(index,
        0,
        {
        "key" : index + 1,
        "text" : "New Answer",
        "points" : 1,
        "category": this.qdata.Categories["1"]
        }
      );

      if (this.question.answers[index + 1].key != 0){
        for (let i = index + 1; i < this.question.answers.length; i++ ){
          this.question.answers[i].key = this.question.answers[i].key + 1;
        }
      }
    },
    deleteAnswer: function(ansClicked){
      
      let index = this.question.answers.indexOf(ansClicked);

      this.question.answers.splice(index, 1)

      if (this.question.answers[index].key != 0){
        for (let i = index; i < this.question.answers.length; i++ ){
          this.question.answers[i].key = this.question.answers[i].key - 1;
        }
      }
    },
    addCategory: function(){

    let cats = this.qdata.Categories;
      
    var last =  function(array, n) {
        if (array == null) 
          return void 0;
        if (n == null) 
          return array[array.length - 1];
        return array.slice(Math.max(array.length - n, 0));  
      };

    var lastc = last(cats, 1)[0];

    var lastcIndex = cats.indexOf(lastc);

    var lastcVal = lastc.value;

    cats.splice(lastcIndex + 1,
              0,
              {
                "value" : lastcVal + 1,
                "text" : this.newCat
              });

    },
    deleteCategory: function(category){
      
      let cats = this.qdata.Categories;
      let index = cats.indexOf(category);
      let value = category.value;

      cats.splice(index, 1)
      
      //all category values by -1

      for(let i = index; i < cats.length; i++){
        cats[i].value = cats[i].value - 1;
      }

      let bank = this.qdata.Bank;

      bank.forEach(function(quest){
        quest.answers.forEach(function(ans){
          if(ans.category == value){
            ans.category = 0;
          }
          else if(ans.category > value){
            ans.category = ans.category - 1;
          }
        });
      });
      //go through every question, if answer.category is equal to or greater than value -> 
      //if equal = set answer.category to 0
      //if greater = set answer.category to answer.category - 1

    },
    addQuestion: function(){
      let bank = this.qdata.Bank;
      let qs_index = bank.length

      var last =  function(array, n) {
        if (array == null) 
          return void 0;
        if (n == null) 
          return array[array.length - 1];
        return array.slice(Math.max(array.length - n, 0));  
      };

      var lastq = last(bank, 1);

      bank.splice(qs_index,
        0,
        {
                "qid" : lastq[0]["qid"] + 1,
                "question" : "Question Text",
                "answers" : [
                    {
                        "key" : 1,
                        "text" : "New Answer 1",
                        "points" : 1,
                        "category": 1
                    }
                ]
            }
      );

      this.question = this.qdata.Bank[qs_index]
    },
    previousQuestion: function(){
      
      let bank = this.qdata.Bank;
      let index = bank.indexOf(this.question);

      this.question = bank[index - 1]
    },
    nextQuestion: function(){
      
      let bank = this.qdata.Bank;
      let index = bank.indexOf(this.question);

      this.question = bank[index + 1]
    },
    deleteQuestion: function(){
      
      let bank = this.qdata.Bank;
      let index = bank.indexOf(this.question);

      bank.splice(index, 1)

      this.question = bank[index - 1]

      for(let i = index; i < bank.length; i++){
        bank[i]["qid"] = bank[i]["qid"] - 1;
      }

    },
    getWidth: function(text){
      var mult = 7.5;
      let width = String(text).length * mult;

      if(width < 65){
        return "65px";
      }
      
      return width + "px";
    }
  },
  computed: {
    nextQuestionExists: function(){
      
      let bank = this.qdata.Bank;
      let index = bank.indexOf(this.question);

      if(bank[index + 1] != undefined){
        return true;
      }

      return false;
    },
    previousQuestionExists: function(){

      let bank = this.qdata.Bank;
      let index = bank.indexOf(this.question);

      if(bank[index - 1] != undefined){
        return true;
      }

      return false;
    },
    notFirstQuestion: function(){
      if(this.question.qid == 1){
        return false;
      }
      return true;
    },
    numQuestions: function(){
      return this.qdata.Bank.length;
    }
  },
  created: function(){

    if(this.validateQData(qbank)){
      this.qdata = qbank
      this.question = qbank.Bank[0]
    }
    else{
      alert("Must fix default qbank corruption.")
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 20px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

select {
  border-radius: 5px;
}

textarea {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  margin: 5px;
  border-radius: 10px;
  resize: none;
}

input {
  width: 10%;
  margin: 5px;
  border-radius: 5px;
}

input[type="number"]{
  width: 25px;
}

catbox {
  display: block;
  margin-left: 10%;
  margin-right: 15%;
  background-color: #EEEEEE;
  width: 75%;
  height: 100%;
  border-radius: 5px;
  padding: 10px;
}

.add{
  background-color:#006400;
}

.remove{
  background-color:#8B0000;
}

.remove:hover{
  background-color: #9C0000;
}

.export{
  background-color:#4682B4;
}

div {
  transition: transform 1s;
}

.big-container{
  width: 100%;
  display: block;
  position: initial;
  height: 420px;
}

button{
  color: white;
  background-color: #2c3e77;
  max-height: 150px;
  max-width: 150px;
}

.smooth-enter-active, .smooth-leave-active {
  transition: max-height .75s ease-in-out, max-width 250ms ease-in-out, opacity .5s ease-in-out,;
}

.catnum{
  font-weight: bold;
}

.catholder{
  overflow: scroll;
  scroll-behavior: smooth;
  max-height: 250px;
}

#qnum{
  font-size: 12px;
}


.fileContainer {
    overflow: hidden;
    position: relative;
}

.fileContainer {
    cursor: pointer;
}

.fileContainer [type=file] {
    cursor: pointer;
    display: block;
    font-size: 999px;
    filter: alpha(opacity=0);
    min-height: 100%;
    min-width: 100%;
    max-height: 30px;
    opacity: 0;
    position: absolute;
    right: 0;
    text-align: right;
    top: 0;
}

/* Example stylistic flourishes */

.fileContainer {
    background:#4682B4;
    color: white;
    border-radius: .5em;
    padding: .5em;
}



</style>
