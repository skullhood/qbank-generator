<template>
  <div class="test">

    <catbox>

      <h5>Categories</h5>

      <div v-for="category in qdata.Categories" v-bind:key="category.value">
        <input type="text" v-model="category.text" :style="{width: getWidth(category.text)}"> <button class="remove" @click="deleteCategory(category)">-</button>
      </div>
      
      <input type="text" v-model="newCat"> <button @click="addCategory" class="add">+</button>
    </catbox>

    <h4> 
    <button @click="previousQuestion" v-if="previousQuestionExists">Previous</button>
      Question {{question.qid}}
    <button @click="nextQuestion" v-if="nextQuestionExists">Next</button> 
    </h4> 

    Question: <br/><textarea type="textbox" v-model="question.question" style="width:40%"/> 
    
    <br/>
    <sub>Answer: Text / Category / Points</sub>

    <div v-for="answer in question.answers" v-bind:key="answer.key"> 
      <input type="text" v-model="answer.text"> 
      <select v-model="answer.category">
        <option v-for="category in qdata.Categories" v-bind:key="category.value" v-bind:value="category.value">
          {{category.text}} 
        </option>
      </select>
      <input type="number" name="points" v-model="answer.points">

      <button class="add" @click="addAnswer(answer)">+</button>
      <button class="remove" @click="deleteAnswer(answer)">-</button>
    
    </div>

    <button class="add" @click="addQuestion">Add Question</button>
    <button class="remove" v-if="notFirstQuestion" @click="deleteQuestion">Delete Question</button>
    <button class="export" @click="exportJson" style="margin-top: 100px">Export Json</button>
    
  </div>
  
</template>

<script>
import qbank from "../assets/data.json"

export default {
  name: 'QView',
  data() { return {
        qdata: Object,
        question: Object,
        newCat: new String,
        lGUID: String
      }
  },
  methods: {
    validateQData: function(){
      let cats = this.qdata.Categories;
      let quests = this.qdata.Bank;

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

      //Check qbank structure
      return true;
    },
    exportJson: function(){
      var dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.qdata, null, 2));
      var downloadAnchorNode = document.createElement('a');
      downloadAnchorNode.setAttribute("href",     dataStr);
      downloadAnchorNode.setAttribute("download", "questionBank.json");
      document.body.appendChild(downloadAnchorNode); // required for firefox
      downloadAnchorNode.click();
      downloadAnchorNode.remove();
    },
    addAnswer: function(ansClicked){

      let index = this.question.answers.indexOf(ansClicked) + 1;

      this.question.answers.splice(index,
        0,
        {
        "key" : index + 1,
        "text" : "New Answer",
        "points" : 4,
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
      alert("hello")
    },
    deleteCategory: function(category){
      
      let index = this.qdata.Categories.indexOf(category);

      this.qdata.Categories.splice(index, 1)

      if (this.question.answers[index].key != 0){
        for (let i = index; i < this.question.answers.length; i++ ){
          this.question.answers[i].key = this.question.answers[i].key - 1;
        }
      }

      //go through every question and 
    },
    addQuestion: function(){
      let bank = this.qdata.Bank;
      let qs_index = bank.length

      bank.splice(qs_index,
        0,
        {
                "qid" : qs_index + 1,
                "question" : "Question Text",
                "answers" : [
                    {
                        "key" : 1,
                        "text" : "New Answer 1",
                        "points" : 4,
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
      alert("DELETE");
    },
    getWidth: function(test){
      alert(test)
      return "30px";
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
    }
  },
  created: function(){
    this.qdata = qbank
    this.question = qbank.Bank[0]

    if(!this.validateQData()){
      this.qdata = null;
      this.question = null;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
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
  margin: 10px;
  border-radius: 5px;
}

input[type="number"]{
  width: 4%;
}

catbox {
  display: block;
  margin-left: 20%;
  margin-right: 20%;
  background-color: #EEEEEE;
  width: 60%;
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

</style>
