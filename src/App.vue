<template>
  <div id="app">
    <div class="app-page">
      <h2 class="app-page__title">Лабораторная работа № 3. Метод Брауна-Робинсона</h2>
      <h4 class="app-page__terminate-message" v-if="terminateForcibly">Количество итераций превысило 10000 или что-то пошло не так</h4>
      <h4 class="app-page__terminate-message" v-if="naN">Пожалуйста, проверьте, все ли поля матрицы заполнены числами.</h4>
        <button class="btn" type="info" @click="load" width="100%">Загрузить</button>
        <input id="downloadData" type="file" hidden ref="file" @change="loadData"/>
        <div>Введите размерность платежной матрицы</div>
        <div class="app-page__selectors-wrapper">
          <select @change = "isConfirmDemension = true" class="app-page__selector-item" title="Количество" v-model="columns">
            <option v-for="n in 9" :key="n">{{n+1}}</option>
          </select>
          <select @change = "isConfirmDemension = true" class="app-page__selector-item" v-model="rows">
            <option v-for="n in 9" :key="n">{{n+1}}</option>
          </select> 
        </div>

        <section v-if="matrix && matrix.length" class="app-page__input-matrix">
          <div v-for="(element,key1) in matrix" :key="key1" class="app-page__matrix-wrapperr">
            <input type="number" v-for="(element1, key2) in element" :key="key2" v-model="matrix[key1][key2]" />
          </div>
        </section>
        <button class="btn" v-if="isConfirmDemension" @click="setDemension">Подтвердить размерность</button>

        <div>
          <button class="btn" @click="isIteration=true">Ввести количество итераций</button>
          <button class="btn" @click="isIteration=false">Ввести эпсилон</button>
        </div>
        <div v-if="isIteration">
          <span>k = </span>
          <input type="number" min="0" @input="epsilon=null" v-model="iteration"/>
        </div>
        <div v-else>
          <span>эпсилон = </span>
          <input type="number" @epsilon="iteration=null" v-model="epsilon"/>
        </div>
        <button class="btn" @click="startAlgorithm">Запустить алгоритм</button>
        <button :disabled="!isStart"  @click="clearField" class="btn">Очистить</button>
        <button :disabled="!isStart" @click="save" class="btn">Сохранить</button>
        <div id="saveContent">
          <div class="app-page__data">
            <div v-if="isStart" class="app-page__data-item">
            <div class="app-page__data-item">
              <h5 class="app-page__data-title">Смешанные стратегии</h5>
              <div>{{probabilityVectorI}}</div>
              <div>{{probabilityVectorJ}}</div>
            </div>
              <h5 class="app-page__data-title">Оценки</h5>
              <div id="1">Нижняя граница {{(+(outputData[outputData.length-1]).M).toFixed(3)}}</div>
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
        </div>     
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
            epsilon: 0.01,
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
            probabilityVectorI: [],
            probabilityVectorJ: [],
            naN: false,
        }
    },
    beforeMount:  function () {
       let arr = new Array(parseInt(this.columns));

      for (var i = 0; i < arr.length; i++) {
         arr[i] = (new Array(parseInt(this.rows))).fill(1);
      }   

      this.matrix = arr;
    },
  computed:{
    file(){
      return this.$refs['file'];
    }
  },
  methods: {
    setDemension() {
      let arr = new Array(parseInt(this.columns));

      for (var i = 0; i < arr.length; i++) {
         arr[i] = (new Array(parseInt(this.rows)));
      }   

      this.matrix = arr;
      console.log(arr);
    },
    startAlgorithm() {
      this.naN = false;
      if(this.isStart) {
        this.clearField();
      }
      this.paymentMatrix = [...this.matrix].map((item) => {
        return ([...item]);
      });

      for(let i = 0; i < this.paymentMatrix.length; i++) {
        this.paymentMatrix[i] = this.paymentMatrix[i].map((item) => {
          return parseInt(item)
        });
      }

      for(let i = 0; i < this.paymentMatrix.length; i++) {
        if((this.paymentMatrix[i].filter((item) => {
          return isNaN(item);
        })).length){
          this.naN = true;
        }
      }

      if(!this.naN) {
        this.isStart = !this.isStart;
        console.log(this.paymentMatrix);

        if(this.isIteration) {
          this.algorithmWithIteration();
        }else {
          this.algorithmWithEpsilon();
        }

        this.vectorOfProbabilities('i');
        this.vectorOfProbabilities('j');
      }
    },
    algorithmWithIteration() {
      this.outputData = [];
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
      this.outputData = [];
      let count = 1;
      this.M= null;
      this.V = null;

      this.arrG = (new Array(parseInt(this.columns))).fill(0);
      this.arrH = (new Array(parseInt(this.rows))).fill(0);

      while((parseFloat(this.epsilon) < Math.abs((this.M/count - this.V/count)) || this.M === null) && count < 10000)
      {
        let currentRow = this.paymentMatrix[this.currentIndexRow];
        let currentColumn = [];
        this.currentIndexRow = 0;
        this.currentIndexColumn = 0;
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

      let arrayObj = arr.reduce(function(acc, el) {
        acc[el] = (acc[el] || 0) + 1;
        return acc;
      }, {});

      for(let i = 0; i < len; i++) {
        if(arrayObj[i]){
          letArrProb.push(arrayObj[i]/this.outputData.length)
        }else{
          letArrProb.push(0);
        }
      }

      if(index === "i"){
        this.probabilityVectorI = letArrProb.map((item) => {return +item.toFixed(3)});
      }else{
        this.probabilityVectorJ = letArrProb.map((item) => {return +item.toFixed(3)});
      }
    },
    clearField(){
      this.isStart = false
      this.outputData = [];
      document.getElementById("downloadData").value = "";
      this.naN = false;
      this.terminateForcibly = false;
    },
    save() {
      this.hiddenElement = true;
      let txtData = document.getElementById("saveContent").innerText;
      this.download(txtData, '3лрДанные.txt', 'text/plain');
      this.hiddenElement = false;
    },
    download(content, fileName, contentType) {
    let a = document.createElement("a");
    let file = new Blob([content], {type: contentType});
    a.href = window.URL.createObjectURL(file);
    a.download = fileName;
    a.click();
    },
    load(){
      this.file.click();
    },
    loadData() {
      this.naN = false;
      let file = this.file.files[0];
      let reader = new FileReader();
      reader.readAsText(file);
      const program = this;
      reader.onload = function() {
          if(!reader.result) {
            console.log(this.result);
            program.naN = true;
          }else{
            console.log(this.result);
            let data = JSON.parse(reader.result);
            let isEqualLength = true;
            if(data && data.length) {
              let elemLen = data[0].length;
              for(let i = 1; i < data.length; i++) {
                if(data[i].length != elemLen) {
                  isEqualLength = false;
                }
              }
              if(isEqualLength && data.length){
              program.matrix = data;
              program.rows = data[0].length;
              program.columns = data.length;
              console.log(data);
              console.log(program.matrix, program.rows, program.columns);
            }else{
              program.naN = true;
            }
          }else{
            program.naN = true;
          }
        }
      };
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
