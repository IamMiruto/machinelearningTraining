<template>
  <div>
    <h1>Time</h1>
  </div>
  <div class="game">
    <div class="generation"></div>
    <span id="mouseX">:</span>
    <span id="mouseY"></span>
  </div>
</template>
<script>
/* eslint-disable */
import "../nn.js";
import "babel-polyfill";
import * as tf from "@tensorflow/tfjs";
// import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";

import { CANVAS_WIDTH, CANVAS_HEIGHT } from "../game/constants";
import Runner from "../game/Runner";
function convertStateToVector(state) {
  // allways return the same size vector
  if (state) {
    return [state.obstacleX / CANVAS_WIDTH, state.obstacleWidth / CANVAS_WIDTH, state.speed / 100];
  }
  return [0, 0, 0];
}

export default {
  name: "dinoTime",
  data() {
    return {};
  },

  mounted() {
    console.log("mounted");
    //print tf backend
    console.log("tf backend", tf.getBackend());
    // initializeTensorFlow();
    let runner = null;
    // initial setup for the game the  setup function is called when the dom gets loaded

    function setup() {
      // Initialize the game Runner.
      runner = new Runner(".game", {
        DINO_COUNT: 1,
        onReset: handleReset,
        onCrash: handleCrash,
        onRunning: handleRunning,
        speed: 100,
      });

      // increase the speed of the game

      // Set runner as a global variable if you need runtime debugging.
      window.runner = runner;
      // Initialize everything in the game and start the game.
      runner.init();
    }
    // variable which tells whether thethe game is being loaded for the first time i.e. not a reset

    let firstTime = true;

    function handleReset(dinos) {
      // running this for single dino at a time
      // console.log(dinos);

      const dino = dinos[0];
      // if the game is being started for the first time initiate
      // the model and compile it to make it ready for training and predicting
      if (firstTime) {
        firstTime = false;
        // creating a tensorflow sequential model
        dino.model = tf.sequential();

        dino.model.add(
          tf.layers.dense({
            inputShape: [5],
            activation: "relu",
            units: 6,
          })
        );
        dino.model.add(
          tf.layers.dense({
            inputShape: [5],
            activation: "relu",
            units: 16,
          })
        );
        dino.model.add(
          tf.layers.dense({
            inputShape: [5],
            activation: "relu",
            units: 16,
          })
        );
        dino.model.add(
          tf.layers.dense({
            inputShape: [5],
            activation: "relu",
            units: 16,
          })
        );

        // Increase the number of units in the output layer
        dino.model.add(
          tf.layers.dense({
            activation: "sigmoid",
            units: 2,
          })
        );

        // Compile the model with adjusted learning rate
        dino.model.compile({
          loss: "meanSquaredError",
          optimizer: tf.train.adam(0.04),
        });

        // object which will containn training data and appropriate labels
        dino.training = {
          inputs: [],
          labels: [],
        };
      } else {
        // hello from here
        // Train the model before restarting. asdfasdfasdf
        // log into console that mo sdfdel will now be trained
        console.info("Training");
        //increase the game ssdpeed by 10%
        // convert the inputs and labels to tenassor2d format and  then training the model
        console.info(tf.tensor2d(dino.training.inputs));

        //the prediction is just making the dino jump all the time
        dino.model.fit(tf.tensor2d(dino.training.inputs), tf.tensor2d(dino.training.labels));

        //print status of the training
        console.info("Training complete");

        //every time use the last state of the dino as the first state of the dino
      }
    }

    /**
     * documentation
     * @param {object} dino
     * @param {object} state
     * returns a promise resolved with an action
     */
    let stepBeforeCrash = [];
    let lebelBeforeCrash = [];

    function handleRunning(dino, state) {
      // keepp pushing state to stepBeforeCrash
      //get the complete state of the game
      stepBeforeCrash.push(state);
      let distancetToNextObstacle = [];
      //if the length of stepBeforeCrash is greater than 10 then remove the oldest element
      if (stepBeforeCrash.length > 10) {
        stepBeforeCrash.shift();
      }
      let horizon = runner.horizon;
      let obstacles = horizon.obstacles;

      for (let i = 0; i < 2; i++) {
        //get xPos of the obstacle and its width
        if (obstacles[i]) {
          let obstacleX = obstacles[i].xPos;
          distancetToNextObstacle.push(obstacleX);
        }
      }

      state.distanceToObstacle = distancetToNextObstacle[0];
      state.distanceToNextObstacle = distancetToNextObstacle[1];
      //feed information to the model that the dino is running fine

      return new Promise((resolve) => {
        let action = 0; // variable for action 1 for jump 0 for not
        const prediction = dino.model.predict(tf.tensor2d([convertStateToVector(state)]));
        const predictionPromise = prediction.data();
        predictionPromise.then((result) => {
          //choose the action with the highest probability as the action and return it
          if (result[0] > result[1]) {
            action = 0;
          } else {
            action = 1;
          }
          resolve(action);
        });
      });
    }
    /**
     *
     * @param {object} dino
     * handles the crash of a dino before restarting the game
     *
     */

    function handleCrash(dino) {
      let input = null;
      let label = null;
      //
      let distancetToNextObstacle = 0;
      // find the distance to the next obstacl
      let horizon = runner.horizon;
      let obstacles = horizon.obstacles;

      for (let i = 0; i < obstacles.length; i++) {
        if (obstacles[i].x > dino.xPos) {
          distancetToNextObstacle = obstacles[i].x - dino.xPos;
          break;
        }
      }
      //add this to the step before crash
      let step = stepBeforeCrash[stepBeforeCrash.length - 5];
      step.distanceToNextObstacle = distancetToNextObstacle;
      if (dino.jumping) {
        //use 2 step before crash
        input = convertStateToVector(step);
        label = [1, 0];
      } else {
        input = convertStateToVector(step);
        label = [0, 1];
      }
      // push the new input to the training set
      dino.training.inputs.push(input);
      // push the label to labels
      dino.training.labels.push(label);
    }

    /**
     *
     * @param {object} state
     * returns an array
     * converts state to a feature scaled array
     */
    function convertStateToVector(state) {
      if (state) {
        return [
          state.obstacleX,
          state.obstacleWidth,
          state.speed,
          state.distanceToNextObstacle,
          state.distanceToObstacle,
        ];
        // return a vector including the distance of the obstacle from the dino and the speed of the game
      }
      return [0, 0, 0, 0, 0];
    }
    // call setup on loading content
    document.addEventListener("DOMContentLoaded", setup);

    // show mouse position on screen for debugging
    document.addEventListener("mousemove", (e) => {
      document.getElementById("mouseX").innerHTML = e.clientX;
      document.getElementById("mouseY").innerHTML = e.clientY;
    });
  },
};
</script>
