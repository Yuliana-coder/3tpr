<template>
  <div id="app">
    <div class="app-page">
      <h2 class="app-page__title">Лабораторная работа № 3. Метод Брауна-Робинсона</h2>
      <h4 class="app-page__terminate-message" v-if="terminateForcibly">Количество итераций превысило 10 или что-то пошло не так</h4>
      <!-- <template v-if="!isStart"> -->
        <span>Введите размерность платежной матрицы</span>
        <div class="app-page__selectors-wrapper">
          <select class="app-page__selector-item" title="Количество" v-model="rows">
            <option>2</option>
            <option>3</option>
            <option>4</option>
            <option>5</option>
            <option>6</option>
            <option>7</option>
            <option>8</option>
            <option>9</option>
            <option>10</option>
          </select>
          <select class="app-page__selector-item" v-model="columns">
            <option>2</option>
            <option>3</option>
            <option>4</option>
            <option>5</option>
            <option>6</option>
            <option>7</option>
            <option>8</option>
            <option>9</option>
            <option>10</option>
          </select> 
          
        </div>
        строк {{rows}} -  столбцов {{columns}}

        <section v-if="matrix && matrix.length" class="app-page__input-matrix">
          <div v-for="(element,key1) in matrix" :key="key1" class="app-page__matrix-wrapperr">
            <input v-for="(element1, key2) in element" :key="key2" v-model="matrix[key1][key2]" />
          </div>
        </section>

        <button @click="startInputMatrix">Далее</button>
      <!-- </template> -->

      <!-- <template v-else> -->
        <h4>Выберите</h4>
        
        <div>{{outputData.i}}</div>
        <div>{{outputData.j}}</div>
        <div>
          <button @click="isIteration=true">Ввести количество итераций</button>
          <button @click="isIteration=false">Ввести епсилон</button>
        </div>
        <div v-if="isIteration">
          <span>k = </span>
          <input @input="epsilon=null" v-model="iteration"/>
        </div>
        <div v-else>
          <span>епсилон = </span>
          <input @epsilon="iteration=null" v-model="epsilon"/>
        </div>
        <button @click="startAlgorithm">Запустить алгоритм</button>
        {{paymentMatrix}}

        <div v-for="(item, key) in outputData" :key="key">{{item}}</div>



      <!-- </template> -->
      
      <!-- <div v-for="(attribute, index) in attributes">
        <p>{{ attribute.name }}</p>
        <div v-for="(attributevalue,indexval) in attribute.attribute_values">
          <input v-model="attributes"  :value="attributevalue.id">
        </div>
</div> -->
      
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
   data(){
        return {
            isStart: false,
            rows: 2,
            columns: 2,
            matrix: null,
            isIteration: true,
            iteration: 3,
            epsilon: null,
            paymentMatrix: null,
            currentIndexRow: 0,
            currentIndexColumn: 0,
            arrG: [],
            arrH: [],
            M: null,
            V: null,
            rowIndMax: null,
            columnIndMin: null,
            outputData: [],
            terminateForcibly: false
        }
    },
    beforeMount:  function () {
       let arr = new Array(parseInt(this.columns));

      for (var i = 0; i < arr.length; i++) {
         arr[i] = (new Array(parseInt(this.rows))).fill(1);
      }   

      this.matrix = arr;
    },
  methods: {
    startInputMatrix() {
      this.isStart = !this.isStart;
      let arr = new Array(parseInt(this.columns));

      for (var i = 0; i < arr.length; i++) {
         arr[i] = (new Array(parseInt(this.rows)));
      }   

      this.matrix = arr;
    },
    startAlgorithm() {
      this.paymentMatrix = [...this.matrix].map((item) => {
        return ([...item]);
      });

      for(let i = 0; i < this.paymentMatrix.length; i++) {
        this.paymentMatrix[i] = this.paymentMatrix[i].map((item) => {
          return parseInt(item)
        });
      }

      if(this.isIteration) {
        this.algorithmWithIteration();
      }else {
        this.algorithmWithEpsilon();
      }

      this.vectorOfProbabilities('i');
      this.vectorOfProbabilities('j');
    },
    algorithmWithIteration() {
      let count = 1;

      this.arrG = (new Array(parseInt(this.columns))).fill(0);
      this.arrH = (new Array(parseInt(this.rows))).fill(0);

      while(this.iteration >= count)
      {
        let currentRow = this.paymentMatrix[this.currentIndexRow];
        let currentColumn = [];
        for(let i = 0; i < this.paymentMatrix.length; i++) {
          currentColumn.push(this.paymentMatrix[i][this.currentIndexColumn]);
        }

        this.currentColumn = currentColumn;
        for(let i = 0; i < currentColumn.length; i++) {
          this.arrG[i] = this.arrG[i] + currentColumn[i];
        }

        for(let i = 0; i < currentRow.length; i++) {
          this.arrH[i] = this.arrH[i] + currentRow[i];
        }

        this.M = Math.min.apply(null,this.arrG);
        this.V = Math.max.apply(null,this.arrH);

        this.outputData.push({
              k: count,
              j: this.currentIndexColumn,
              arrG: [...this.arrG],
              M: this.M/count,
              V: this.V/count,
              arrH: [...this.arrH],
              i: this.currentIndexRow
        });

        this.currentIndexColumn = this.arrH.indexOf(this.V);
        this.currentIndexRow =  this.arrG.indexOf(this.M);
        count = count + 1;
        if(count > 25) {
          this.terminateForcibly = true;
        }
      }
    },
    algorithmWithEpsilon() {
      let count = 1;

      this.arrG = (new Array(parseInt(this.columns))).fill(0);
      this.arrH = (new Array(parseInt(this.rows))).fill(0);

      while((parseFloat(this.epsilon) < Math.abs((this.M/count - this.V/count)) || this.M === null) && count < 100)
      {
        let currentRow = this.paymentMatrix[this.currentIndexRow];
        let currentColumn = [];
        for(let i = 0; i < this.paymentMatrix.length; i++) {
          currentColumn.push(this.paymentMatrix[i][this.currentIndexColumn]);
        }

        this.currentColumn = currentColumn;
        for(let i = 0; i < currentColumn.length; i++) {
          this.arrG[i] = this.arrG[i] + currentColumn[i];
        }

        for(let i = 0; i < currentRow.length; i++) {
          this.arrH[i] = this.arrH[i] + currentRow[i];
        }

        this.M = Math.min.apply(null,this.arrG);
        this.V = Math.max.apply(null,this.arrH);

        this.outputData.push({
              k: count,
              j: this.currentIndexColumn,
              arrG: [...this.arrG],
              M: this.M/count,
              V: this.V/count,
              arrH: [...this.arrH],
              i: this.currentIndexRow
        });

        this.currentIndexColumn = this.arrH.indexOf(this.V);
        this.currentIndexRow =  this.arrG.indexOf(this.M);
        count = count + 1;
        if(count > 25) {
          this.terminateForcibly = true;
        }
      }
    },
    vectorOfProbabilities(index) {
      let arr = [...this.outputData].map((item) => {
        return item[index];
      });

      let len;
      if(index === "i"){
        len = parseInt(this.columns );
      }else{
        len = parseInt(this.rows);
      }

      let letArrProb = [];
      arr = arr.sort();

      console.log(arr);

      let count = 1;
      
      for(let i = 1; i < arr.length; i++) {
        if(arr[i - 1] != arr[i]){
          letArrProb.push(count/arr.length);
          if(arr[i - 1] != arr[i] - 1){
            letArrProb.push(0); // нулевая вероятность, такой индекс не встречался
          }
          count = 1;
        }else if(i === arr.length - 1 && count != 0){
          count = count + 1
          letArrProb.push(count/arr.length);
        }
        else{
          count = count + 1;
        }
      }

      if(letArrProb.length < len){
        letArrProb.push(count/arr.length); // если нет последнего индекса, то он встречался, 0 вреоятность
        console.log(len);
      }

      console.log(letArrProb);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.app-page__selectors-wrapper {
  margin-top: 15px;
  margin-bottom: 7px;
}
.app-page__selector-item {
  margin: 0 5px;
}
.app-page__terminate-message {
  color:red;
}
</style>
