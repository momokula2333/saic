<template>
	<div class="swiper-slide vertical-pages">
		<div class="swiper-wrapper">
      <slogan></slogan>
      <runway></runway>
			<space></space>
		</div>
	</div>
</template>

<script>
  import Swiper from '../../static/swiper.js';
  import {
    swiperAnimateCache,
    swiperAnimate,
  } from '../utils/swiper.animate.min.js';
  import Slogan from './vertical/Slogan';
  import Runway from './vertical/Runway';
  import Space from './vertical/Space';
  let verticalTopSwiper;

  export default {
    data() {
      return {
        moveStartY: 0,
        moveStartX: 0,
        selectedMajor: 1,
      };
    },
    components: {
      Slogan,
      Runway,
      Space,
    },
    ready() {
      verticalTopSwiper = new Swiper('.vertical-pages', {
        onInit: (swiper) => {
          swiperAnimateCache(swiper);
          this.$broadcast('startAnimation');
        },
        onSlideChangeStart: () => {
          this.$broadcast('initAnimation');
        },
        onSlideChangeEnd: (swiper) => {
          swiperAnimate(swiper);
          if (swiper.activeIndex === 2) {
            // verticalTopSwiper.unlockSwipeToPrev();
            this.$broadcast('startCarAnimation', this.selectedMajor);
          }
          if (swiper.activeIndex === 0 && swiper.previousIndex === 1) {
            // verticalTopSwiper.unlockSwipeToPrev();
            this.$broadcast('setToBottom');
            this.$dispatch('changeEarth');
            this.$broadcast('reAnimateSlogan');
            // this.$dispatch('playMusic1');
          }
          if (swiper.activeIndex === 1) {
            // verticalTopSwiper.lockSwipeToPrev();
            verticalTopSwiper.lockSwipes();
            this.$broadcast('startRunwayCarAnimation');
            this.$dispatch('showEarth');
          }
        },
        // onTouchStart: (swiper, event) => {
        //   if (swiper.activeIndex === 2) {
        //     this.moveStartY = event.changedTouches[0].pageY;
        //   }
        //   if (swiper.activeIndex === 0) {
        //     this.moveStartX = event.changedTouches[0].pageX;
        //     this.moveStartY = event.changedTouches[0].pageY;
        //   }
        // },
        // onTouchMove: (swiper, event) => {
          // if (swiper.activeIndex === 2) {
          //   const moveDistance = event.changedTouches[0].pageY - this.moveStartY;
          //   if (moveDistance > 100) {
          //     this.$dispatch('goWechatMajor', this.selectedMajor);
          //     this.moveStartY = 0;
          //     return;
          //   }
          // }
          // if (swiper.activeIndex === 2) {
          //   const moveDistanceX = event.changedTouches[0].pageX - this.moveStartX;
          //   const moveDistanceY = event.changedTouches[0].pageY - this.moveStartY;
          //   if (moveDistanceX > 100 && moveDistanceY < 100) {
          //     this.$dispatch('verticalToHorizen');
          //     this.moveStartX = 0;
          //     this.moveStartY = 0;
          //     return;
          //   }
          // }
        // },
        initialSlide: 0,
        direction: 'vertical',
        loop: false,
        autoHeight: true,
        hashnav: true,
      });
    },
    events: {
      'startPageAnimation'() {
        swiperAnimate(verticalTopSwiper);
      },
      'startCarAnimation'() {
        swiperAnimate(verticalTopSwiper);
        this.$broadcast('startCarAnimation', this.selectedMajor);
      },
      'slideToSpace'(index) {
        verticalTopSwiper.unlockSwipeToNext();
        this.selectedMajor = index;
        verticalTopSwiper.slideTo(2);
      },
      'goSelectMajorPage'() {
        if (verticalTopSwiper) verticalTopSwiper.slideTo(1, 0);
      },
      'unlockRunwayToPrev'() {
        if (verticalTopSwiper) verticalTopSwiper.unlockSwipeToPrev();
      },
    },
  };
</script>

<style scoped>
	@import '../assets/swiper-3.3.1.min.css';
	@import '../assets/animate.min.css';
	.horizontal-pages {
		height: 100%;
		width: 100%;
	}

  .horizontal-pages .swiper-wrapper {
    height: 100%;
		width: 100%;
  }

	.horizontal-pages .swiper-slide {
		overflow: hidden;
    height: 100%;
    width: 100%;
    text-align: center;
	}

  .demoBg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
  }

  .demoText, .demoTit {
    font-size: 20px;
    color: #FFF;
  }

  .demoTit {
    margin-top: 50%;
  }

</style>
