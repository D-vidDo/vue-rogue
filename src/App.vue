<template>
  <v-app>
    <v-main>
      <h1>Rogue Game</h1>
      <div>
        gold: {{ gold }} crit chance: {{ critchance }} hp bonus: {{ hpBonus }} overheal: {{overheal}}
        <table class="table table-striped table-bordered">
            <thead>
                <tr>
                    <th>Item</th>
                    <th>Cost</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in items" :key="item.name">
                    <!-- <td>{{item.value}}</td> -->
                    <td>{{item.name}}</td>
                    <td>{{item.cost}}</td>
                    <td><v-btn v-on:click="purchaseItem(item.cost, item.type, item.value), shopSnack()">Buy</v-btn></td>
                </tr>

            </tbody>
        </table>


<!--  -->



        <Snackbar />
        <v-snackbar v-model="snackbar" timeout="2500">
          {{ message }}
          <template v-slot:action="{ attrs }">
            <v-btn color="pink" text v-bind="attrs" @click="snackbar = false">
              Close
            </v-btn>
          </template>
        </v-snackbar>
      </div>
      <body>
        your hp: {{ playerHP }}
        <v-progress-linear color="green" height="20" v-bind:value="playerHP">
        </v-progress-linear>

        enemy hp: {{ enemyHP }}
        <v-progress-linear color="red" height="20" v-bind:value="enemyHP">
        </v-progress-linear>

        <v-btn v-on:click="attack()">Attack</v-btn>
        <v-btn v-on:click="heal()">Heal</v-btn>

        <v-sheet color="white" elevation="1">
          <v-alert type="success">{{ playerMessage }}</v-alert>
          <v-alert type="error">{{ enemyMessage }}</v-alert>
        </v-sheet>
      </body>
    </v-main>
  </v-app>
</template>

<script>
import EventBus, { ACTIONS } from "../EventBus/index";
import Snackbar from "./components/snackbar.vue";
export default {
  name: "App",

  data: () => {
    return {
      snackbar: "",
      timeout: 500,
      // base health
      enemyHP: 100,
      playerHP: 100,
      hpBonus: 0,
      overheal: false,
      // attack base damage stats
      playerDamageMin: 1,
      playerDamageMax: 10,
      critchance: 0,
      // base heal stats
      playerHealMin: 5,
      playerHealMax: 20,
      // enemy base damage stats
      enemyDamageMin: 0,
      enemyDamageMax: 0,
      // gold
      gold: 0,
      // messages
      playerMessage: "",
      enemyMessage: "",
      message: "",
      // scaling stats
      hpScaling: 0,
      goldScaling: 0,
      // shop
      items: [
        // health
      { value: 10, name: 'HP +10', cost: 50, type: "hp" },
      { value: 30, name: 'HP +30', cost: 75, type: "hp" },
      { value: 50, name: 'HP +50', cost: 120, type: "hp" },
      //crit
      { value: 0.1, name: 'Crit +10%', cost: 150, type: "crit" },
      { value: 0.25, name: 'Crit +25%', cost: 300, type: "crit" },
      // overheal
      {value: "overheal", name: 'Overheal', cost: 500, type: "item"},
    ],
    };
  },

  methods: {
    attack() {
      let dmg = this.calculateDamage(
        this.playerDamageMin,
        this.playerDamageMax
      );
      if (Math.random() >= 1 - this.critchance) {
        dmg *= 2;
        this.enemyHP -= dmg;
        this.playerMessage = "⚔️⚡You CRITICALLY STRIKE for " + dmg + "!!⚡⚔️";
      } else {
        this.enemyHP -= dmg;
        this.playerMessage = "⚔️You attack for " + dmg + "⚔️";
      }
      if (this.checkWin()) {
        return;
      }
      this.enemyAttack();
    },
    shopSnack() {
      return (this.snackbar = true);
    },
    enemyAttack() {
      let dmg = this.calculateDamage(this.enemyDamageMin, this.enemyDamageMax);
      this.enemyMessage = "⚔️The enemy attacks for " + dmg + "⚔️";
      this.playerHP -= dmg;
      this.checkWin();
      return;
    },
    calculateDamage(min, max) {
      return Math.max(Math.floor(Math.random() * max) + 1, min);
    },
    heal() {
      let heal = this.calculateDamage(this.playerHealMin, this.playerHealMax);

      if (!this.overheal) {
        if (this.playerHP + heal > (this.playerHP + this.hpBonus)) {
          this.playerHP = 100 + this.hpBonus;
        }
      } else {
        this.playerHP += heal;
      }

      this.playerMessage = "🩹You heal for " + heal + "🩹";
      if (this.checkWin()) {
        return;
      }
      this.enemyAttack();
    },
    checkWin() {
      if (this.enemyHP <= 0) {
        confirm("You slaughtered the enemy... A stronger one approaches");
        this.goldScaling += 25;
        this.hpScaling += 25;
        this.gold += 25 + this.goldScaling;
        this.nextEnemy();
        return true;
      } else if (this.playerHP <= 0) {
        confirm("You lose!");
      }
    },
    nextEnemy() {
      this.playerHP = 100 + this.hpBonus;
      this.enemyHP = 100 + this.hpScaling;
    },
    purchaseItem(itemCost, itemStat, itemVal) {
      if (itemCost > this.gold) {
        this.message = "Not Enough Gold!";
      } else {
        if (itemStat == "hp") {
          this.hpBonus += itemVal;
          this.playerHP = 100 + this.hpBonus;
          this.gold -= itemCost;
          this.message = "Item Bought!";
        } else if (itemStat == "crit") {
          this.critchance += itemVal;
          this.gold -= itemCost;
          this.message = "Item Bought!";
        } else if (itemStat == "item") {
          if (itemVal=="overheal"){
            this.overheal = true;
          }
          this.gold -= itemCost;
          this.message = "Item Bought!";
        }
      }
    },
  },
  components: {
    //Snackbar,
    Snackbar,
  },
  mounted() {
    EventBus.$on(ACTIONS.SNACKBAR, (message) => {
      this.snackbarMessage = message;
      this.snackbar = true;
    });
  },
};
</script>

<style scoped>
body {
  text-align: center;
}
table,
th,
td {
  border: 1px solid black;
  text-align: center;
}
v-row {
  text-align: center;
}
</style>
