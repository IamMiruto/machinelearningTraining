<template>
  <div>
    <H1>Prediction</H1>
    <table>
      <thead>
        <tr>
          <th>Ticket ID</th>
          <th>Number of People</th>
          <th>Gender</th>
          <th>Likes</th>
          <th>Allocation</th>
        </tr>
      </thead>
      <tbody>
        <!-- Loop through the allocation data and generate table rows -->
        <tr v-for="(entry, index) in allocationData" :key="index">
          <td>{{ entry.ticketId }}</td>
          <td>{{ entry.numberofpeople }}</td>

          <td>{{ entry.allocation }}</td>
        </tr>
      </tbody>
    </table>
    <!-- table for input data -->
    <h1>Input Data</h1>
    <table>
      <thead>
        <tr>
          <th>Ticket ID</th>
          <th>Number of People</th>
          <th>Gender</th>
          <th>Likes</th>
        </tr>
      </thead>
      <tbody>
        <!-- Loop through the allocation data and generate table rows -->
        <tr v-for="(entry, index) in inputData" :key="index">
          <td>{{ entry.ticketid }}</td>
          <td>{{ entry.numberofpeople }}</td>
          <td>{{ entry.gender }}</td>
          <td>{{ entry.likes }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
/* eslint-disable */
import * as tf from "@tensorflow/tfjs";

export default {
  data() {
    return {
      inputData: [], // Placeholder for the input data
      allocationData: [], // Placeholder for the allocation data
    };
  },
  mounted() {
    // Fetch the allocation data and update the allocationData property
    // Replace this with your actual logic to fetch and update the data
    this.fetchAllocationData();
    this.fetchInputData();
  },
  methods: {
    fetchInputData() {
      const baseObject = {
        ticketid: 1,
        numberofpeople: 5,
        gender: "male",
        likes: "sightseeing",
      };

      const generatedObjects = [];

      for (let i = 0; i < 20; i++) {
        const generatedObject = {
          ticketid: Math.random(1, 1000) * 1000,
          numberofpeople: getRandomNumber(1, 10), // Generate a random number between 1 and 10
          gender: getRandomGender(),
          likes: getRandomLikes(),
        };

        generatedObjects.push(generatedObject);
      }

      // Helper function to generate a random number between min and max (inclusive)
      function getRandomNumber(min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
      }

      // Helper function to randomly select a gender
      function getRandomGender() {
        const genders = ["male", "female"];
        const randomIndex = Math.floor(Math.random() * genders.length);
        return genders[randomIndex];
      }

      // Helper function to randomly select a likes category
      function getRandomLikes() {
        const likes = ["sightseeing", "adventure", "nature", "food", "music"];
        const randomIndex = Math.floor(Math.random() * likes.length);
        return likes[randomIndex];
      }

      console.log("Hello from generated onkects", generatedObjects);

      this.inputData = generatedObjects;
    },
    fetchAllocationData() {
      const availableBalloons = [
        { name: "Balloon A", capacity: 2, likes: "sightseeing" },
        { name: "Balloon B", capacity: 4, likes: "adventure" },
        { name: "Balloon C", capacity: 6, likes: "nature" },
        { name: "Balloon D", capacity: 8, likes: "food" },
        { name: "Balloon E", capacity: 10, likes: "music" },
      ];

      let sampleInput = [
        { numberofpeople: 5, likes: "sightseeing", balloon: "Balloon A" },
        { numberofpeople: 5, likes: "sightseeing", balloon: "Balloon A" },
        { numberofpeople: 5, likes: "sightseeing", balloon: "Balloon A" },
        { numberofpeople: 5, likes: "sightseeing", balloon: "Balloon A" },
      ];

      let allocationData = [];

      // Convert input data to tensor
      const inputTensor = tf.tensor2d(
        sampleInput.map((ticket) => [ticket.numberofpeople]),
        [sampleInput.length, 1]
      );

      // Normalize the input tensor (optional)
      const normalizedInputTensor = tf.div(inputTensor, tf.scalar(10));

      // Define the TensorFlow neural network model
      const model = tf.sequential();
      model.add(tf.layers.dense({ units: 16, activation: "relu", inputShape: [1] }));
      model.add(tf.layers.dense({ units: 16, activation: "relu", useBias: true }));
      model.add(tf.layers.dense({ units: 16, activation: "relu", useBias: true }));
      model.add(tf.layers.dense({ units: 16, activation: "relu", useBias: true }));
      model.add(tf.layers.dense({ units: availableBalloons.length, activation: "softmax" }));

      // Compile the model
      model.compile({
        optimizer: "adam",
        loss: "categoricalCrossentropy",
        metrics: ["accuracy"],
      });

      // Train the model
      const labelsTensor = tf.oneHot(
        tf.tensor1d(
          sampleInput.map((ticket) => availableBalloons.findIndex((b) => b.likes === ticket.likes)),
          "int32"
        ),

        availableBalloons.length
      );
      model
        .fit(normalizedInputTensor, labelsTensor, { epochs: 100 })
        .then(() => {
          // Make allocation predictions
          const predictions = model.predict(normalizedInputTensor);
          const allocationPredictions = Array.from(predictions.argMax(1).dataSync());

          // Allocate tickets to balloons based on predictions
          allocationPredictions.forEach((allocationIndex, i) => {
            const ticket = sampleInput[i];
            const balloon = availableBalloons[allocationIndex];
            if (ticket.numberofpeople <= balloon.capacity) {
              allocationData.push({ ticketId: ticket.ticketId, allocation: balloon.name });
            } else {
              var remainingPeople = ticket.numberofpeople - balloon.capacity;
              allocationData.push({ ticketId: ticket.ticketId, allocation: balloon.name });
              // Allocate remaining people to the next balloon
              for (let j = allocationIndex + 1; j < availableBalloons.length; j++) {
                const nextBalloon = availableBalloons[j];
                if (remainingPeople <= nextBalloon.capacity) {
                  allocationData.push({ ticketId: ticket.ticketId, allocation: nextBalloon.name });
                  break;
                } else {
                  allocationData.push({ ticketId: ticket.ticketId, allocation: nextBalloon.name });
                  remainingPeople -= nextBalloon.capacity;
                }
              }
            }
          });
          console.log("Hello from allocationData", this.inputData);
          const testInput = this.inputData;
          const testInputTensor = tf.tensor2d(
            testInput.map((ticket) => [ticket.numberofpeople]),
            [testInput.length, 1]
          );
          const normalizedTestInputTensor = tf.div(testInputTensor, tf.scalar(10));
          const testPredictions = model.predict(normalizedTestInputTensor);
          const testAllocationPredictions = Array.from(testPredictions.argMax(1).dataSync());
          const testAllocationData = [];

          testAllocationPredictions.forEach((allocationIndex, i) => {
            const ticket = testInput[i];
            const balloon = availableBalloons[allocationIndex];

            if (ticket.numberofpeople <= balloon.capacity) {
              // Allocate the entire group to the balloon
              testAllocationData.push({
                ticketId: ticket.ticketId,
                allocation: balloon.name,
                numberofpeople: ticket.numberofpeople,
              });
            } else {
              // Allocate half of the group to the current balloon and the remaining half to the next balloon
              testAllocationData.push({
                ticketId: ticket.ticketId,
                allocation: balloon.name,
                numberofpeople: balloon.capacity,
              });

              let remainingPeople = ticket.numberofpeople - balloon.capacity;
              let nextBalloonIndex = allocationIndex + 1;

              while (remainingPeople > 0 && nextBalloonIndex < availableBalloons.length) {
                const nextBalloon = availableBalloons[nextBalloonIndex];

                if (remainingPeople <= nextBalloon.capacity) {
                  // Allocate the remaining people to the next balloon
                  testAllocationData.push({
                    ticketId: ticket.ticketid,
                    allocation: nextBalloon.name,
                    numberofpeople: remainingPeople,
                  });
                  break;
                } else {
                  // Allocate the full capacity of the next balloon
                  testAllocationData.push({
                    ticketId: ticket.ticketid,
                    allocation: nextBalloon.name,
                    numberofpeople: nextBalloon.capacity,
                  });
                  remainingPeople -= nextBalloon.capacity;
                  nextBalloonIndex++;
                }
              }
            }
          });

          console.log("Test Allocation data:", testAllocationData);
          this.allocationData = testAllocationData;
          console.log("Allocation data:", allocationData);
        })
        .catch((error) => {
          console.error("An error occurred during training:", error);
        });
    },
  },
};
</script>
