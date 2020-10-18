<template>
  <div id="app">
    <div class="app-page">
      <h2 class="app-page__title">Лабораторная работа № 3. Метод Брауна-Робинсона</h2>
      <h4 class="app-page__terminate-message" v-if="terminateForcibly">Количество итераций превысило 10000 или что-то пошло не так</h4>
      <!-- <template v-if="!isStart"> -->
        <span>Введите размерность платежной матрицы</span>
        <div class="app-page__selectors-wrapper">
          <select @change = "isConfirmDemension = true" class="app-page__selector-item" title="Количество" v-model="columns">
            <option v-for="n in 9" :key="n">{{n+1}}</option>
          </select>
          <select @change = "isConfirmDemension = true" class="app-page__selector-item" v-model="rows">
            <option v-for="n in 9" :key="n">{{n+1}}</option>
          </select> 
          
        </div>
        строк {{rows}} -  столбцов {{columns}}

        <section v-if="matrix && matrix.length" class="app-page__input-matrix">
          <div v-for="(element,key1) in matrix" :key="key1" class="app-page__matrix-wrapperr">
            <input type="number" v-for="(element1, key2) in element" :key="key2" v-model="matrix[key1][key2]" />
          </div>
        </section>
        <button class="btn" v-if="isConfirmDemension" @click="setDemension">Подтвердить размерность</button>

        <!-- <button class="btn" @click="startInputMatrix">Далее</button> -->
      <!-- </template> -->

      <!-- <template v-else> -->
        
        <div>
          <button class="btn" @click="isIteration=true">Ввести количество итераций</button>
          <button class="btn" @click="isIteration=false">Ввести епсилон</button>
        </div>
        <div v-if="isIteration">
          <span>k = </span>
          <input type="number" min="0" @input="epsilon=null" v-model="iteration"/>
        </div>
        <div v-else>
          <span>епсилон = </span>
          <input type="number" @epsilon="iteration=null" v-model="epsilon"/>
        </div>
        <button class="btn" @click="startAlgorithm">Запустить алгоритм</button>

        <div class="app-page__data">
          <div class="app-page__data-item">
            <h5 class="app-page__data-title">Смешанные стратегии</h5>
            <div>{{probabilityVectorI}}</div>
            <div>{{probabilityVectorJ}}</div>
          </div>
          <div v-if="isStart" class="app-page__data-item">
            <h5 class="app-page__data-title">Оценки</h5>
            <div>Няжняя граница {{(+(outputData[outputData.length-1]).M).toFixed(3)}}</div>
            <div>Верхняя граница {{(+(outputData[outputData.length-1]).V).toFixed(3)}}</div>
         </div>
        </div>
        <template v-if="isStart">
          <div class="app-page__table-wrapper">
            <table class="app-page__table">
              <tr class="app-page__table-header">
                <td>K</td>
                <td>j</td>
                <td v-for="key4 in arrG.length" :key="key4">g{{key4}}</td>
                <td>M</td>
                <td>V</td>
                <td v-for="key3 in arrH.length" :key="key3+arrG.length">h{{key3}}</td>
                <td>i</td>
              </tr>
              <tr v-for="(item, key5) in outputData" :key="key5">
                <td>{{item.k}}</td>
                <td>{{item.j}}</td>
                <td v-for="(g,key1) in item.arrG" :key="key1">{{+g.toFixed(3)}}</td>
                <td>{{+(item.M).toFixed(3)}}</td>
                <td>{{+(item.V).toFixed(3)}}</td>
                <td v-for="(h,key2) in item.arrH" :key="key2+item.arrG.length">{{+h.toFixed(3)}}</td>
                <td>{{item.i}}</td>
              </tr>
            </table>
          </div>
        </template>

      <!-- </template> -->
      
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
   data(){
        return {
            isStart: false,
            isConfirmDemension: false,
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
            terminateForcibly: false,
            saddlePoint: null,
            probabilityVectorI: [],
            probabilityVectorJ: []
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
      // this.isStart = !this.isStart;
    },
    setDemension() {
      let arr = new Array(parseInt(this.columns));

      for (var i = 0; i < arr.length; i++) {
         arr[i] = (new Array(parseInt(this.rows)));
      }   

      this.matrix = arr;
    },
    startAlgorithm() {
      this.isStart = !this.isStart;
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
        if(count > 10000) {
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
        if(count > 500) {
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
      
      for(let i = 1; i < arr.length + 1; i++) {
        if(arr[i - 1] != arr[i]){
          letArrProb.push(count/arr.length);
          if(arr[i - 1] != arr[i] - 1){
            letArrProb.push(0); // нулевая вероятность, такой индекс не встречался
          }
          count = 1;
        }
        else if(i === arr.length-1 && count === 1 && letArrProb.length < len){
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

      if(index === "i"){
        this.probabilityVectorI = letArrProb;
      }else{
        this.probabilityVectorJ = letArrProb;
      }
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
.btn {
    margin: 10px 15px;
    padding: 10px 8px;
    border: none;
    max-width: 280px;
    border-radius: 7px;
    font-size: 14px;
    font-weight: 700;
    background-color: lightseagreen;
    line-height: 1;
    cursor: pointer;
    outline: 0;
}
.btn:hover {
  color: #fff;
}
input {
  width: 50px;
  margin: 4px;
  border: 1px solid #707070;
}
.app-page__table-wrapper {
  padding: 0;
  width: 100%;
  display: flex;
  justify-content: center;
  margin: 20px;
}
.app-page__table {
  font-size: 15px;
  border-radius: 10px;
  border-spacing: 0;
  text-align: center;
}
.app-page__table-header {
  background-color: lightsteelblue;
}
table {
  border-collapse: collapse;
}

th, td {
  border: 1px solid black;
}
table {
  border: 1px solid black;
}
table {
  width: 80%;
}

th {
  height: 20px;
}
th {
  text-align: left;
}
td {
  height: 20px;
  vertical-align: bottom;
}
th, td {
  padding: 5px;
  text-align: left;
}
.app-page__data {
  display: flex;
  justify-content: center;
}
.app-page__data-title {
  margin: 5px 0;
}
.app-page__data-item {
  margin-left: 30px;
}
</style>
