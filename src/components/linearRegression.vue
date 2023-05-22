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
    // const W = tf.variable(tf.scalar(0.0));

    // const b = tf.variable(tf.scalar(0.0));

    // Define the linear regression model
    // const predictLinear = (x) => tf.add(tf.mul(x, W), b);

    // Define the loss function (mean squared error) for linear model
    const lossLinear = (predictions, labels) => tf.mean(tf.square(tf.sub(predictions, labels)));

    // Define the loss function (mean squared error) for non-linear model

    // Define the optimizer for linear model
    const learningRateLinear = 0.01;
    const optimizerLinear = tf.train.sgd(learningRateLinear);

    // Define the optimizer for non-linear model

    // Train the linear model
    const numIterations = 100;
    // for (let i = 0; i < numIterations; i++) {
    //   optimizerLinear.minimize(() => {
    //     const predictedValues = predictLinear(X_train);
    //     const currentLoss = lossLinear(predictedValues, y_train);
    //     return currentLoss;
    //   });
    // }

    // Train the non-linear model

    // Train the non-linear model

    // Make predictions using the linear model
    const predictedValuesLinear = predictLinear(X_test);

    // Convert tensors to arrays for plotting
    const X_testArray = Array.from(X_test.dataSync());
    const y_testArray = Array.from(y_test.dataSync());
    // const predictedValuesArray = Array.from(predictedValuesLinear.dataSync());
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
      // {
      //   x: X_testArray,
      //   y: predictedValuesArray,
      //   type: "scatter",
      //   name: "Predicted",
      // },
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
