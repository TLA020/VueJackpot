<template>
  <v-card shaped class="darken-1 pa-4">
    <v-card-title> Jackpot</v-card-title>
    <div>
      <div id="wheel">
        <JackpotWheel
          ref="wheel"
          :pool="pool"
          :offset-degrees="270"
          font-color="white"
          :idle-speed="1"
          :chart-width="60"
          :time-left="timeLeft"
          @stopped="onStopped"
          :winner-text="winnerText"
        ></JackpotWheel>
      </div>

      <div v-if="!userBets">
        Place bets
      </div>

      <v-expansion-panels popout>
        <v-expansion-panel
          v-for="(userBet, i) in game.userBets"
          :key="i"
          hide-actions
        >
          <v-expansion-panel-header>
            <v-row align="center" class="spacer" no-gutters>
              <v-col cols="2">
                <v-avatar size="36px">
                  <img alt="Avatar" :src="getAvatar(userBet.player.avatar)" />
                </v-avatar>
              </v-col>

              <v-col cols="6">
                <strong v-html="userBet.player.email"></strong>
              </v-col>

              <v-col class="text-no-wrap" cols="4">
                <v-chip color="light" class="ml-0 mr-2 black--text" label>
                  <strong class="mr-1">
                    €{{
                      userBet.bets
                        .map(o => o.amount)
                        .reduce((a, c) => {
                          return a + c;
                        })
                        .toFixed(2)
                    }},-
                  </strong>
                </v-chip>
              </v-col>
            </v-row>
          </v-expansion-panel-header>

          <v-expansion-panel-content>
            <strong>Bets placed</strong>
            <v-divider></v-divider>
            <v-card-text>
              <ul v-for="bet in userBet.bets" :key="bet.Created">
                <li>
                  <span>€{{ bet.amount }},-</span>
                </li>
              </ul>
            </v-card-text>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
    </div>
  </v-card>
</template>

<script>
import JackpotWheel from "@/components/JackpotWheel";
export default {
  name: "Game",
  components: { JackpotWheel },

  data: () => ({
    winnerText: ""
  }),

  mounted() {
    this.$socketEvent("winner-picked", res => {
      this.$refs.wheel.spin(res.percentage);
      this.winnerText = res.player.email + " wins!";
    });
  },

  methods: {
    getAvatar(x) {
      if (process.env.VUE_APP_DEVELOPMENT === "1" && !x.includes("bear")) {
        return "http://localhost:5001/" + x;
      }
      return x;
    },

    onStopped() {
      setTimeout(() => {
        this.$refs.wheel.reset();
      }, 10000);
    },

    onSliceClicked(bet) {
      console.log(bet);
    }
  },

  computed: {
    pool() {
      if (!this.game.userBets) return [];
      return this.game.userBets.map(ub => ({
        name: ub.player.username || ub.player.email,
        amount: ub.bets
          .map(o => o.amount)
          .reduce((a, c) => {
            return a + c;
          }),
        avatar: this.getAvatar(ub.player.avatar)
      }));
    },

    game() {
      return this.$store.state.$game.game;
    },

    userBets() {
      return this.game.userBets;
    },

    timeLeft() {
      return this.$store.state.$game.timeLeft;
    },

    winner() {
      return this.$store.state.$game.winner;
    },

    gameState() {
      const game_states = ["idle", "inProgress", "ended", "winnerPicked"];
      let currentState = this.$store.state.$game.game.state || 0;
      return game_states[currentState];
    }
  }
};
</script>

<style scoped>
#wheel {
  text-align: center;
  width: 300px;
  height: 300px;
}
</style>
