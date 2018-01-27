<template>
  <v-app>
    <v-navigation-drawer
      fixed
      :mini-variant="miniVariant"
      :clipped="clipped"
      v-model="drawer"
      app
    >
      <v-list>
        <v-list-tile 
          value="true"
          v-for="(item, i) in items"
          :key="i"
        >
          <v-list-tile-action>
            <v-icon v-html="item.icon"></v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title v-text="item.title"></v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar fixed app :clipped-left="clipped">
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-btn icon @click.stop="miniVariant = !miniVariant">
        <v-icon v-html="miniVariant ? 'chevron_right' : 'chevron_left'"></v-icon>
      </v-btn>
      <v-btn icon @click.stop="clipped = !clipped">
        <v-icon>web</v-icon>
      </v-btn>
      <v-btn icon @click.stop="fixed = !fixed">
        <v-icon>remove</v-icon>
      </v-btn>
      <v-toolbar-title v-text="title"></v-toolbar-title>
    </v-toolbar>
    <v-content>
      <v-container fluid>
          <v-layout row aline-center >
            <v-flex xs12 sm6 >
                  <v-card>
                    <v-toolbar  color="red">
                        <v-text-field
                          type='text'
                          v-model='search'
                          label="Type and Enter"
                          prepend-icon="search"
                          @keyup.enter='getResult'
                        >
                        </v-text-field>
                    </v-toolbar>
                    <transition name='fade'>
                      <v-list two-line>
                        <template v-for="item in search_items">
                          <v-divider></v-divider>
                          <v-list-tile @click='getId(item)'>
                            <v-list-tile-avatar>
                              <img :src='item.snippet.thumbnails.high.url'>
                            </v-list-tile-avatar>
                            <v-list-tile-content>
                              <v-list-tile-title>{{item.snippet.title}}</v-list-tile-title>
                              <v-list-tile-sub-title>{{item.snippet.description}} </v-list-tile-sub-title>
                            </v-list-tile-content>
                          </v-list-tile>
                        </template>
                      </v-list>
                    </transition>
                  </v-card>
                </v-flex>
          </v-layout>
      </v-container>
    </v-content>
    <!-- video playing and comments section -->
    <v-container fluid ref='playSection'>
      <!-- for playing videos -->
      <v-layout row>
          <v-flex xs12 sm6 >
            <v-card>
              <v-flex d-flex>
                <v-card-media>
                  <!-- custom section to play youtube video -->
                  <youtube :video-id="videoId"></youtube>
                </v-card-media>
              </v-flex>
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">{{selectedVideoDetail.title}} </h3>
                  <div>{{selectedVideoDetail.description}}</div>
                </div>
              </v-card-title>
              <v-card-actions>
                <v-btn flat color="orange">Share</v-btn>
                <v-btn flat color="orange">Explore</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
          <!-- related video section or playlist item section -->
          <v-flex xs12 sm6>
            <v-card>
              <v-toolbar color="cyan" dark>
                <v-menu offset-y>
                      <v-toolbar-side-icon slot="activator"></v-toolbar-side-icon>
                      <v-list>
                        <v-list-tile v-for="item in items" :key="item.title" @click="">
                          <v-list-tile-title>{{ item.title }}</v-list-tile-title>
                        </v-list-tile>
                      </v-list>
                    </v-menu>
                <v-toolbar-title>Related Videos</v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>
              <v-list two-line v-for="item in relatedVideos">
                <template >
                  <v-subheader v-if="item.snippet.title" v-text="item.snippet.title"></v-subheader>
                  <v-divider></v-divider>
                  <v-list-tile @click='getId(item)' avatar>
                    <v-list-tile-avatar>
                      <img v-bind:src="item.snippet.thumbnails.high.url">
                    </v-list-tile-avatar>
                    <v-list-tile-content>
                      <v-list-tile-title v-html="item.snippet.title"></v-list-tile-title>
                      <v-list-tile-sub-title v-html="item.snippet.description"></v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                </template>
              </v-list>
            </v-card>
          </v-flex>
        </v-layout>
    </v-container>
    <v-footer :fixed="fixed" app>
      <span>&copy; 2017</span>
    </v-footer>
  </v-app>
</template>

<script>
import axios from 'axios'
  export default {
    data () {
      return {
        playlistId: '',
        videoId: '',
        clipped: false,
        drawer: false,
        fixed: false,
        items: [
          { icon: 'bubble_chart', title: 'Inspire' }
        ],
        miniVariant: false,
        title: 'Youtube App',
        search_items: [],
        search:'',
        relatedVideos: [],
        selectedVideoDetail:[]
      }
    },
    methods: {
      getResult() {
        var q = ""+this.search
        var url = 'https://www.googleapis.com/youtube/v3/search?q='+q+'&key=AIzaSyAKyA5GvavFeXTOAdhvn-ZqKp7l9dK50BI&maxResults=5&part=snippet'
        axios.get(url)
        .then((res) => {
          console.log(res.data.items)
          this.search_items = res.data.items
        })
        .catch((err) => {
          console.log(err)
        })
      },
      getId(videoInfo) {
        this.videoId = videoInfo.id.videoId
        this.playlistId = videoInfo.id.playlistId
        if (this.videoId) {
          this.selectedVideoDetail = videoInfo.snippet
          this.$refs.playSection.scrollIntoView({behavior:'smooth'})
          // console.log(videoInfo.snippet)
          // to get the related videos
          axios.get('https://www.googleapis.com/youtube/v3/search?relatedToVideoId='+this.videoId+'&key=AIzaSyAKyA5GvavFeXTOAdhvn-ZqKp7l9dK50BI&maxResults=5&part=snippet&type=video')
          .then((res) => {
            console.log(res.data.items)
            this.relatedVideos = res.data.items
          })
          .catch((err) => {
            console.log(err)
          })
        }
        // to get the playlist items and show them
        else if (this.playlistId) {
          this.selectedVideoDetail = videoInfo.snippet
          this.$refs.playSection.scrollIntoView({behavior:'smooth'})
          // console.log(videoInfo.snippet)
          // to get the related videos
          axios.get('https://www.googleapis.com/youtube/v3/playlistItems?playlistId='+this.playlistId+'&key=AIzaSyAKyA5GvavFeXTOAdhvn-ZqKp7l9dK50BI&maxResults=5&part=snippet,contentDetails')
          .then((res) => {
            console.log(res.data.items)
            this.relatedVideos = res.data.items
          })
          .catch((err) => {
            console.log(err)
          })
        }
        // get the video id from the and play the video
        else if (videoInfo.contentDetails.videoId) {
          this.videoId = videoInfo.contentDetails.videoId
        }
      }
    },
  }
</script>

<style>
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
</style>