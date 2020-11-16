<template>
  <v-app>
    <v-main>
      <v-container>
        <p>Most recent action:
          {{ this.voting[this.voting.length - 1] ? this.voting[this.voting.length - 1].verdict || 'None' : 'None' }}</p>
        <v-btn v-if="!isVoting" @click="startVoting()">Track votes</v-btn>
        <v-btn v-else @click="endVoting()">End voting</v-btn>
        <v-btn @click="score()">Score</v-btn>
        <v-row>
          <v-col v-for="(player, i) in players" :key="i" class="col-6">
            <v-menu :close-on-content-click="false" v-model="menus[i]" :disabled="isVoting">
              <template v-slot:activator="{on, attrs}">
                <v-card v-on="on" v-bind="attrs" :disabled="isDead(i)">
                  <v-card-text>
                    <div class="player" style="text-align: left;">
                      <img
                          :src="require('./assets/colors/' + colors[player.color].color + '.webp')"
                          :alt="colors[player.color].color + ' crewmate'" width="48px"
                          height="auto"
                          style="vertical-align: middle">
                      <span class="text-h4" style="margin-left: 20px;">{{ player.name }}
                      <span v-if="isVoting && !getKilledBy(player)" style="display:block;">
                        <v-btn-toggle v-model="votes[i]" multiple>
                          <v-btn v-for="pl in players" :key="pl.name" :disabled="!isVoteButtonEnabled(pl, i)">
                            <v-avatar tile size="16">
                              <img v-if="isVoteButtonEnabled(pl, i)"
                                   :src="require('./assets/colors/' + colors[pl.color].color + '.webp')"
                                   :alt="colors[pl.color].color">
                              <v-icon v-else>mdi-close</v-icon>
                            </v-avatar>
                          </v-btn>
                        </v-btn-toggle>
                      </span>
                      <span v-else-if="getKilledBy(player)" style="font-size: 16pt;">
                        <br><v-icon>mdi-close</v-icon>Killed by {{ getKilledBy(player).name }}
                      </span>
                      <span v-else-if="player.voted_out && !player.impostor" style="font-size: 16pt;">
                        <br><v-icon>mdi-close</v-icon>Voted out
                      </span>
                      <span v-else-if="player.voted_out && player.impostor" style="font-size: 16pt;">
                        <br><v-icon>mdi-close</v-icon>Voted out (Impostor)
                      </span>
                      <span v-else-if="player.impostor" style="color: red; font-size: 16pt;">
                        <br><v-icon color="red">mdi-close</v-icon>{{ player.kills.length }} kills
                      </span>
                      <span v-else style="color: green; font-size: 16pt;">
                        <br><v-icon color="green">mdi-check</v-icon>Alive
                      </span>
                    </span>
                    </div>
                  </v-card-text>
                </v-card>
              </template>
              <v-card>
                <v-card-title>Edit {{ player.name }}</v-card-title>
                <v-card-text>
                  <v-text-field v-model="players[i].name"></v-text-field>
                  <v-btn-toggle v-model="players[i].color" mandatory>
                    <v-btn v-for="object in colors" :key="object.color" :disabled="isColorUsed(object.color)">
                      <v-avatar tile size="38">
                        <img v-if="!isColorUsed(object.color)"
                             :src="require('./assets/colors/' + object.color + '.webp')" :alt="object.color"/>
                      </v-avatar>
                    </v-btn>
                  </v-btn-toggle>
                  <v-text-field v-model="players[i].notes" placeholder="Notes about this player"></v-text-field>
                  <v-checkbox v-model="players[i].impostor" :label="'Impostor'"></v-checkbox>
                  <div v-if="players[i].impostor">
                    <p>Kills</p>
                    <v-select v-model="players[i].kills" single-line multiple :items="generateKillsMenu(players[i])"/>
                  </div>
                </v-card-text>
              </v-card>
            </v-menu>
          </v-col>
          <v-col class="col-12">
            <v-row>
              <v-card v-if="isVoting" class="col-12">
                <v-card-text>
                  <v-col class="col-5">
                    <!-- Skip Vote -->
                    <div class="player" style="text-align: left;">
                      <v-icon x-large>mdi-skip-next</v-icon>
                      <span class="text-h4" style="margin-left: 20px;">Skip Vote</span>
                    </div>
                  </v-col>
                  <v-col class="col-6">
                    <v-btn-toggle v-model="votes['skip']" multiple>
                      <v-btn v-for="(pl) in players" :key="pl.name" :disabled="!isVoteButtonEnabled(pl, -1)">
                        <v-avatar tile size="16">
                          <img v-if="isVoteButtonEnabled(pl, -1)"
                               :src="require('./assets/colors/' + colors[pl.color].color + '.webp')"
                               :alt="colors[pl.color].color"/>
                          <v-icon v-else>mdi-close</v-icon>
                        </v-avatar>
                      </v-btn>
                    </v-btn-toggle>
                  </v-col>
                </v-card-text>
              </v-card>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      //List of players being tracked
      players: [
        {
          name: 'Player 1',
          impostor: undefined,
          kills: [],
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 2',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 3',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 4',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 5',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 6',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 7',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 8',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 9',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }, {
          name: 'Player 10',
          impostor: undefined,
          kills: 0,
          color: 0,
          notes: '',
          voted_out: false,
        }
      ],
      //Votes for the previous round
      voting: [],
      //Votes for the current round
      votes: {},
      //Variables related to voting state
      isVoting: false,
      votingRound: 0,
      //Variables for other UI components
      colors: [
        {color: 'red'},
        {color: 'blue'},
        {color: 'green'},
        {color: 'pink'},
        {color: 'orange'},
        {color: 'yellow'},
        {color: 'black'},
        {color: 'white'},
        {color: 'purple'},
        {color: 'brown'},
        {color: 'cyan'},
        {color: 'lime'},
      ],
      menus: [],
    }
  }, methods: {
    score() {
      //Crewmate
      const POINTS_PER_CREW_WIN = 4;
      const POINTS_PER_CREW_LOSS = -4;
      //Impostor
      const POINTS_PER_IMPOSTOR_WIN = 4;
      const POINTS_PER_KILL = 1;
      const POINTS_PER_IMPOSTOR_LOSE = -4;
      //Both
      const POINTS_PER_VOTE_OUT_CREW = -2;
      const POINTS_PER_VOTE_OUT_IMPOSTOR = 1;

      const impostorWin = this.isImpostorWin();

      for(let pl of this.players) {
        pl.score = 0;
        if(pl.impostor) {
          //Score kills
          pl.score += (pl.kills.length * POINTS_PER_KILL)
          //Score impostor wins
          if(impostorWin) {
            pl.score += POINTS_PER_IMPOSTOR_WIN;
          } else {
            pl.score += POINTS_PER_IMPOSTOR_LOSE;
          }
        } else {
          //Score crewmate wins
          if(impostorWin) {
            pl.score += POINTS_PER_CREW_LOSS;
          } else {
            pl.score += POINTS_PER_CREW_WIN;
          }
        }
        //Score votes
        for(let round of this.voting) {
          if(round.player_voted_out) {
            let player = this.players[round.player_voted_out];
            if(player.impostor) {
              pl.score += POINTS_PER_VOTE_OUT_IMPOSTOR;
            } else {
              pl.score += POINTS_PER_VOTE_OUT_CREW;
            }
          }
        }
        console.log(pl.name, "earned", pl.score, "points.");
      }
    },
    isImpostorWin() {
      let crew = 0;
      let impostors = 0;
      for(let pl of this.players) {
        if(pl.impostor && !pl.voted_out) {
          impostors ++;
        } else if(!pl.impostor && !pl.voted_out && !this.getKilledBy(pl)) {
          crew ++;
        }
      }
      if(impostors >= crew) return true;
      return false;
    },
    isVoteButtonEnabled(pl, i) {
      let vfw = this.votedForWho(pl);
      let cv = this.playerCanVote(pl);
      return vfw == i || cv;
    },
    votedForWho(pl) {
      let index = this.getIndex(pl);
      return this.hasPlayerVoted(index);
    },
    playerCanVote(pl) {
      return this.votedForWho(pl) == undefined;
    },
    hasPlayerVoted(i) {
      for (let v in this.votes) {
        if (this.votes[v].includes(i)) {
          return v;
        }
      }
    },
    startVoting() {
      this.isVoting = true;
      this.votingRound++;
      this.votes = {
        0: [],
        1: [],
        2: [],
        3: [],
        4: [],
        5: [],
        6: [],
        7: [],
        8: [],
        9: [],
      };
    },
    endVoting() {
      this.isVoting = false;
      //Check who was voted out
      let mostVotedVotes = 0;
      let mostVotedIndex = null;
      let tied = true;
      for (let vote in this.votes) {
        if (this.votes[vote].length > mostVotedVotes) {
          mostVotedVotes = this.votes[vote].length;
          mostVotedIndex = vote;
          tied = false;
        } else if (this.votes[vote].length == mostVotedVotes) {
          tied = true;
        }
      }
      let verdict = "";
      if (tied) {
        verdict = "No one was ejected. (Tie)";
        this.votes["player_voted_out"] = null;
      } else if (!this.players[mostVotedIndex]) {
        verdict = "No one was ejected. (Skipped)";
        this.votes["player_voted_out"] = null;
      } else {
        verdict = this.players[mostVotedIndex].name + " was ejected.";
        this.votes["player_voted_out"] = mostVotedIndex;
        this.players[mostVotedIndex].voted_out = true;
      }
      this.votes["verdict"] = verdict;
      //Store for later
      this.voting.push(this.votes);
      this.votes = {};
    },
    getIndex(player) {
      let i = 0;
      for (let p of this.players) {
        if (p.name === player.name) {
          return i;
        }
        i++;
      }
    },
    getKilledBy(player) {
      let i = this.getIndex(player);
      for (let p of this.players) {
        if (p.kills && p.kills.includes(i)) {
          return p;
        }
      }
    },
    generateKillsMenu(player) {
      let i = 0;
      let list = [];
      for (let p of this.players) {
        list.push({'text': p.name, 'value': i, 'disabled': !this.canKill(player, p)});
        i++;
      }
      return list;
    }, canKill(player1, player2) {
      if (player1.impostor && player2.impostor) return false;
      if (this.isDeadByPlayer(player2) && this.getKilledBy(player2) !== player1) return false;
      return player1.name !== player2.name;
    }, isDeadByPlayer(player) {
      let i = 0;
      for (let pl of this.players) {
        if (pl.name == player.name) {
          return this.isDead(i);
        }
        i++;
      }
    }, isDead(i) {
      for (let pl of this.players) {
        if (pl.kills && pl.kills.includes(i)) return true;
      }
      return false;
    }, isColorUsed(color) {
      for (let pl of this.players) {
        if (this.colors[pl.color].color == color) {
          return true;
        }
      }
      return false;
    }
  }, watch: {
    players: {
      handler: function (val) {
        let storage = window.localStorage;
        storage.setItem('among_us_tracker_data', JSON.stringify(val));
      },
      deep: true
    }
  }, mounted() {
    let storage = window.localStorage
    let val = storage.getItem('among_us_tracker_data');
    if (val) {
      this.players = JSON.parse(val);
    }
  }
}
</script>

<style scoped>
@import url(https://cdn.jsdelivr.net/npm/@mdi/font@4.x/css/materialdesignicons.min.css);

.grid-container {
  display: grid;
  grid-template-rows: repeat(5, 1fr);
  grid-template-columns: repeat(2, 1fr);
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
