<template>
	<div class="swiper-slide">
		<div class="wechatSwiper">
			<div class="swiper-wrapper">
				<div class="swiper-slide" v-for="message in messages" track-by="$index" :id=`slide${message.index}`>
					<div class="top-container" v-if="$index === 0">
						<img src="../../static/wechat-top.png" class="wechat-topbg">
						<img v-if="user.avatar" :src='user.avatar' class="avatar-user">
						<img v-else src='../../static/avatar-user.png' class="avatar-user">
						<p class="username">{{ user.name }}</p>
					</div>
					<div class="message">
						<img class="friend-avatar" :src='message.avatar'>
						<div class="message-detail">
							<span class="friend-name">{{ message.username }}</span>
							<p class="message-content">{{ message.content }}</p>
              <img :src='message.img' class="detail-img">
							<p class="pic-location">{{ message.location }}</p>
							<div class="action-container">
								<p class="message-time">{{ message.time }}</p>
                <!--@click="showBtns($index)"-->
								<div class="icon-button-action">
									<div class="action-btns" v-bind:style="visible($index)" v-if="!isWrittingComment">
                    <!--@click="follow($index)"-->
										<button><div class="icon-heart-white btn-icon"></div>赞</button>
                    <!--@click="writeComment()"-->
										<button><div class="icon-comment btn-icon"></div>评论</button>
									</div>
								</div>
							</div>
							<div class="comment-container">
								<div class="follows">
									<div class="icon-heart"></div>
									<p>
										<span v-for='follow in message.follows'>{{ follow }}</span>
									</p>
								</div>
								<div class="comments" v-if='message.comments.length > 0'>
									<p v-for='comment in message.comments'>
										<span>{{ comment.user }}</span>
										<span v-if='comment.replyTo' class="normal-text">回复</span>
										<span>{{ comment.replyTo }}</span>: {{ comment.detail }}
									</p>
								</div>
							</div>
							<div :class="{ inputContainer: isWrittingComment, hideInput: !isWrittingComment }" v-if="activeBtnId === $index">
								<input type="text" :class="{ activeInput: isWrittingComment, hideInput: !isWrittingComment }" v-model="commentInput" @keyup.enter="submit"
									id="bottomInput" placeholder="请输入需要发表评论">
                  <!--@click="submit()"-->
								<button :class="{ activeSubmitBtn: isWrittingComment, hideInput: !isWrittingComment }">发送</button>
							</div>
						</div>
					</div>
          <div v-if="showAngel && $index === 5" transition="opacity-animation" class="angel-container">
            <div class="angel-animation">
              <div class="gradientMask" v-if="showGradientMask" transition="opacity-animation"></div>
              <div class="hand-up">
              <img src="../../static/arrow.png" class="arrow">
              <img src="../../static/hand.png" class="hand">
            </div>
            </div>
          </div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
  import Swiper from '../../static/swiper.js';
  import messageData from '../utils/mock-data.js';
  import { $, User } from '../utils/utils.js';
  let wechatSwiper;
  // let slideNextTimeout;
  export default {
    data() {
      return {
        user: { name: User.name, avatar: User.avatar },
        messages: messageData,
        activeBtnId: null,
        commentInput: null,
        shouldHideBtn: false,
        isWxCircles: true,
        isWrittingComment: false,
        hasReturned: false,
        moveStartY: 0,
        // hasSlideNext: false,
        hasSlidePrev: false,
        activeBannerIndex: 1,
        showAngel: false,
        showGradientMask: true,
        // simulateTouch: false,
      };
    },
    ready() {
      this.isWxCircles = true;
      this.initSwiper();
      wechatSwiper.stopAutoplay();
      // wechatSwiper.disableTouchControl();
      window.addEventListener('click', () => {
        if (!this.shouldHideBtn) return;
        const clearActionBtn = setTimeout(() => {
          this.activeBtnId = null;
          this.shouldHideBtn = false;
          clearTimeout(clearActionBtn);
        }, 200);
      });
      window.addEventListener('touchend', (e) => {
        const actionBtn = $('.icon-button-action')[0];
        const actionBtnDiv = $('.action-btns');
        const actionIcon = $('.btn-icon');
        const btn = $('button');
        const input = $('input');
        const exculdeEl = [actionBtn, ...btn, ...input, ...actionIcon, ...actionBtnDiv];
        const shouldExculde = exculdeEl.find(x => x === e.target);
        if (shouldExculde || !this.isWxCircles) return;
        this.hideAction();
      });
    },
    events: {
      // 'initHasSlideNext'() {
      //   this.hasSlideNext = false;
      // },
      'initHasSlidePrev'() {
        this.hasSlidePrev = false;
      },
      'setMajorRelativePosition'(index) {
        // wechatSwiper.stopAutoplay();
        this.activeBannerIndex = index;
        const newMessageData = messageData.filter((data) => data.index !== index);
        this.messages = [messageData[index - 1], ...newMessageData];
        // const startPlayTimeout = setTimeout(() => {
        //   wechatSwiper.startAutoplay();
        //   clearTimeout(startPlayTimeout);
        // }, 2000);
        // wechatSwiper.slideTo((index - 1), 0);
        // if (index !== 1)
        // wechatSwiper.setWrapperTranslate(wechatSwiper.getWrapperTranslate() + 256);
      },
    },
    methods: {
      showBtns(index) {
        this.activeBtnId = index;
      },
      visible(id) {
        let btnClass;
        if (id !== this.activeBtnId) {
          btnClass = { display: 'none' };
        } else {
          btnClass = { display: '' };
        }
        return btnClass;
      },
      follow(index) {
        // 查找是否包含自己
        const isExist = this.messages[index].follows.find(x => x === '我');
        this.shouldHideBtn = true;
        if (isExist) return;
        this.messages[index].follows = [...this.messages[index].follows, '我'];
      },
      writeComment() {
        this.isWrittingComment = true;
        const focusTimeout = setTimeout(() => {
          $('#bottomInput')[0].focus();
          clearTimeout(focusTimeout);
        }, 200);
      },
      submit() {
        if (this.commentInput !== null) {
          this.messages[this.activeBtnId].comments = [...this.messages[this.activeBtnId].comments, {
            user: '我', replyTo: null, detail: this.commentInput,
          }];
        }
        this.isWrittingComment = false;
        this.activeBtnId = null;
        this.commentInput = null;
      },
      hideAction() {
        this.isWrittingComment = false;
        this.activeBtnId = null;
        this.commentInput = null;
      },
      initSwiper() {
        wechatSwiper = new Swiper('.wechatSwiper', {
          direction: 'vertical',
          nested: true,
          freeMode: true,
          // freeModeSticky: true,
          // freeModeMomentum: false,
          // resistance: true,
          // resistanceRatio: 0.9,
          autoHeight: true,
          slidesPerView: 'auto',
          // autoplay: 1,
          // autoplayDisableOnInteraction: false,
          // speed: 6000,
          // autoplayStopOnLast: true,
          loop: false,
          hashnav: true,
          // onReachBeginning: (swiper) => {
          //   if (swiper.activeIndex === 0) {
          //     this.$dispatch('slideMajor');
          //   }
          // },
          onReachEnd: () => {
            this.showAngel = true;
            // slideNextTimeout = setTimeout(() => {
            //   this.$dispatch('slideNextMajor');
            //   clearTimeout(slideNextTimeout);
            // }, 3000);
            const hideGradient = setTimeout(() => {
              this.showGradientMask = false;
              clearTimeout(hideGradient);
            }, 2000);
          },
          onTouchStart: (swiper, event) => {
            if (swiper.activeIndex === 4 || swiper.activeIndex === 0) {
              this.moveStartY = event.changedTouches[0].pageY;
            }
          },
          onTouchEnd: () => {
            if (this.activeBannerIndex !== 1) {
              this.activeBannerIndex = 1;
            }
          },
          onSliderMove: (swiper, event) => {
            if (swiper.activeIndex === 5) {
              const moveDistance = this.moveStartY - event.changedTouches[0].pageY;
              if (moveDistance > 20 && this.moveStartY !== 0) {
                this.$dispatch('slideNextMajor');
                // this.hasSlideNext = true;
                this.moveStartY = 0;
              }
            }
            // if (swiper.activeIndex === 0) {
            //   const moveDistance = event.changedTouches[0].pageY - this.moveStartY;
            //   if (moveDistance > 200 && !this.hasSlidePrev && this.moveStartY !== 0) {
            //     this.$dispatch('slideMajor');
            //     this.hasSlidePrev = true;
            //     this.moveStartY = 0;
            //   }
            // }
          },
        });
      },
    },
  };
</script>

<style scoped>
	.wechatSwiper {
		width: 100%;
		height: 100%;
    background-color: #FFF;
	}

	.wechatSwiper .swiper-wrapper {
		width: 100%;
    position: relative;
	}

	.wechatSwiper .swiper-slide {
		width: 100%;
    position: relative;
	}

	.wx-circles {
		position: relative;
		overflow-y: scroll;
	}

	.top-container {
		/*background: url('../images/bg-wx-circles.png') no-repeat 100% 100%;
		background-size: cover;*/
    background-color: #021844;
		/*width: 100%;
		height: 256px;*/
		position: relative;
		overflow: visible;
		text-align: center;
		margin-bottom: 30px;
	}

  .wechat-topbg {
    width: 100%;
    height: auto;
    display: block;
    vertical-align: top;
  }

	.icon-2022 {
		margin: 0 auto;
		position: relative;
		top: 53px;
	}

	.avatar-user {
		border: 2px solid #FFF;
		position: absolute;
		bottom: -20px;
		right: 12px;
		width: 70px;
		height: 70px;
		display: inline-block;
		vertical-align: top;
		background-color: transparent;
	}

	.username {
		color: #FFF;
		font-size: 15px;
		margin: 0;
		position: absolute;
		right: 105px;
		bottom: 8px;
	}

	.message {
		padding: 20px 12px;
		border-bottom: 1px solid #E1E2DF;
	}

	.friend-avatar {
		width: 42px;
		height: 42px;
		display: inline-block;
		vertical-align: top;
		float: left;
	}

	.friend-name {
		color: #576B95;
		font-size: 15px;
	}

	.message-detail {
		display: inline-block;
		width: 80%;
		margin: 0 0 0 9px;
	}

	.message-content {
		margin: 0;
		font-size: 15px;
	}

	.detail-img {
		max-width: 100%;
		margin: 5px 0;
	}

	.pic-location {
		color: #576B95;
		font-size: 12px;
		margin: 0;
	}

	.action-container {
		position: relative;
	}

	.message-time {
		color: #737373;
		font-size: 12px;
		display: inline-block;
	}

	.icon-button-action {
		position: absolute;
		display: inline-block;
		top: 0;
		right: -10px;
		overflow: visible;
	}

	.action-btns {
		position: absolute;
		left: -165px;
		top: -10px;
		border-radius: 5px;
		background-color: #353a3d;
		z-index: 2;
		display: visible;
	}

	.activeBtn {
		display: visible;
	}

	.action-btns button {
		display: inline-block;
		float: left;
		width: 80px;
		height: 40px;
		color: #FFF;
		outline: none;
		border: none;
		background-color: transparent;
		vertical-align: middle;
		font-size: 14px;
		font-weight: normal;
		position: relative;
	}

	.commentInput {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: transparent;
		z-index: 1;
		border: none;
		outline: none;
		color: transparent;
	}

	.hideInput {
		width: 0;
		height: 0;
		background-color: transparent;
		color: transparent;
		border: none;
		outline: none;
		padding: 0;
		margin: 0;
		font-size: 0;
	}

	.inputContainer {
		width: 100%;
		background-color: #FFF;
	}

	.activeInput {
		padding: 15px;
		background-color: #FFF;
		border: 1px solid #EBEBEB;
		outline: none;
		color: #4A4A4A;
		border-radius: 5px;
		position: relative;
		z-index: 2;
	}

	.activeSubmitBtn {
		color: #4A4A4A;
		background-color: #F3F3F5;
		font-size: 14px;
		text-align: center;
		padding: 10px;
		border: none;
		border-radius: 5px;
		position: relative;
		z-index: 2;
	}

	.btn-icon {
		display: inline-block;
		vertical-align: top;
		margin-right: 4px;
	}

	.comment-container {
		margin: 10px 0;
		padding: 4px 6px;
		background-color: #F3F3F5;
		position: relative;
	}

	.comments {
		border-top: 1px solid #E1E2DF;
		padding: 2px 0;
		font-size: 15px;
	}

	.comments span {
		color: #576B95;
		font-weight: bold;
		font-size: 15px;
	}

	.icon-heart {
		display: inline-block;
		vertical-align: top;
		position: relative;
		top: 4px;
	}

	.follows {
		padding-bottom: 2px;
	}

	.follows p {
		display: inline-block;
		width: 90%;
		color: #576B95;
		font-size: 15px;
	}

	.comments .normal-text {
		color: #000;
		font-size: 15px;
		font-weight: normal;
	}

	.comment-container:before {
		content: '';
		display: block;
		position: absolute;
		left: 4;
		top: -8px;
		width: 0;
		height: 0;
		border-right: 6px solid transparent;
		border-left: 6px solid transparent;
		border-bottom: 8px solid #F3F3F5;
	}
  .angel-container {
    position: absolute;
    bottom: 0;
    width: 100%;
    z-index: 4;
  }
  .angel-animation {
    /*background: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,.6));*/
    width: 100%;
    text-align: center;
    position: relative;
  }
  .gradientMask {
    background: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,.6));
    background: -webkit-linear-gradient(rgba(0,0,0,0), rgba(0,0,0,.6));
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }
  .hand-up {
    margin: 5px auto;
    display: inline-block;
    height: 60px;
    position: relative;
  }
  .hand-up img {
    display: inline-block;
  }
  @keyframes rise-arrow {
    0% { transform: scaleY(0); opacity: 0; }
    100% { transform: scaleY(1); opacity: 1;}
  }
  @-webkit-keyframes rise-arrow {
    0% { -webkit-transform: scaleY(0); opacity: 0;}
    100% { -webkit-transform: scaleY(1); opacity: 1;}
  }
  .arrow {
    width: 18px;
    height: 56px;
    vertical-align: bottom;
    animation: rise-arrow 1.2s linear infinite;
    -webkit-animation: rise-arrow 1.2s linear infinite;
    /*position: absolute;
    bottom: 0;*/
    transform-origin: center bottom;
    -webkit-transform-origin: center bottom;
  }
  @keyframes rise {
    0% { transform: translateY(40px); }
    100% { transform: translateY(-10); }
  }
  @-webkit-keyframes rise {
    0% {-webkit-transform: translateY(40px);}
    100% {-webkit-transform: translateY(-10);}
  }
  .hand {
    width: 48px;
    height: 34px;
    animation: rise 1.2s linear infinite;
    -webkit-animation: rise 1.2s linear infinite;
  }
  .up-angel {
    width: 48px;
    height: 48px;
    border: 2px solid #4EDEFD;
    border-width: 2px 2px 0 0;
    transform: rotate(135deg);
    -webkit-transform: rotate(135deg);
    margin: 0 auto;
  }
  .opacity-animation-transition {
    transition: all 1s ease;
    -webkit-transition: all 1s ease;
    /*overflow: hidden;*/
  }
  .opacity-animation-enter, .opacity-animation-leave {
    opacity: 0;
  }
</style>
