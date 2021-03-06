<template>
  <div id="app">

    <b-navbar toggleable="md" type="light" variant="info" fixed class="fixed-top">

      <b-nav-toggle target="nav_collapse"></b-nav-toggle>

      <b-navbar-brand :to="{ name: 'Landing' }">
        <img src="./assets/logo.svg" class="logo"/>
      </b-navbar-brand>

      <b-collapse is-nav id="nav_collapse">

        <b-navbar-nav is-nav-bar>
          <b-nav-item :to="{ name: 'Landing' }" exact>Home</b-nav-item>
          <b-nav-item :to="{ name: 'Tutorial', params: {task: task} }">Tutorial</b-nav-item>
          <b-nav-item v-if="login.consent" :to="{ name: 'Play', params: {task: task} }">Play</b-nav-item>
          <b-nav-item v-else :to="{ name: 'Consent' }">Play</b-nav-item>
          <b-nav-item :to="{ name: 'Leaderboard', params: {task: task} }">Leaderboard</b-nav-item>

          <!--<b-nav-item to="/coins/ethereum" exact>Ethereum</b-nav-item>
          <b-nav-item to="/coins/bitcoin" exact>Bitcoin</b-nav-item>-->
          <b-nav-item-dropdown right v-if="login.loginType == 'github'">
            <!-- Using button-content slot -->
            <template slot="button-content" >
              <em>{{login.nickname}}</em>
            </template>
            <b-dropdown-item :to="{ name: 'Profile', params: {id: this.login.id}}">Profile</b-dropdown-item>
            <b-dropdown-item href="#" @click="logout()">Signout</b-dropdown-item>
          </b-nav-item-dropdown>
          <b-button v-else size="sm" class="my-2 my-sm-0"
          type="submit" @click="authenticate()">Login</b-button>

        </b-navbar-nav>

      </b-collapse>

      <!-- Right aligned nav items -->
      <b-navbar-nav is-nav-bar class="ml-auto" v-show="$route.path.indexOf('/play') == 0">

        <b-nav-form>
          <!--<b-form-input size="sm" class="mr-sm-2" type="text" placeholder="Search"/>-->
          <b-button variant="danger" size="sm" class="my-2 my-sm-0" v-on:click="doAction" :disabled="!status">
            <span v-if="status">
              {{status}}
            </span>
            <span v-else>
              loading
            </span>
            <!-- TODO: spinner here -->
          </b-button>


        </b-nav-form>

      </b-navbar-nav>

      <!-- Right aligned nav items -->
      <b-navbar-nav is-nav-bar class="ml-auto" v-show="$route.path.indexOf('/tutorial') == 0">

        <b-nav-form>
          <!--<b-form-input size="sm" class="mr-sm-2" type="text" placeholder="Search"/>-->
          <b-button size="sm" class="my-2 my-sm-0" v-on:click="doAction" :disabled="!status" v-if="tutorialShow()">
            <span v-if="status">
              {{status}}
            </span>
            <span v-else>
              loading
            </span>
            <!-- TODO: spinner here -->
          </b-button>


        </b-nav-form>

      </b-navbar-nav>

    </b-navbar>

    <!-- Set Nickname -->
    <b-modal id="nickname" title="Set your nickname" ref="nickname" @hidden="putNickname">
      <p class="my-4"> Choose your nickname to display on the leaderboard </p>
      <input class="form" placeholder="nickname" v-model="login.nickname"/>
    </b-modal>

    <!-- This is like a navbar. There are two ways to define this
    <router-link :to="{ name: 'Hello' }">Home</router-link>
    <router-link to="/about">About</router-link>-->
    <!-- These are dynamically created routes
    <router-link to="/coins/ethereum">Ethereum</router-link>
    <router-link to="/coins/bitcoin">Bitcoin</router-link>-->

    <!--This is the current route -->
    <div class="routerContent">

        <router-view ref="route" :login="login" :isAuthenticated="isAuthenticated"
        :all_tasks="all_tasks" :task="task"
        v-on:change_task="setTask" v-on:change_status="changeStatus"
        v-on:has_consented="setConsent"
        v-on:needs_authentication="authenticate"
        >
        </router-view>

    </div>

    <footer class="footer pt-5 pb-5 bg-info dark" v-show="!($route.path.indexOf('/play') == 0) && !($route.path.indexOf('/tutorial') == 0)">
      <b-container responsive>
        <b-row>
          <b-col>
            <b-link to="/about">About</b-link>
          </b-col>
        </b-row>
      </b-container>
    </footer>

  </div>
</template>

<script>
import Vue from 'vue';
import VueAxios from 'vue-axios';
import axios from 'axios';
import VueResize from 'vue-resize';
import chai from 'chai';
import store from 'store';
import auth from './lib/auth'; // leave authentication stuff outside of Vue ?
import config from './config';


/* eslint no-underscore-dangle: ["error", { "allow": ["_items", "_meta", "_links", "_id"] }] */

Vue.use(VueResize);

Vue.use(VueAxios, axios);

/*document.body.addEventListener('touchmove', (e) => {
  console.log('touchmove', e.target, window.location.href);

  if (window.location.href.indexOf('tutorial') >= 0) {
    if (e.target.id !== 'canvas-id' && e.target.className !== 'card-text') {
      console.log('preventing default', e);
      e.preventDefault();
    }
  }
});*/

export default {
  name: 'app',
  data() {
    return {
      status: 'Submit',
      mode: 'draw',
      task: 'cheng_et_al_001',
      all_tasks: [
        { name: 'Meningioma',
          task: 'cheng_et_al_001',
          level: 'easy',
          text: `
        Menigiomas are tumors that grow from the brains covering. Help us teach computers to spot these tumors so we can automate detection and predict outcomes.

        Menigiomas stand out from the surroundings and are a great place to start.

        The data is provided by the brain tumor dataset, by Jun Cheng, 2017. figshare.
        `,
          tutorial_title: 'Finding Meningiomas',
          desc: `Meningiomas are a type of brain tumor.
          They look bright on an MRI scan, and can be anywhere in the brain.
        `,
        },
        { name: 'Stroke',
          task: 'atlas_lesions',
          level: 'medium',
          text: `
        Strokes occur when the brain is damaged by lack of blood flow or by bleeding in the brain. Help us teach computers how to find damage caused by strokes.

        The areas most damaged by a stroke are easy to spot, but finding the full extent of the damage can be tricky.

        The data is provided by the ATLAS dataset, by Liew et al. 2017
        `,
          tutorial_title: 'Finding Stroke Lesions',
          desc: `Stroke lesion descripton.
        They look dark on an MRI scan, and can be anywhere in the brain.
      `,
        },
        { name: 'Dentate Gyrus',
          task: 'db_cor_context03',
          level: 'hard',
          text: `
        The dentate gyrus is part of the hippocampus that helps form new memories. Help us spot its complex, curving structure so we can learn more about how it works.

        The dentate gyrus is a band of darker tissue within the hippocampus that can be quite challenging to spot.
        `,
          tutorial_title: 'Finding the Dentate Gyrus',
          desc: `Dentate Gyrus descripton.

    `,
        },
        { name: 'Prostate Cancer',
          task: 'prostate001',
          level: 'hard',
          text: `
        This task currently has no training data.
        `,
          tutorial_title: 'Finding the ???',
          desc: `Enter descripton.

    `,
        },
      ],
      isAuthenticated: false,
      login: {
        username: null,
        nickname: null,
        avatar: 'images/Octocat1.jpg',
        etag: null,
        github_id: null,
        consent: false,
        total_score: 0,
        use_profile_pic: null,
        send_emails: null,
        do_dismiss: false,
        ave_score: 0,
        n_subs: 0,
        n_test: 0,
        n_try: 0,
        id: null,
        transfer_token: null,
        transfer_user_id: null,
        user_id: null,
        endpoint_id: null,
        token: null,
        loginType: 'anon',
      },
    };
  },
  computed: {

  },

  methods: {
    doAction() {
      if (this.status === 'Submit') {
        this.$refs.route.submitImg();
      } else {
        this.$refs.route.changeImg();
      }

      // console.log('new papersource', this.$refs.route.paperSrc)
    },

    putNickname() {
      console.log('nickname', this.login.nickname);
      const upass = `${this.login.id}:${this.login.token.token}`;
      axios({
        method: 'PATCH',
        url: `${config.player_url}${this.login.endpoint_id}`,
        crossDomain: true,
        processData: false,
        headers: {
          authorization: `Basic ${btoa(upass)}`, // config.edit_token,
          'content-type': 'application/json',
          'if-match': this.login.etag,
        },
        data: JSON.stringify({ nickname: this.login.nickname }),
      }).then((resp) => {
        console.log('response is', resp);
      }).catch((e) => {
        console.log('put error', e);
      });
    },

    tutorialShow() {
      if (this.$refs.route) {
        if (this.$refs.route.currentStep) {
          return this.$refs.route.currentStep('submit');
        }
      }
      return false;
    },

    changeStatus(status) {
      this.status = status;
    },

    authenticate() {
      const self = this;
      auth.login(self.login, (resp) => {
        console.log('authenticate resp', resp);
        self.getUserInfo();
      });
    },

    setConsent(inputs) {
      chai.assert.equal(inputs.user_id, this.login.id);
      this.login.consent = inputs.has_consented;
      store.set('has_consented', inputs.has_consented);
    },

    getUserInfo(Token, isAnon) {
      const token = auth.getToken().user_id || Token;

      chai.assert.isNotNull(token);
      // chai.assert.isDefined(token);
      console.log('is anonymous', isAnon, token);
      const self = this;

      // TODO: CHANGE THIS TO YOUR SERVER
      // In this example, we are getting user info from github
      // If this fails, then our token is bad; we are NOT authenticated and
      // should be logged out

      const url = `${config.player_url}?where=user_id%3D%3D%22${token}%22`;


      axios.get(url).then((resp) => {
        self.isAuthenticated = true;
        // TODO: do stuff here, like setting user info variables
        // console.log('the response from', url, "is", resp.data);
        self.setUserInfo(resp, isAnon);
      }).catch((e) => {
        console.log('error here', e);
        self.logout();
      });
    },

    setTask(task) {
      console.log('setting task', task);
      this.task = task;
    },

    setUserInfo(resp, isAnon) {
      let data = resp.data;
      if (data._items.length === 0) {
        this.isAuthenticated = false;
        const e = {};
        e.msg = 'ERROR, 0 users returned';
        throw e;
      }
      data = data._items[0];
      // console.log('setting user info', data)
      if (!isAnon) {
        this.isAuthenticated = true;
        this.login.loginType = 'github';
        this.login.token = auth.getToken();
        this.login.etag = data._etag;
        console.log('logged in with GitHub');
      }
      this.login.ave_score = data.ave_score;
      this.login.consent = data.has_consented;
      this.login.n_subs = data.n_subs;
      this.login.n_test = data.n_test;
      this.login.n_try = data.n_try;
      this.login.total_score = data.total_score;
      this.login.id = data.user_id;
      this.login.avatar = data.avatar;
      this.login.github_id = data.id;
      this.login.endpoint_id = data._id;
      if (!data.nickname && data._etag && this.login.loginType === 'github') {
        console.log('NEED USERNAME');
        this.$refs.nickname.show();
      } else {
        console.log('data', data);
        this.login.nickname = data.nickname;
      }

    },

    logout() {
      auth.logout();
      this.isAuthenticated = false;
      this.login.loginType = 'anon';
      this.anonymousLogin();
    },

    anonymousLogin() {
      console.log('logging out');
      axios.get(config.anonymous_url).then((resp) => {
        console.log('anonLogin', resp);
        this.login.token = { token: resp.data.token };
        this.login.transfer_token = resp.data.transfer_token;
        this.login.transfer_user_id = resp.data.user_id;
        this.getUserInfo(resp.data.user_id, true);
      }).catch((e) => {
        console.log(e);
      });
    },

  },

  created() {
    this.getUserInfo();
  },

  mounted() {


  },

  watch: {
    $route(to) {
      if (to.params.task) {
        this.task = to.params.task;
      }
    },
  },
};

</script>

<style lang="scss">

  @import "./custom-bootstrap.scss";
  @import "../node_modules/bootstrap/scss/bootstrap.scss";


  .missed {
    cursor: pointer;
    width: 45px;
    height: 45px;
    margin-right: 5px;
    background-color: black;
    border-color: $red;
    border-style: solid;
    border-radius: 10px;
  }

  .missed.view {
    background-color: $red;
  }

  .incorrect {
    cursor: pointer;
    width: 45px;
    height: 45px;
    margin-right: 5px;
    background-color: black;
    border-color: $light-blue;
    border-style: solid;
    border-radius: 10px;
  }

  .incorrect.view {
    background-color: $light-blue;
  }

  .correct {
    cursor: pointer;
    width: 45px;
    height: 45px;
    margin-right: 5px;
    background-color: black;
    border-color: $brand-warning;
    border-style: solid;
    border-radius: 10px;
  }

  .correct.view {
    background-color: $brand-warning;
  }
  </style>

<style>
  #app {
    /* font-family: 'Avenir', Helvetica, Arial, sans-serif; */
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    height: auto;
    overflow: auto;
    /*margin-top: 60px;*/
  }

  a .active {
    color: red;
  }

  .routerContent{
    margin-top: 56px;
    height: 100%;
    width: 100%;
  }

  .logo {
    height: 25px;
    width: 25px;
  }

  nav {
    z-index: 9;
  }

  .jumbotron {
    padding-top: 6rem;
    padding-bottom: 6rem;
    margin-bottom: 0;
    background-color: #fff;
  }

  .jumbotron p:last-child {
    margin-bottom: 0;
  }

  .jumbotron-heading {
    font-weight: 300;
  }

  .jumbotron .container {
    max-width: 40rem;
  }

  .jumbotron h1 {
    margin-bottom: 20px;
    padding-bottom: 9px;
    border-bottom: 1px solid #eee;
  }

  /*.footer{
    background-color: black;
    color:white;
  }

  .footer a {
    color: white;
  }*/

</style>
