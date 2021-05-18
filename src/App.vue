<template>
  <v-app>
    <v-main>
      <h1>Rogue Game</h1>
      <div>
        <table
          class="table table-striped table-bordered"
          style="margin-top: 10px; margin-left: 20px"
        >
          <thead>
            <tr>
              <th>Item</th>
              <th>Cost</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in items" :key="item.name">
              <td>{{ item.name }}</td>
              <td>{{ item.cost }}</td>
              <td>
                <v-btn
                  v-on:click="
                    purchaseItem(item.cost, item.type, item.value), shopAlert()
                  "
                  >Buy</v-btn
                >
              </td>
            </tr>
          </tbody>
        </table>
        <table style="margin-top: 10px; margin-left: 20px">
          <tr>
            <td>crit chance: {{ critchance * 100 }}%</td>
            <td>gold: {{ gold }}</td>
            <td>hp bonus:{{ hpBonus }}</td>
          </tr>
          <tr>
            <td>overheal: {{ overheal }}</td>
            <td>lifesteal: {{ lifesteal }}</td>
            <td>kills: {{ killCount }}</td>
          </tr>
          <tr>
            <td>hero min atk: {{ playerDamageMin }}</td>
            <td>hero max atk: {{ playerDamageMax }}</td>
          </tr>
          <tr>
            <td>enemy min atk: {{ enemyDamageMin }}</td>
            <td>enemy max atk: {{ enemyDamageMax }}</td>
          </tr>
        </table>

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
        <br />your hp: {{ playerHP }}
        <v-progress-linear color="green" height="20" v-bind:value="playerHP">
        </v-progress-linear>

        enemy hp: {{ enemyHP }}
        <v-progress-linear color="red" height="20" v-bind:value="enemyHP">
        </v-progress-linear>

        <v-sheet
          style="padding-top: 50px; margin-left: 600px; margin-right: 600px"
        >
          <v-alert id="v-alert1" STYLE="display:none" dense type="success">{{
            playerMessage
          }}</v-alert>
          <v-alert id="v-alert2" STYLE="display:none" dense type="error">{{
            enemyMessage
          }}</v-alert>
        </v-sheet>
      </body>
    </v-main>
    <v-bottom-navigation>
      <v-btn v-on:click="attack()">Attack</v-btn>
      <v-btn v-on:click="specialAttack()">Special Attack</v-btn>
      <v-btn v-on:click="heal()">Heal - {{ healcd }}</v-btn>
    </v-bottom-navigation>
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
      inBattle: false,
      killCount: 0,
      // base health
      enemyHP: 100,
      playerHP: 100,
      hpBonus: 0,
      healcd: 5,
      // attack base damage stats
      playerDamageMin: 5,
      playerDamageMax: 10,
      critchance: 0,
      // base heal stats
      playerHealMin: 5,
      playerHealMax: 20,
      // enemy base damage stats
      enemyDamageMin: 2,
      enemyDamageMax: 8,
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
        { value: 10, name: "HP +10", cost: 50, type: "hp" },
        { value: 30, name: "HP +30", cost: 125, type: "hp" },
        { value: 100, name: "HP +100", cost: 400, type: "hp" },
        { value: 500, name: "HP +500", cost: 1800, type: "hp" },
        { value: 1000, name: "HP +1000", cost: 3500, type: "hp" },
        // crit
        { value: 0.1, name: "Crit +10%", cost: 400, type: "crit" },
        { value: 0.25, name: "Crit +25%", cost: 900, type: "crit" },
        // overheal
        { value: "overheal", name: "Overheal", cost: 500, type: "item" },
        {
          value: "lifesteal",
          name: "Lifesteal (25% Damage)",
          cost: 500,
          type: "item",
        },
      ],
      // item buffs
      overheal: false,
      lifesteal: false,
    };
  },

  methods: {
    attack() {
      let dmg = this.calculateDamage(
        this.playerDamageMin,
        this.playerDamageMax
      );
      if (Math.random() >= 1 - this.critchance) {
        dmg *= 1.5;
        this.enemyHP -= dmg;
        this.playerMessage = "⚔️⚡You CRITICALLY STRIKE for " + dmg + "!!⚡⚔️";
        if (this.lifesteal) {
          this.playerHP += Math.round(dmg * 0.25);
          this.playerMessage +=
            "\n🩹You lifesteal for " + Math.round(dmg * 0.25) + "🩹";
        }
      } else {
        this.enemyHP -= dmg;
        this.playerMessage = "⚔️You attack for " + dmg + "⚔️";
        if (this.lifesteal) {
          this.playerHP += Math.round(dmg * 0.25);
          this.playerMessage +=
            "\n🩹You lifesteal for " + Math.round(dmg * 0.25) + "🩹";
        }
      }
      this.healcd--;
      if (this.checkWin()) {
        return;
      }
      this.enemyAttack();
    },
    specialAttack() {
      let dmg =
        this.calculateDamage(this.playerDamageMin, this.playerDamageMax) * 2;
      if (Math.random() >= 1 - this.critchance) {
        dmg *= 1.5;
        this.enemyHP -= dmg;
        this.playerMessage =
          "⚔️⚡YOUR SPECIAL ATTACK CRIT!!! FOR " + dmg + "!!⚡⚔️";
        if (this.lifesteal) {
          this.playerHP += Math.round(dmg * 0.25);
          this.playerMessage +=
            "\n🩹You lifesteal for " + Math.round(dmg * 0.25) + "🩹";
        }
      } else {
        this.enemyHP -= dmg;
        this.playerMessage = "⚔️Special Attack!!!! " + dmg + "⚔️";
        if (this.lifesteal) {
          this.playerHP += Math.round(dmg * 0.25);
          this.playerMessage +=
            "\n🩹You lifesteal for " + Math.round(dmg * 0.25) + "🩹";
        }
      }
      this.healcd--;
      if (this.checkWin()) {
        return;
      }
      this.enemyAttack();
    },
    shopAlert() {
      return (this.snackbar = true);
    },
    enemyAttack() {
      let dmg = this.calculateDamage(this.enemyDamageMin, this.enemyDamageMax);
      this.enemyMessage = "⚔️The enemy attacks for " + dmg + "⚔️";
      this.playerHP -= dmg;
      this.checkWin();
      this.inBattle = true;
      var x = document.getElementById("v-alert1");
      var y = document.getElementById("v-alert2");
      if (x.style.display === "none") {
        x.style.display = "block";
        y.style.display = "block";
      }
      return;
    },
    calculateDamage(min, max) {
      return Math.max(Math.floor(Math.random() * max) + 1, min);
    },
    heal() {
      let heal = this.calculateDamage(this.playerHealMin, this.playerHealMax);
      if (this.healcd <= 0) {
        if (!this.overheal) {
          if (this.playerHP + heal >= 100 + this.hpBonus) {
            this.playerHP = 100 + this.hpBonus;
            this.healcd = 5;
          } else {
            this.playerHP += heal;
            this.healcd = 5;
          }
        } else {
          this.playerHP += heal;
          this.healcd = 5;
        }
      } else {
        this.playerMessage =
          "Heal is on Cool Down! " + this.healcd + " turn(s) left!";
        return;
      }

      this.playerMessage = "🩹You heal for " + heal + "🩹";
      if (this.checkWin()) {
        return;
      }
      this.enemyAttack();
    },
    checkWin() {
      if (this.enemyHP <= 0) {
        confirm(
          "You slaughtered the enemy... A stronger one approaches\nAtk +5"
        );
        this.nextEnemy();
        return true;
      } else if (this.playerHP <= 0) {
        confirm("You lose!" + this.killCount);
        this.resetGame();
      }
    },
    resetGame() {
      // base health
      this.enemyHP = 100;
      this.playerHP = 100;
      this.hpBonus = 0;
      this.overheal = false;
      this.healcd = 5;
      // attack base damage stats
      this.playerDamageMin = 5;
      this.playerDamageMax = 10;
      this.critchance = 0;
      // base heal stats
      this.playerHealMin = 5;
      this.playerHealMax = 20;
      // enemy base damage stats
      this.enemyDamageMin = 2;
      this.enemyDamageMax = 8;
      // gold
      this.gold = 0;
      // scaling stats
      this.hpScaling = 0;
      this.goldScaling = 0;
      this.killCount = 0;
    },
    nextEnemy() {
      this.playerHP = 100 + this.hpBonus;
      this.hpScaling += 10;
      this.enemyHP = 100 + this.hpScaling;
      this.goldScaling += 25;
      this.enemyDamageMin += 2;
      this.enemyDamageMax += 3;
      this.playerDamageMin += 1;
      this.playerDamageMax += 4;
      this.playerHealMin += 5;
      this.playerHealMax += 10;
      this.healcd = 5;
      this.gold += 25 + this.goldScaling;
      this.inBattle = false;
      this.killCount++;
      this.message = "Shop is now open! Buy items before you enter combat!";
      this.shopAlert();
      var x = document.getElementById("v-alert1");
      var y = document.getElementById("v-alert2");
      if (x.style.display === "block") {
        x.style.display = "none";
        y.style.display = "none";
      }
    },
    purchaseItem(itemCost, itemStat, itemVal) {
      if (this.inBattle) {
        this.message = "Currently in battle! Can't buy items.";
      } else {
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
            if (itemVal == "overheal") {
              this.overheal = true;
            } else if (itemVal == "lifesteal") {
              this.lifesteal = true;
            }
            this.gold -= itemCost;
            this.message = "Item Bought!";
          }
        }
      }
    },
  },
  components: {
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
</style>
