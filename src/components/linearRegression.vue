<template>
  <div>
    <h1>Linear regression</h1>
  </div>
  <div class="game">
    <div class="generation"></div>
  </div>
</template>
<script>
/* eslint-disable */
import "../nn.js";
import "babel-polyfill";
import * as tf from "@tensorflow/tfjs";
// import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";
import Plotly from "plotly.js-dist";

export default {
  name: "dinoTime",
  data() {
    return {};
  },
  methods: {
    // incrementGeneration() {
    //   // ...
    //   if (this.generation < 2) {
    //     this.dino.model = tf.sequential();
    //     // Add layers
    //     this.dino.model.add(
    //       tf.layers.dense({
    //         inputShape: [3],
    //         units: 6,
    //         activation: "sigmoid",
    //       })
    //     );
    //     this.dino.model.add(
    //       tf.layers.dense({
    //         inputShape: [6],
    //         units: 2,
    //         activation: "sigmoid",
    //       })
    //     );
    //     this.dino.model.compile({
    //       loss: "meanSquaredError",
    //       optimizer: tf.train.adam(0.1),
    //     });
    //     // Train the model
    //     // ...
    //   } else {
    //     // Convert the inputs and outputs to tensors
    //     const inputs = tf.tensor2d(this.dino.training.inputs);
    //     //outouts are labels
    //     // const labels = tf.tensor2d(this.dino.training.outputs);
    //     const values = this.dino.training.outputs.map((output) => {
    //       return [output[0], output[1]];
    //     });
    //     // c
    //     const tensor = tf.tensor2d(values, [values.length, 2]);
    //     console.log("values", tensor);
    //     // // Train the model
    //     this.dino.model.fit(inputs, tensor);
    //     //consolog training output
    //     console.log("inputs", tensor);
    //   }
    // },
    // handleCrash() {
    //   const jumpState = window.runner.tRex.jumping;
    //   const duckState = window.runner.tRex.ducking;
    //   const obstaclesOnScreen = window.runner.horizon.obstacles;
    //   const nextObstacle = obstaclesOnScreen[0];
    //   const obstacleX = nextObstacle.xPos;
    //   const obstacleWidth = nextObstacle.width;
    //   const speed = window.runner.currentSpeed;
    //   const state = {
    //     jumpState,
    //     duckState,
    //     obstacleX,
    //     obstacleWidth,
    //     speed,
    //   };
    //   const vector = convertStateToVector(state);
    //   if (jumpState) {
    //     this.dino.training.inputs.push(vector);
    //     this.dino.training.outputs.push([1, 0]);
    //   } else {
    //     this.dino.training.inputs.push(vector);
    //     this.dino.training.outputs.push([0, 1]);
    //   }
    //   this.generation++;
    // },
    // handleRunning() {
    //   //get the state of dino get state from the div
    //   const jumpState = window.runner.tRex.jumping;
    //   const duckState = window.runner.tRex.ducking;
    //   const obstaclesOnScreen = window.runner.horizon.obstacles;
    //   //find the distance between the dino and the next obstacle and the width of the obstacle and the position of the obstacle
    //   const nextObstacle = obstaclesOnScreen[0];
    //   const obstacleX = nextObstacle.xPos;
    //   const obstacleWidth = nextObstacle.width;
    //   const speed = window.runner.currentSpeed;
    //   const state = {
    //     jumpState,
    //     duckState,
    //     obstacleX,
    //     obstacleWidth,
    //     speed,
    //   };
    //   if (!jumpState && this.generation > 1) {
    //     const vector = convertStateToVector(state);
    //     //predict the action
    //     // const input = tf.tensor2d([vector]);
    //     // const input = tf.tensor2d([vector]);
    //     // const output = this.dino.model.predict(input);
    //     // console.log("input", output);
    //   }
    // },
  },
  mounted() {
    // Import TensorFlow.js library
    // const tf = require('@tensorflow/tfjs');

    const tf = require("@tensorflow/tfjs");

    const X = tf.linspace(0, 10, 100);
    const noise = tf.randomNormal([100]);
    const y = tf.add(tf.mul(4, X), tf.add(2, noise));

    // Split the data into training and testing sets
    const X_train = X.slice([0], [80]);
    const y_train = y.slice([0], [80]);
    const X_test = X.slice([80], [20]);
    const y_test = y.slice([80], [20]);

    // Create TensorFlow variables for weights and bias
    const W = tf.variable(tf.scalar(0.0));

    const b = tf.variable(tf.scalar(0.0));

    // Define the linear regression model
    const predictLinear = (x) => tf.add(tf.mul(x, W), b);

    // Define the non-linear regression model
    const predictNonLinear = (x) => tf.add(tf.add(tf.mul(W1, tf.square(x)), tf.mul(W2, x)), b);

    // Define the loss function (mean squared error) for linear model
    const lossLinear = (predictions, labels) => tf.mean(tf.square(tf.sub(predictions, labels)));

    // Define the loss function (mean squared error) for non-linear model
    const lossNonLinear = (predictions, labels) => tf.mean(tf.square(tf.sub(predictions, labels)));

    // Define the optimizer for linear model
    const learningRateLinear = 0.01;
    const optimizerLinear = tf.train.sgd(learningRateLinear);

    // Define the optimizer for non-linear model
    const learningRateNonLinear = 0.01;
    const optimizerNonLinear = tf.train.sgd(learningRateNonLinear);

    // Train the linear model
    const numIterations = 100;
    for (let i = 0; i < numIterations; i++) {
      optimizerLinear.minimize(() => {
        const predictedValues = predictLinear(X_train);
        const currentLoss = lossLinear(predictedValues, y_train);
        return currentLoss;
      });
    }

    // Train the non-linear model

    // Train the non-linear model

    // Make predictions using the linear model
    const predictedValuesLinear = predictLinear(X_test);

    // Convert tensors to arrays for plotting
    const X_testArray = Array.from(X_test.dataSync());
    const y_testArray = Array.from(y_test.dataSync());
    const predictedValuesArray = Array.from(predictedValuesLinear.dataSync());
    // console.log("X_testArray", predictedValuesNonLinearArray);

    //create a new div with id chart

    // plot the data using plotly
    const data = [
      {
        x: X_testArray,
        y: y_testArray,
        type: "scatter",
        name: "Ground Truth",
      },
      {
        x: X_testArray,
        y: predictedValuesArray,
        type: "scatter",
        name: "Predicted",
      },
    ];

    const layout = {
      title: "Linear Regression",
      xaxis: {
        title: "X",
      },
      yaxis: {
        title: "Y",
      },
    };

    Plotly.newPlot("chart", data, layout);

    // Print the predicted values and the ground truth values
    console.log("Predicted Values:");
    //print test values and predicted values
    for (let i = 0; i < X_testArray.length; i++) {
      console.log(`X: ${X_testArray[i]}, Ground Truth: ${y_testArray[i]}, Predicted: ${predictedValuesArray[i]}`);
    }

    //do an accuracy test
    const accuracy = tf.losses.meanSquaredError(y_test, predictedValuesLinear);
    console.log("accuracy", accuracy);
    //accuracy in percentage
    console.log("accuracy", accuracy.dataSync()[0].toFixed(2));
    //plot the accuracy in the center of the plot
    const accuracyDiv = document.createElement("div");
    accuracyDiv.innerHTML = `Accuracy: ${accuracy.dataSync()[0].toFixed(2)}`;

    document.getElementById("chart").appendChild(accuracyDiv);
  },
};
</script>
