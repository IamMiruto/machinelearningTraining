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

      window.runner = runner;
      // Initialize everything in the game and start the game.
      runner.init();
    }

    let firstTime = true;
    function handleReset(dinos) {
      // running this for single dino at a time
      // console.log(dinos);

      const dino = dinos[0];
      // if the game is being started for the first time initiate
      // the model and compile it to make it ready for training and predicting
      if (firstTime) {
        // object which will containn training data and appropriate labels
        dino.training = {
          inputs: [],
          labels: [],
        };
      } else {
        // hello from here
        // Train the model before restarting. asdfasdfasdf
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

    function handleRunning(dino, state) {}
    /**
     *
     * @param {object} dino
     * handles the crash of a dino before restarting the game
     *
     */

    function handleCrash(dino) {}

    /**
     *
     * @param {object} state
     * returns an array
     * converts state to a feature scaled array
     */
    function convertStateToVector(state) {}
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
