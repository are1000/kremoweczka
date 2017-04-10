<template>
  <div class="wrapper">
    <div class="header">PopeClicker</div>
    <div class="state">{{ round(save.creampies) }} <span class="icon-creampie"></span><br/>
      <span v-if="save.kps > 0 && !save.virus">{{ round(save.kps*save.slave_multiplier*save.global_multiplier) }} <span class="icon-creampie"></span>/s</span>
      <span v-if="save.virus" class="virustip">wirus watykańczyk</span>
      <h2 v-if="save.virus">{{save.virusleft}} / {{save.virushp}}</h2>
    </div>
    <img src="/static/creampie.png" v-if="!save.virus" class="creampie" v-on:click="creampie"/>
    <img src="/static/jp2.png" v-if="save.virus" class="yellowface" v-on:click="fightpope"/>
    <div class="ulwrapper" v-bind:class="{ hidden: (save.max_creampies < 50) }">
      <h2>niewolnicy / fabryki</h2>
      <ul class="slaves">
        <li v-for="(slave, key) in slaves" v-bind:class="{ hidden: (save.max_creampies < slave.cost/2) }">
          <span class="row bold">
            <span class="seg">{{ slave.name }}</span>
          </span>
          <span class="row smaller">
            <span class="seg">{{ slave.kps }} <span class="icon-creampie"></span>/s</span>
            <span class="seg">{{ round(slave.cost) }} <span class="icon-creampie"></span></span>
            <span class="seg count" v-if="save.slaves.hasOwnProperty(key)">{{ save.slaves[key] }} ( {{ round(save.slaves[key]*slave.kps) }} <span class="icon-creampie"></span>/s )</span>
          </span>
          <span class="row smaller mtop">
            <span class="seg">{{ slave.des }}</span>
          </span>
          <span class="button buy" v-bind:class="{ active: (save.creampies >= slave.cost) }" v-on:click="buyslave(key)">KUP</span>
        </li>
      </ul>
    </div>
    <div class="ulwrapper hidden">
      <h2>apgrejdy</h2>
      <ul class="upgrades">
        <li v-for="(upgrade, key) in upgrades" v-bind:class="{ hidden: (save.max_creampies < upgrade.cost/2) }">
          <span class="row">
            <span class="seg">{{ upgrade.name }} <span v-if="save.upgrades.hasOwnProperty(key)">{{ roman(save.upgrades[key]+1) }}</span></span>
            <span class="seg">{{ round(upgrade.cost) }} <span class="icon-creampie"></span></span>
          </span>
          <span class="row">
            <span class="seg">{{ upgrade.des }}</span>
          </span>
          <span class="row">
            <span class="seg">{{ upgrade.udes }}</span>
          </span>
          <span class="button buy" v-bind:class="{ active: (save.creampies >= upgrade.cost) }" v-on:click="buyupgrade(key)">KUP</span>
        </li>
      </ul>
    </div>
    <div class="ulwrapper" v-bind:class="{ hidden: (save.a_count == 0) }">
      <h2>acziwmenty</h2>
      <ul class="achievements"></ul>
    </div>
    <div class="footer">protip: {{ protip }}<br />
    designed by michau z wykop.pl, lat 8<br />
    v{{ version }}<br />
    <a href="#" v-on:click="hardreset" v-if="!save.virus">hard reset</a></div>
  </div>
</template>

<script>
  import $ from 'jquery';
  const version = "0.0.3";

  const slaves = {
    'pszczoly': {
      'name': 'pszczoły',
      'kps': 0.1,
      'base_cost': 10,
      'cost_multiplier': 1.05,
      'des': 'jedzą gówno'
    },
    'kwejkowicz': {
      'name': 'kwejkowicz',
      'kps': 1,
      'base_cost': 100,
      'cost_multiplier': 1.25,
      'des': 'ostatnie ogniwo łańcucha pokarmowego gówna'
    },
    'wykopek': {
      'name': 'wykopek',
      'kps': 3,
      'base_cost': 500,
      'cost_multiplier': 1.25,
      'des': 'dej plusika x-DDDD'
    },
    'karaczaners': {
      'name': 'karaczaners',
      'kps': 5,
      'base_cost': 1000,
      'cost_multiplier': 1.25,
      'des': 'elita internetu'
    },
    'rolewski': {
      'name': 'rolewski',
      'kps': 10,
      'base_cost': 5000,
      'cost_multiplier': 1.25,
      'des': '♿'
    },
    'zalgo': {
      'name': 'zalgo',
      'kps': 25,
      'base_cost': 10000,
      'cost_multiplier': 1.25,
      'des': 'jestem studentem prawa'
    },
    'wadowice': {
      'name': 'wadowice',
      'kps': 100,
      'base_cost': 50000,
      'cost_multiplier': 1.5,
      'des': 'to w tym mieście'
    },
    'portal': {
      'name': 'portal do podziemnego świata kremówek',
      'kps': 150,
      'base_cost': 100000,
      'cost_multiplier': 1.5,
      'des': 'o kurwełe'
    },
    'kremowka': {
      'name': 'kremówka śmierci',
      'kps': 300,
      'base_cost': 1000000,
      'cost_multiplier': 1.5,
      'des': 'zabija nienarodzone płody'
    },
    'gimba': {
      'name': 'gimba 2000',
      'kps': 1000,
      'base_cost': 100000000,
      'cost_multiplier': 1.5,
      'des': 'seba'
    }
  };

  const upgrades = {};

  const protipy = [
    '/nk/ protipy',
    'early access to dobry sposób na oszukanie frajerów',
    'do gejowskiego baru wchodzą pedofil, złodziej i ksiądz - a to dopiero pierwszy klient',
    'dumny ty z siebie jesteś?'
  ];

  let cp = protipy[0];

  const achievements = {};
  const iap = {};
  const dlc = {};

  const default_save = {'version': version, 'virus': false, 'virusleft': 0, 'virushp': 0, 'viruschance': 0.01, 'creampies': 0, 'all_creampies': 0, 'max_creampies': 0, 'a_count': 0, 'salt': 0, 'all_salt': 0, 'max_salt': 0, 'kps': 0, 'global_multiplier': 1, 'slave_multiplier': 1, 'click_multiplier': 1, 'slaves': {}, 'upgrades': {}, 'achievements': {}, 'iap': {}, 'dlc': {}};
  let save = default_save;

  /* Load save */
  try {
    save = JSON.parse(localStorage.getItem('save'));
  } catch (e) {
    save = default_save;
  }
  if (!save)
    save = default_save;

  recalculateCosts();

  if (version != save.version) {
    save.version = version;
    save.kps = 0;
    for (var k in save.slaves) {
      if (save.slaves.hasOwnProperty(k) && save.slaves[k] > 0 && slaves[k]) {
        save.kps += slaves[k].kps * save.slaves[k];
      }
    }
  }

  if (save.virus) viruson(true);

  function shitsAnimatedYo(shit, animation) {
    $(shit).removeClass(animation).addClass(animation).one('webkitAnimationEnd mozAnimationEnd MSAnimationEnd oanimationend animationend', function(){
      $(this).removeClass(animation);
    });
  };

  function ispow(num) {
    var str = num.toString();
    if (str[0] == "1") {
      for (var i = 1; i < str.length; i++) {
        if (str[i] != "0") return false;
      }
      return true;
    } else {
      return false;
    }
  }

  function creampies(number) {
    save.all_creampies += number;
    save.creampies += number;
    if (save.creampies > save.max_creampies) save.max_creampies = save.creampies;
    //if (save.max_creampies >= 100 && ispow(save.max_creampies)) viruson(true);
    shitsAnimatedYo('.creampie', 'pulse');
  }

  function recalculateCosts() {
    for (var k in slaves) {
      if (slaves.hasOwnProperty(k)) {
        slaves[k].cost = slaves[k].base_cost;
      }
    }
    for (var k in save.slaves) {
      if (save.slaves.hasOwnProperty(k) && save.slaves[k] > 0 && slaves[k]) {
        slaves[k].cost = slaves[k].base_cost * Math.pow(slaves[k].cost_multiplier, save.slaves[k]);
      }
    }
  }

  function protip() {
    cp = protipy[Math.floor(Math.random() * protipy.length)];
  }

  function viruson(yes) {
    save.virus = yes;
    if (yes) {
      save.virushp = 25;
      if (50*save.kps > save.creampies) save.virushp = 100;
      save.virusleft = save.virushp;
      $("body").addClass("virus");
    } else {
      $("body").removeClass("virus");
    }
  }

  setInterval(function () {
    if (save.kps*save.slave_multiplier*save.global_multiplier > 0 && !save.virus) {
      creampies(save.kps*save.slave_multiplier*save.global_multiplier);
    }
    if (save.virus) {
      if (save.creampies - save.kps*save.slave_multiplier*save.global_multiplier > 0) {
        save.creampies -= save.kps*save.slave_multiplier*save.global_multiplier;
      } else {
        save.creampies = 0;
      }

    }

    if (!save.virus && Math.random() < save.viruschance) viruson(true);
    localStorage.setItem("save", JSON.stringify(save));
  }, 1000);

  setInterval(protip, 30000);
  protip();

  export default {
    el: "#app",
    data() {
      return {
        "save": save,
        "slaves": slaves,
        "upgrades": upgrades,
        "protip": cp,
        "version": version
      }
    },
    methods: {
      creampie: function () { creampies(save.click_multiplier*save.global_multiplier); },
      fightpope: function () { if (save.virusleft) {
        save.virusleft -= save.click_multiplier*save.global_multiplier;
        if (save.virusleft <= 0) viruson(false);
      } else {
        viruson(false);
      }},
      buyslave: function (what) { if (save.creampies > slaves[what].cost) { save.creampies -= slaves[what].cost; if (save.slaves[what]) { save.slaves[what]++; } else { save.slaves[what] = 1; }  save.kps += slaves[what].kps; } recalculateCosts(); },
      buyupgrade: function (what) {},
      round: function (num) { return Math.round(num*10)/10; },
      roman: function (num) { return num; },
      hardreset: function() {
        save = default_save;
        localStorage.setItem("save", JSON.stringify(save));
        window.location.reload();
      },
      reset: function () {

      }
    }
  }
</script>

<style>
  @font-face {
    font-family: 'Bebas Neue';
    font-style: normal;
    font-weight: normal;
    src: local('Bebas Neue'), url('/static/BebasNeue.woff') format('woff');
  }

  body {
    -webkit-user-select: none;
    -moz-user-select: -moz-none;
    -ms-user-select: none;
    user-select: none;
    background: #242431;
    color: white;
    font-family: sans-serif;
  }

  .wrapper {
    max-width: 640px;
    margin: 0 auto;
  }

  .wrapper > * {
    margin: 0 auto;
    text-align: center;
  }

  .header {
    font-family: "Bebas Neue", "Impact", sans-serif;
    font-size: 4em;
  }

  .creampie, .yellowface {
    width: 200px;
    height: 200px;
    display: block;
    cursor: pointer;
    animation-duration: 0.9s;
    animation-fill-mode: both;
  }

  .yellowface {
    animation-duration: 0.25s;
    animation-name: fucc;
    animation-iteration-count: infinite;
  }

  @keyframes fucc {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(359deg);
    }
  }

  @keyframes virus {
    from {
      color: white;
      background: #ff4444;
    }
    to {
      color: red;
      background: #ffffff;
    }
  }


  @keyframes pulse {
    from {
      transform: scale3d(1, 1, 1);
    }

    50% {
      transform: scale3d(1.05, 1.05, 1.05);
    }

    to {
      transform: scale3d(1, 1, 1);
    }
  }

  .pulse {
    animation-name: pulse;
  }

  .slaves {
    display: block;
    list-style-type: none;
    padding: 0;
    margin: 0;
  }

  .slaves li {
    position: relative;
    background: rgba(0,0,0,0.25);
    padding: 5px;
    margin: 5px;
  }
  .row {
    display: block;
  }
  .seg {
    margin-right: 10px;
  }

  .buy.active {
    background: rgba(0,0,0,0.25);
    cursor: pointer;
    visibility: visible;
  }

  .buy {
    padding: 20px 15px;
    display:table-cell;
    vertical-align: middle;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    visibility: hidden;
  }

  .icon-creampie {
    display:inline-block;
    background: url('/static/creampie.png');
    background-size: contain;
    width: 0.9em;
    height: 0.9em;
  }

  .hidden {
    display: none;
  }

  .count {
    font-weight: bold;
  }

  .footer {
    margin-top: 20px;
    font-size: 0.75em;
  }
  .smaller {
    font-size: 0.75em;
  }
  .mtop {
    margin-top: 5px;
  }

  .virus {
    background: #ff4444;
    animation-duration: 0.25s;
    animation-name: virus;
    animation-iteration-count: infinite;
  }

  a, a:visited {
    color: #4444ff;
  }
</style>