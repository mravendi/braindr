<template name="tutorial">
  <div class="tutorial">
    <div class="">
      <h1 v-if="!currentImage"> Tutorial</h1>
      <div v-if="!currentImage">
        <p>
          Your objective is to classify these slices as "Pass" or "Fail" by swiping right or left
        </p>

        <p>
          Any similarities to the Tinder dating app are completely coincidental
        </p>

        <p>
          You get points for each image you rate.
        </p>

        <p>
          You can unlock the following animals based on the number of points you have:
        </p>

        <b-container fluid class="p-4">
          <b-row>
            <b-col v-for="lev in levels" v-if="lev.img && lev.level <= 3">
              <div>
                <b-img fluid class="pokemon" :src="lev.img_grey" alt="Thumbnail"/>
                <br>
                Level {{lev.level}}
                <br>
                <span>{{lev.min}} points</span>
              </div>
            </b-col>
          </b-row>
        </b-container>
      </div>
      <p class="lead">
        <span :class="{ '': this.currentType === null, 'text-danger': this.currentType === 0, 'text-success': this.currentType === 1 }">{{message}}</span>
      </p>
      <b-button @click="setImage(1)" v-if="!currentImage" variant="primary">Start</b-button>
      <b-button to="/play" v-if="count >= 5" class="mb-3" variant="primary">Start Playing</b-button>
      <transition :key="swipe" :name="swipe">
        <div class="user-card" :key="currentType" v-if="currentImage">
            <div class="image_area">
              <img class="user-card__picture mx-auto" :src="currentImage.pic"
              v-hammer:swipe.horizontal="onSwipe"
              ></img>
            </div>
          <div class="user-card__name">
            <b-button variant="danger"
              style="float:left"
              @click="swipeLeft"
              v-shortkey="['arrowleft']"
              @shortkey="swipeLeft"
              v-hammer:swipe.left="swipeLeft"
            > <i class="fa fa-long-arrow-left" aria-hidden="true"></i>  Fail </b-button>
            <span class="align-middle">Fail or Pass</span>
            <b-button variant="success"
              style="float:right"
              @click="swipeRight"
              v-shortkey="['arrowright']"
              @shortkey="swipeRight"
            > Pass <i class="fa fa-long-arrow-right" aria-hidden="true"></i> </b-button>
          </div>
        </div>
      </transition>

  </div>
</div>

</template>

<style>
/*https://github.com/pudymody/tinderSwipe/blob/gh-pages/style.css*/
.user-card {
    max-width: 500px;
    height: fit-content;
    width: 100%;
    border: 1px solid #ccc;
    padding: 8px;
    box-shadow: 0px 2px 5px 0px #ccc;
    position: absolute;
    left: 0;
    right: 0;
    margin: auto
}

.user-card__picture {
    width: 100%;
    display: block;
}

.image_area {
  background: black;
}

.user-card__name {
    margin-bottom: 0;
    margin-top: 8px;
}

.swipe-left {
    -webkit-animation: swipe-left 1s forwards;
            animation: swipe-left 1s forwards;
}

.swipe-right {
    -webkit-animation: swipe-right 1s forwards;
            animation: swipe-right 1s forwards;
}

@-webkit-keyframes swipe-left {
    to {
        -webkit-transform: rotate(-13deg) translate3d(-100%, 0, 0);
                transform: rotate(-13deg) translate3d(-100%, 0, 0);
        opacity: 0;
    }
}

@keyframes swipe-left {
    to {
        -webkit-transform: rotate(-13deg) translate3d(-100%, 0, 0);
                transform: rotate(-13deg) translate3d(-100%, 0, 0);
        opacity: 0;
    }
}

@-webkit-keyframes swipe-right {
    to {
        -webkit-transform: rotate(13deg) translate3d(100%, 0, 0);
                transform: rotate(13deg) translate3d(100%, 0, 0);
        opacity: 0;
    }
}

@keyframes swipe-right {
    to {
        -webkit-transform: rotate(13deg) translate3d(100%, 0, 0);
                transform: rotate(13deg) translate3d(100%, 0, 0);
        opacity: 0;
    }
}

/* Enter and leave animations can use different */
/* durations and timing functions.              */
.swipe-right-enter-active {
  transition: all .3s ease;
}

.swipe-right-enter-to {
  transition: all .3s ease;
}

.swipe-right-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.swipe-right-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  -webkit-transform: rotate(13deg) translate3d(100%, 0, 0);
          transform: rotate(13deg) translate3d(100%, 0, 0);
  opacity: 0;
}
.swipe-left-enter-active {
  transition: all .3s ease;
}
.swipe-left-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.swipe-left-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  -webkit-transform: rotate(-13deg) translate3d(-100%, 0, 0);
          transform: rotate(-13deg) translate3d(-100%, 0, 0);
  opacity: 0;
}

.toast {
  width: auto;
  max-width: 300px;
  top: 60px;
  left: 0;
  margin: auto;
  position: absolute;
  right: 0;
}

.right {
  color: green;
}
</style>

<script>
import Vue from 'vue';
import { VueHammer } from 'vue2-hammer';
import _ from 'lodash';
import { db } from '../firebaseConfig';

function randomInt(min, max) {
  return Math.floor(Math.random() * ((max - min) + 1)) + min;
}

Vue.use(VueHammer);

export default {
  name: 'tutorial',
  firebase: {
    imgCounts: db.ref('imageCount').orderByChild('num_votes').startAt(1).limitToFirst(50),
  },
  data() {
    return {
      imgCounts: [],
      currentType: null,
      currentImage: null,
      startTime: 0,
      swipe: null,
      count: 0,
    };
  },
  computed: {
    passes() {
      return _.filter(this.imgCounts, val => val.ave_score > 0.7);
    },
    fails() {
      return _.filter(this.imgCounts, val => val.ave_score < 0.3);
    },
    message() {
      if (this.currentType === null) {
        return 'Ready to train?';
      } else if (this.currentType === 1) {
        return 'Swipe Right (or press right arrow) to Pass this image';
      }

      return 'Swipe Left (or press left arrow) to Fail this image';
    },
  },
  components: { VueHammer },
  props: ['levels'],
  watch: {
    count() {
      if (this.count === 5) {
        this.$emit('taken_tutorial', true);
      }
    },
  },
  methods: {
    swipeLeft() {
      if (this.currentType === 0) {
        this.setSwipe('swipe-left');
        this.setImage(1);
        this.count += 1;
      } else {
        console.log('wrong way!');
      }
    },
    swipeRight() {
      if (this.currentType === 1) {
        this.setSwipe('swipe-right');
        this.setImage(0);
        this.count += 1;
      } else {
        console.log('wrong way!');
      }
    },
    setImage(type) {
      let img = null;
      if (type) {
        const N = this.passes.length;
        const rando = randomInt(0, N - 1);
        img = this.passes[rando];
      } else {
        const N = this.fails.length;
        const rando = randomInt(0, N - 1);
        img = this.fails[rando];
      }
      db.ref('images').child(img['.key']).once('value').then((snap) => {
        this.currentImage = snap.val();
        this.startTime = new Date();
        this.currentType = type;
      });
    },
    setSwipe(sw) {
      console.log('setting swipe', sw);
      this.swipe = sw;
    },
    onSwipe(evt) {
      if (evt.direction === 2) {
        this.swipeLeft();
      } else {
        this.swipeRight();
      }
    },
  },
};
</script>
