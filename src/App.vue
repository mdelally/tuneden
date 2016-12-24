<template>
  <section class="hero is-fullheight is-bold is-dark">
      <div class="hero-body">
        <div class="container window" v-if="loaded">
          <h1 class="title is-1">{{ user }}'s tuneden</h1>
          
          <p class="control has-addons">
            <input type="text" class="input is-expanded"
                   v-model="user"
                   @keyup.enter.prevent="getUserData"
                   placeholder="Username to inspect..."
            >
            <button class="button is-success" @click.prevent="getUserData"><i class="fa fa-fw fa-search"></i></button>
          </p>
          
          <hr>
          
          <div v-if="nowPlaying">
            <p><h2 class="title is-2 loading">Now Playing</h2></p>

            <article class="media">
              <figure class="media-left">
                <p class="image is-128x128">
                  <img :src="nowPlaying.image[2]['#text']" v-if="nowPlaying.image[2]['#text']">
                  <img src="/static/no-album-art.png" v-else>
                </p>
              </figure>

              <div class="media-content">
                <h3 class="title is-3"><strong>{{ nowPlaying.name }}</strong></h3>
                <h4 class="title is-4"><em>{{ nowPlaying.artist['#text'] }}</em></h4>
                <h4 class="subtitle is-4">{{ nowPlaying.album['#text'] }}</h4>
              </div>
            </article>
            
            <hr>
          </div>
          
          <div class="columns">
            <div class="column is-one-third">
              <div v-show="topTracks.length > 0">
                <h4 class="title is-4">Recent Tracks</h4>
                <article class="media" v-for="track in recentTracks">
                  <figure class="media-left">
                    <p class="image is-64x64">
                      <img :src="track.image[1]['#text']" v-if="track.image[1]['#text']">
                      <img src="/static/no-album-art.png" v-else>
                    </p>
                  </figure>
                  <div class="media-content">
                    <h5 class="title is-5">
                      <strong>
                        <a :href="track.url" target="_blank">{{ track.name }}</a>
                        <small v-show="track['@attr']"><span class="loading">Now Playing</span></small>
                      </strong>
                      <br>
                      <em> {{ track.artist['#text'] }} </em>
                    </h5>
                  </div>
                </article>
              </div>
            </div>
            
            <div class="column is-one-third">
              <div v-show="topTracks.length > 0">
                <h4 class="title is-4">Top Artists</h4>
                <article class="media" v-for="artist in topArtists">
                  <figure class="media-left">
                    <p class="image is-64x64">
                      <img :src="artist.image[1]['#text']">
                    </p>
                  </figure>
                  <div class="media-content">
                    <h5 class="title is-5">
                      <span class="tag is-warning rank"><strong>{{ artist['@attr'].rank }}</strong></span>
                      <a :href="artist.url" target="_blank">{{ artist.name }}</a>
                      <span class="tag pull-right is-success"><i class="fa fa-fw fa-play"></i>{{ artist.playcount }}</span>
                    </h5>
                  </div>
                </article>
              </div>
            </div>
            
            <div class="column is-one-third">
              <div v-show="topTracks.length > 0">
                <h4 class="title is-4">Top Tracks</h4>
                <article class="media" v-for="track in topTracks">
                  <figure class="media-left">
                    <p class="image is-64x64">
                      <img :src="track.image[1]['#text']">
                    </p>
                  </figure>
                  <div class="media-content">
                    <h5 class="title is-5">
                      <strong><a :href="track.url" target="_blank">{{ track.name }}</a></strong>
                      <span class="tag pull-right is-success"><i class="fa fa-fw fa-play"></i>{{ track.playcount }}</span>
                      <br>
                      <em>{{ track.artist.name }}</em>
                    </h5>
                  </div>
                </article>
              </div>
            </div>
          </div>
        </div>

        <div class="container has-text-centered" v-else>
          <i class="fa fa-fw fa-lg fa-5x fa-spinner fa-spin"></i>
          <p>&nbsp;</p>
          <h3 class="title is-3"><span class="loading">Loading tuneden</span></p>
        </div>

      </div>
      
      <div class="hero-foot">
        <div class="container has-text-centered">
          <p>Copyright&copy;2017 Michael DeLally</p>
          <p>Powered by <a href="https://last.fm" class="is-white">last.fm</a></p>
          <p>&nbsp;</p>
        </div>
      </div>
    </section>
</template>

<script>
  export default {
    data() {
      return {
          loaded: false,
          user: "mdelally",
          topTracks: [],
          topArtists: [],
          recentTracks: [],
          nowPlaying: {}
        }
      },
      
      mounted() {
        this.getUserData();

        setInterval(() => {
          this.checkNowPlaying();
          console.log("New song..?")
        }, 10000);
      },

      methods: {
        getUserData() {
          this.loaded = false;

          this.getRecentTracks();
          this.getTopTracks();
          this.getTopArtists();
        },
        checkNowPlaying() {
          this.$http.get("https://ws.audioscrobbler.com/2.0/?method=user.getrecenttracks&user=" + this.user + "&api_key=e17e747cf811296552bbea1b4bb4dd17&format=json&limit=1")
          .then(response => {
            let now = response.body.recenttracks.track[0]['@attr'] || false;
            if (now) {
              this.nowPlaying = response.body.recenttracks.track[0];
              this.getRecentTracks();
            }
          }, reponse => {
            //
          });
        },
        getTopTracks() {
          this.$http.get("https://ws.audioscrobbler.com/2.0/?method=user.gettoptracks&user=" + this.user + "&api_key=e17e747cf811296552bbea1b4bb4dd17&format=json")
          .then(response => {
            // console.log(response.body.toptracks.track[0]);
            this.topTracks = response.body.toptracks.track;
          }, reponse => {
            this.topTracks = [];
          });
        },
        getTopArtists() {
          this.$http.get("https://ws.audioscrobbler.com/2.0/?method=user.gettopartists&user=" + this.user + "&api_key=e17e747cf811296552bbea1b4bb4dd17&format=json")
          .then(response => {
            // console.log(response);
            // console.log(response.body.topartists.artist[0]);
            this.topArtists = response.body.topartists.artist;
          }, reponse => {
            this.topArtists = [];
          });
        },
        getRecentTracks() {
          this.$http.get("https://ws.audioscrobbler.com/2.0/?method=user.getrecenttracks&user=" + this.user + "&api_key=e17e747cf811296552bbea1b4bb4dd17&format=json")
          .then(response => {
            // console.log(response.body.recenttracks.track);
            this.recentTracks = response.body.recenttracks.track;
            let now = response.body.recenttracks.track[0]['@attr'] || false;
            if (now) {
              this.nowPlaying = response.body.recenttracks.track[0];
              this.recentTracks.splice(0, 1);
            }
            this.loaded = true;
          }, reponse => {
            this.recentTracks = [];
          });
        }
      }
  }
</script>

<style>
  body {
    text-shadow: 0px 0px 3px rgb(0,0,0);
  }

  .media {
    color: white;
  }

  .loading:after {
    overflow: hidden;
    display: inline-block;
    vertical-align: bottom;
    -webkit-animation: ellipsis steps(4,end) 1200ms infinite;      
    animation: ellipsis steps(4,end) 900ms infinite;
    content: "\2026"; /* ascii code for the ellipsis character */
    width: 0px;
  }

  @keyframes ellipsis {
    to {
      width: 1.25em;    
    }
  }

  @-webkit-keyframes ellipsis {
    to {
      width: 1.25em;    
    }
  }

  .is-white {
    color: white;
  }

  .rank {
    text-shadow: none;
    font-weight: 700;
  }
  
</style>