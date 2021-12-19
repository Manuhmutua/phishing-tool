<template>
  <div>
    <div class="predict-controls">
      <h2 class="section col-sm-1">Predicting</h2>
      <input class="field element" v-model="valueToPredict" type="text" placeholder="Enter email to test"><br>
      <div class="element" v-html="predictedValue"></div>
      <button class="element button--green" v-on:click="predict" :disabled="!modelReady">Predict</button>
    </div>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs';
// import * as use from '@tensorflow-models/universal-sentence-encoder';
import axios from 'axios';

export default {
  data() {
    return {
      modelReady: false,
      predictedValue:'Click on train!',
      valueToPredict: ''
    }
  },
  mounted() {
    let that = this;
    async function loadModel() {
      that.model = await tf.loadLayersModel('http://localhost:8080/shared/model/model.json');
      that.modelReady = true;
      that.predictedValue = 'Ready for making predictions';
    }
    loadModel();
  },
  methods: {
    predict() {
      let inputs = [this.valueToPredict];
      axios.get('http://localhost:8080/vocabulary.txt').then(response => {
        let v = response.data.split(/\r?\n/);
        let t = inputs[0].split(' ');
        const parse = () => v.map((w,i) => t.reduce((a, b) => b === w ? i++ : a , 0));
        const tfarray = tf.tensor([parse()]);
        const prediction = this.model.predict(tfarray);
        const values = prediction.dataSync();
        const arr = Array.from(values);
        console.log(arr.reduce((a, b) => a + b, 0)/arr.length)
        if ((10 + arr.reduce((a, b) => a + b, 0)/arr.length) >= 1){
           this.predictedValue = "Phishing";
        }else {
           this.predictedValue = "Not Phishing";
        }
      })
    }
  }
}
</script>

<style>
.field, .field-label {
  height: 30px;
  float: left;
  padding: 0px 15px;
  float: left;
  width: 50%;
}
.field {
  border-radius: 0px 5px 5px 0px;
  border: 1px solid #eee;
  margin-bottom: 15px;
  height: 40px;
}
.col-sm-1:after {
    content: "";
    display: table;
    clear: both;
}
.section, .field-label {
  text-align: left;
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
  font-weight: 100;
}
.field-label {
  font-weight: 700;
}
.button-add-example {
  width: 100%;
  margin-bottom: 10px;
}
.button-train {
  width: 100%;
}
.predict-controls {
  padding-top: 30px;
  padding-bottom: 30px;
}
.predict-controls .element {
  width: 50%;
  display: block;
}
button {
  margin-top: 10px;
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
  font-weight: 700;
}
</style>