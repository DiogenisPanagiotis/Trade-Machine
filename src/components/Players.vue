<template>
  <div class="container mobileHide">
    {{ validateTrade() }}
    <div class="row">
      <div class="col-xs-2 col-xs-offset-2">
        <button class="btn btn-md valid" @click="reSelectTeams()">
          <span class="glyphicon glyphicon-arrow-left" area-hidden="true"></span>
        </button>
      </div>
      <div class="col-xs-4 text-center">
        <button v-if="valid" class="btn btn-md valid" @click="completeTrade()">
          <span class="glyphicon glyphicon-transfer" area-hidden="true"></span>
        </button>
        <button v-if="!valid" class="btn btn-md notValid">
          <span class="glyphicon glyphicon-ban-circle" area-hidden="true"></span>
        </button>
      </div>
      <div class="col-xs-2 text-right">
        <button @click="renderFormSpree()" class="btn btn-md feedback">
          <span class="glyphicon glyphicon-send" area-hidden="true"></span>
        </button>
      </div>
    </div>
    <br/>
    <!-- Team & Net -->
    <div class="row">
      <div class="col-xs-4 col-xs-offset-2">
        <ul class="list-group">
          <li class="list-group-item list-group-team">
            <img class="logos" :src="team1Logo" height="30" width="30" /> </span class="team-in-trade">{{ team1 }}</span>
            <span v-if="team1Data.net >= 0" class="team-badge badge pos">+{{ dollarify(team1Data.net) }}</span>
            <span v-if="team1Data.net < 0" class="team-badge badge neg">-{{ dollarify(team1Data.net) }}</span>
          </li>
        </ul>
      </div>
      <div class="col-xs-4">
        <ul class="list-group">
          <li class="list-group-item list-group-team">
            <img class="logos" :src="team2Logo" height="30" width="30" /> <span class="team-in-trade">{{ team2 }}</span>
            <span v-if="team2Data.net >= 0" class="team-badge badge pos">+{{ dollarify(team2Data.net) }}</span>
            <span v-if="team2Data.net < 0" class="team-badge badge neg">-{{ dollarify(team2Data.net) }}</span>
          </li>
        </ul>
      </div>
    </div>
    <!-- Traded players -->
    <div class="row">
      <div class="col-xs-4 col-xs-offset-2">
        <ul class="list-group">
          <li v-for="(player, index) in team2Data.players" class="list-group-item list-group-trading" @click="undoTradePlayer(player, index)">
            {{ player.name }} - {{ player.position }}
            <span class="badge">{{ dollarify(player.salary) }}</span>
          </li>
        </ul>
      </div>
      <div class="col-xs-4">
        <ul class="list-group">
          <li v-for="(player, index) in team1Data.players" class="list-group-item list-group-trading" @click="undoTradePlayer(player, index)">
            {{ player.name }} - {{ player.position }}
            <span class="badge">{{ dollarify(player.salary) }}</span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Origin players -->
    <div class="row">
      <div class="col-xs-4 col-xs-offset-2">
        <ul class="list-group">
          <li v-for="(player, index) in team1Players" class="list-group-item" @click='tradePlayer(player)'>
            {{ player.name }} - {{ player.position }}
            <span class="badge">{{ dollarify(player.salary) }}</span>
          </li>
        </ul>
      </div>
      <div class="col-xs-4">
        <ul class="list-group">
          <li v-for="(player, index) in team2Players" class="list-group-item" @click='tradePlayer(player)'>
            {{ player.name }} - {{ player.position }}
            <span class="badge">{{ dollarify(player.salary) }}</span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <button @click="resetTrade()"type="button" class="close" data-dismiss="modal" aria-label="Close"><span class="closeModal" aria-hidden="true">&times;</span></button>
            <span class="glyphicon glyphicon-transfer success" aria-hidden="true"></span>
          </div>
          <div class="modal-body">

            <div class="row first-row">
              <div class="col-xs-6">
                <img :src="team1Logo" height="120px" width="120px" class="modal-img"/>
                <!-- <ul class="list-group">
                  <li class="list-group-item list-group-team modal-team">
                    {{ team1 }}
                    <span v-if="received.team1Salary >= 0" class="team-badge badge modal-badge pos">+{{ dollarify(received.team1Salary) }}</span>
                    <span v-if="received.team1Salary < 0" class="team-badge badge modal-badge neg">-{{ dollarify(received.team1Salary) }}</span>
                  </li>
                </ul> -->
              </div>
              <div class="col-xs-6">
                <img :src="team2Logo" height="120px" width="120px" class="modal-img"/>
                <!-- <ul class="list-group">
                  <li class="list-group-item list-group-team modal-team-right">
                    {{ team2 }}
                    <span v-if="received.team2Salary >= 0" class="team-badge badge modal-badge pos">+{{ dollarify(received.team2Salary) }}</span>
                    <span v-if="received.team2Salary < 0" class="team-badge badge modal-badge neg">-{{ dollarify(received.team2Salary) }}</span>
                  </li>
                </ul> -->
              </div>
            </div>

            <div class="row">
              <div class="col-xs-6">
                <ul class="list-group">
                  <li v-for="(player, index) in received.team1" class="list-group-item list-group-trading modal-list-items">
                    {{ player.name }} - {{ player.position }}
                    <span class="badge">{{ dollarify(player.salary) }}</span>
                  </li>
                </ul>
              </div>
              <div class="col-xs-6">
                <ul class="list-group">
                  <li v-for="(player, index) in received.team2" class="list-group-item list-group-trading modal-list-items-right">
                    {{ player.name }} - {{ player.position }}
                    <span class="badge">{{ dollarify(player.salary) }}</span>
                  </li>
                </ul>
              </div>
            </div>

          </div>
        </div>
      </div>
    </div>
    <!-- <Feedback></Feedback> -->

  </div>
</template>

<script>
import Feedback from './Feedback.vue';
import players from '../data/players.js';
import utils from '../utils.js';
export default {
  name: 'players',
  props: ['reSelectTeams', 'team1', 'team2', 'team1Logo', 'team2Logo'],
  data(){
    return {
      players1: players,
      players2: players,
      team1Data: {
        players: [],
        net: 0,
        trading: 0
      },
      team2Data: {
        players: [],
        net: 0,
        trading: 0
      },
      received: {
        'team1': [],
        'team1Salary': 0,
        'team2': [],
        'team2Salary': 0
      },
      completed: false,
      valid: null,
      disabled: [],
      active: false,
      spliced: null
    }
  },
  computed: {
    team1Players: utils.team1Players,
    team2Players: utils.team2Players
  },
  methods: {
    dollarify: utils.dollarify,
    tradePlayer: utils.tradePlayer,
    undoTradePlayer: utils.undoTradePlayer,
    validateTrade: utils.validateTrade,
    completeTrade: utils.completeTrade,
    resetTrade: utils.resetTrade,
    renderFormSpree: utils.renderFormSpree
  }
}
</script>

<style scoped>

.list-group-item {
  background-color: black;
}

.list-group-item:hover {
  background-color: black;
  /*color: #fff;*/
}
.team-badge {
  margin-top: 10px;
}
.team-badge-2 {
  float: left;
}
.badge {
  background-color: inherit;
}

.list-group-team {
  font-size: 16px;
  border: #fff solid 1px;
}
.list-group-team:hover {
  color: #fff;
  text-decoration: none;
  cursor: default;
}

.list-group-trading {
  border-color: #33B17D;
}
.list-group-trading:hover {
  /*background-color: #2f1115;*/
  color: #EE3017;
  text-decoration: line-through;
}

.pos {
  color: #33B17D;
}

.neg {
  color: #EE3017;
}

.btn-md {
  background-color: black;
  border: #33B17D 1px solid;
  vertical-align: middle;
  width: 100px;
}
.feedback {
  background-color: black;
  border: #4AA0DD 1px solid;
  vertical-align: middle;
  width: 100px;
}

.btn-md:hover {
  background-color: #33B17D;
}
.feedback:hover {
  background-color: #4AA0DD;
}

.btn-md:hover .glyphicon-transfer {
  color: black;
}
.btn-md:hover .glyphicon-arrow-left {
  color: black;
}
.btn-md:hover .glyphicon-send {
  color: black;
}

.glyphicon-transfer {
  background-color: transparent;
  color: #33B17D;
  margin-top: 3px;
}
.glyphicon-arrow-left {
  background-color: transparent;
  color: #33B17D;
  margin-top: 3px;
}

.glyphicon-send {
  background-color: transparent;
  color: #4AA0DD;
  margin-top: 3px;
}

.notValid {
  background-color: black;
  border: #EE3017 1px solid;
  vertical-align: middle;
  width: 100px;
  cursor: not-allowed;
}

.notValid:hover .glyphicon-ban-circle {
  color: black;
}

.notValid:hover {
  background-color: #EE3017;
}

.glyphicon-ban-circle {
  background-color: transparent;
  color: #EE3017;
  margin-top: 3px;
}
.glyphicon-ban-circle:hover {
  cursor: not-allowed;
}
.modal-header {
  /*margin-top: 150px;*/
  padding-top: 20px;
  background-color: black;
  border-bottom: 5px solid #33B17D;
  vertical-align: middle;
}
.modal-body {
  padding: 5px 5px 0px 5px;
  /*border: 1px solid #33B17D;*/
  background-color: black;
  margin-bottom: 0px;
}
.close {
  color: #fff;
  opacity: 1;
  text-shadow: none;
  /*transition: 0.5s ease-out;*/
}
.close:hover {
  color: #33B17D;
  text-shadow: none;
  /*transform: rotate(360deg);*/
}

.modal-dialog {
  padding-top: 12.5%;
}

.col-xs-6 {
  padding: none;
}

.modal-team {
  font-size: 15px;
  border: none;
  padding-right: 0px;
  padding-bottom: 0px;
}
.modal-team:hover {
  text-decoration: none;
  cursor: default;
  color: #fff;
}

.modal-team-right {
  font-size: 15px;
  border: none;
  padding-left: 0px;
  background-color: #33B17D;
}
.modal-team-right:hover {
  text-decoration: none;
  cursor: default;
  color: #fff;
}

.modal-list-items {
  border: none;
  padding-right: 0px;
  margin-left: 7px;
}
.modal-list-items:hover {
  cursor: default;
  color: #fff;
  text-decoration: none;
}
.modal-list-items-right {
  border: none;
  padding-left: 0px;
  margin-left: 7px;
}
.modal-list-items-right:hover {
  text-decoration: none;
  cursor: default;
  color: #fff;
}

.modal-badge {
  margin-top: 3px;
}

.modal-content {
border: 5px solid #33B17D;
overflow: hidden;
-webkit-border-radius: 5px !important;
-moz-border-radius: 5px !important;
border-radius: 5px !important;
}

.header-success {
  font-size: 15px;
  color: #33B17D;
}
.success {
  font-size: 20px;
  display: inline;
  color: #33B17D;
  margin-left: 10px;
  margin-top: 15px;
}

@media only screen
and (min-device-width : 320px)
and (max-device-width : 480px){  .mobileHide { display: none;}}

.logos {
  margin-bottom: 3px;
}
.btn:focus {
  outline: none;
}
.modal-img {
  margin-top: 20px;
  margin-bottom: 20px;
  margin-left: 80px;
}
hr {
  margin-bottom: 0px;
  padding-bottom: 0px;
}
.first-row {
  margin-bottom: 0px;
  padding-bottom: 0px;
}
</style>
