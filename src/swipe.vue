<style>
  .mint-swipe {
    overflow: hidden;
    position: relative;
    width: 100%;
    height: 100%;

  }
  .mint-swipe-items-wrap {
    position: relative;
    overflow: hidden;
    width: 100%;
    height: 100%;
    -webkit-transform: translateZ(0);
    transform: translateZ(0)
  }
  .mint-swipe-items-wrap > div {
    position: absolute;
    -webkit-transform: translateY(-100%);
    transform: translateY(-100%);
    width: 100%;
    height: 100%;
    display: none
  }
  .mint-swipe-items-wrap > div.is-active {
    display: block;
    -webkit-transform: none;
    transform: none;
  }
  .mint-swipe-indicators {
    position: absolute;
    right: 8px;
    top: 50%;
    -webkit-transform: translateY(-50%);
    transform: translateY(-50%);
  }
  .mint-swipe-indicator {
    width: 8px;
    height: 8px;
    display: inline-block;
    border-radius: 100%;
    background: #000;
    opacity: 0.2;
    margin: 0 3px;
  }
  .mint-swipe-indicator.is-active {
    background: #fff;
  }
</style>

<template>
  <div class="mint-swipe">
    <div class="mint-swipe-items-wrap" ref="wrap">
      <slot></slot>
    </div>
    <div class="mint-swipe-indicators" v-show="showIndicators">
      <div class="mint-swipe-indicator"
           v-for="(page, $index) in pages"
           :key="$index"
           :class="{ 'is-active': $index === index }"></div>
    </div>
  </div>
</template>

<script>
  import { once } from 'wind-dom/src/event';
  import { addClass, removeClass } from 'wind-dom/src/class';

  export default {
    name: 'mt-swipe',

    created() {
      this.dragState = {};
    },

    data() {
      return {
        ready: false,
        dragging: false,
        userScrolling: false,
        animating: false,
        index: 0,
        pages: [],
        timer: null,
        reInitTimer: null,
        noDrag: false
      };
    },

    props: {
      speed: {
        type: Number,
        default: 300
      },

      defaultIndex: {
        type: Number,
        default: 0
      },

      disabled: {
        type: Boolean,
        default: false
      },

      auto: {
        type: Number,
        default: 3000
      },

      continuous: {
        type: Boolean,
        default: true
      },

      showIndicators: {
        type: Boolean,
        default: true
      },

      noDragWhenSingle: {
        type: Boolean,
        default: true
      },

      prevent: {
        type: Boolean,
        default: false
      },

      propagation: {
        type: Boolean,
        default: false
      }

    },

    methods: {
      swipeItemCreated() {
        if (!this.ready) return;

        clearTimeout(this.reInitTimer);
        this.reInitTimer = setTimeout(() => {
          this.reInitPages();
        }, 100);
      },

      swipeItemDestroyed() {
        if (!this.ready) return;

        clearTimeout(this.reInitTimer);
        this.reInitTimer = setTimeout(() => {
          this.reInitPages();
        }, 100);
      },

      translate(element, offset, speed, callback) {
        if (speed) {
          this.animating = true;
          element.style.webkitTransition = '-webkit-transform ' + speed + 'ms ease-in-out';
          setTimeout(() => {
            element.style.webkitTransform = `translate3d(0, ${offset}px, 0)`;
          }, 50);

          var called = false;

          var transitionEndCallback = () => {
            if (called) return;
            called = true;
            this.animating = false;
            element.style.webkitTransition = '';
            element.style.webkitTransform = '';
            if (callback) {
              callback.apply(this, arguments);
            }
          };

          once(element, 'webkitTransitionEnd', transitionEndCallback);
          setTimeout(transitionEndCallback, speed + 100); // webkitTransitionEnd maybe not fire on lower version android.
        } else {
          element.style.webkitTransition = '';
          element.style.webkitTransform = `translate3d(0, ${offset}px, 0)`;
        }
      },

      reInitPages() {
        var children = this.$children;
        this.noDrag = children.length === 1 && this.noDragWhenSingle;

        var pages = [];
        this.index = this.defaultIndex;

        children.forEach((child, index) => {
          pages.push(child.$el);

          removeClass(child.$el, 'is-active');

          if (index === this.defaultIndex) {
            addClass(child.$el, 'is-active');
          }
        });

        this.pages = pages;
      },

      doAnimate(towards, options) {
        if (this.$children.length === 0) return;
        if (!options && this.$children.length < 2) return;

        var prevPage, nextPage, currentPage, pageHeight, offsetTop;
        var speed = this.speed || 300;
        var index = this.index;
        var pages = this.pages;
        var pageCount = pages.length;

        if (!options || towards === 'goto') {
          options = options || {};
          pageHeight = this.$el.clientHeight;
          currentPage = pages[index];
          if (towards === 'goto') {
            prevPage = options.prevPage;
            nextPage = options.nextPage;
          } else {
            prevPage = pages[index - 1];
            nextPage = pages[index + 1];
          }

          if (this.continuous && pages.length > 1) {
            if (!prevPage) {
              prevPage = pages[pages.length - 1];
            }
            if (!nextPage) {
              nextPage = pages[0];
            }
          }
          if (prevPage) {
            prevPage.style.display = 'block';
            this.translate(prevPage, -pageHeight);
          }
          if (nextPage) {
            nextPage.style.display = 'block';
            this.translate(nextPage, pageHeight);
          }
        } else {
          prevPage = options.prevPage;
          currentPage = options.currentPage;
          nextPage = options.nextPage;
          pageHeight = options.pageHeight;
          offsetTop = options.offsetTop;
        }

        var newIndex;

        var oldPage = this.$children[index].$el;

        if (towards === 'prev') {
          if (index > 0) {
            newIndex = index - 1;
          }
          if (this.continuous && index === 0) {
            newIndex = pageCount - 1;
          }
        } else if (towards === 'next') {
          if (index < pageCount - 1) {
            newIndex = index + 1;
          }
          if (this.continuous && index === pageCount - 1) {
            newIndex = 0;
          }
        } else if (towards === 'goto') {
          if (options.newIndex > -1 && options.newIndex < pageCount) {
            newIndex = options.newIndex;
          }
        }

        var callback = () => {
          if (newIndex !== undefined) {
            var newPage = this.$children[newIndex].$el;
            removeClass(oldPage, 'is-active');
            addClass(newPage, 'is-active');

            this.index = newIndex;

            this.$emit('change', newIndex, index);
          }

          if (prevPage) {
            prevPage.style.display = '';
          }

          if (nextPage) {
            nextPage.style.display = '';
          }
        };

        setTimeout(() => {
          if (towards === 'next') {
            this.translate(currentPage, -pageHeight, speed, callback);
            if (nextPage) {
              this.translate(nextPage, 0, speed);
            }
          } else if (towards === 'prev') {
            this.translate(currentPage, pageHeight, speed, callback);
            if (prevPage) {
              this.translate(prevPage, 0, speed);
            }
          } else if (towards === 'goto') {
            if (prevPage) {
              this.translate(currentPage, pageHeight, speed, callback);
              this.translate(prevPage, 0, speed);
            } else if (nextPage) {
              this.translate(currentPage, -pageHeight, speed, callback);
              this.translate(nextPage, 0, speed);
            }
          } else {
            this.translate(currentPage, 0, speed, callback);
            if (typeof offsetTop !== 'undefined') {
              if (prevPage && offsetTop > 0) {
                this.translate(prevPage, pageHeight * -1, speed);
              }
              if (nextPage && offsetTop < 0) {
                this.translate(nextPage, pageHeight, speed);
              }
            } else {
              if (prevPage) {
                this.translate(prevPage, pageHeight * -1, speed);
              }
              if (nextPage) {
                this.translate(nextPage, pageHeight, speed);
              }
            }
          }
        }, 10);
      },

      next() {
        this.doAnimate('next');
      },

      prev() {
        this.doAnimate('prev');
      },

      goto(newIndex) {
        if (this.index === newIndex) return;

        if (newIndex < this.index) {
          this.doAnimate('goto', {
            newIndex,
            prevPage: this.pages[newIndex]
          });
        } else {
          this.doAnimate('goto', {
            newIndex,
            nextPage: this.pages[newIndex]
          });
        }
      },

      doOnTouchStart(event) {
        if (this.noDrag || this.disabled) return;

        var element = this.$el;
        var dragState = this.dragState;
        var touch = event.changedTouches ? event.changedTouches[0] : event;

        dragState.startTime = new Date();
        dragState.startLeft = touch.pageX;
        dragState.startTop = touch.pageY;
        dragState.startTopAbsolute = touch.clientY;

        dragState.pageWidth = element.offsetWidth;
        dragState.pageHeight = element.offsetHeight;

        var prevPage = this.$children[this.index - 1];
        var dragPage = this.$children[this.index];
        var nextPage = this.$children[this.index + 1];

        if (this.continuous && this.pages.length > 1) {
          if (!prevPage) {
            prevPage = this.$children[this.$children.length - 1];
          }
          if (!nextPage) {
            nextPage = this.$children[0];
          }
        }

        dragState.prevPage = prevPage ? prevPage.$el : null;
        dragState.dragPage = dragPage ? dragPage.$el : null;
        dragState.nextPage = nextPage ? nextPage.$el : null;

        if (dragState.prevPage) {
          dragState.prevPage.style.display = 'block';
        }

        if (dragState.nextPage) {
          dragState.nextPage.style.display = 'block';
        }
      },

      doOnTouchMove(event) {
        if (this.noDrag || this.disabled) return;

        var dragState = this.dragState;
        var touch = event.changedTouches ? event.changedTouches[0] : event;

        dragState.currentLeft = touch.pageX;
        dragState.currentTop = touch.pageY;
        dragState.currentTopAbsolute = touch.clientY;

        var offsetLeft = dragState.currentLeft - dragState.startLeft;
        var offsetTop = dragState.currentTopAbsolute - dragState.startTopAbsolute;

        var distanceX = Math.abs(offsetLeft);
        var distanceY = Math.abs(offsetTop);
        if (distanceY < 5 || (distanceY >= 5 && distanceY >= 1.73 * distanceY)) {
          this.userScrolling = true;
          return;
        } else {
          this.userScrolling = false;
          event.preventDefault();
        }
        offsetTop = Math.min(Math.max(-dragState.pageHeight + 1, offsetTop), dragState.pageHeight - 1);

        var towards = offsetTop < 0 ? 'next' : 'prev';

        if (dragState.prevPage && towards === 'prev') {
          this.translate(dragState.prevPage, offsetTop - dragState.pageHeight);
        } else if (dragState.nextPage && towards === 'next') {
          this.translate(dragState.nextPage, offsetTop + dragState.pageHeight);
        } else {
          // when continuous=false and it's the end of each side,
          // limit swipe width with quadratic-functional ease
          // y = (-1 / dk) x (|x| - 2k)
          const k = dragState.pageHeight;
          const x = offsetTop;
          const d = 6; // scroll until 1/d of screenWidth at maximum
          offsetTop = -1 / d / k * x * (Math.abs(x) - 2 * k);
        }
        this.translate(dragState.dragPage, offsetTop);
      },

      doOnTouchEnd() {
        if (this.noDrag || this.disabled) return;

        var dragState = this.dragState;

        var dragDuration = new Date() - dragState.startTime;
        var towards = null;

        var offsetLeft = dragState.currentLeft - dragState.startLeft;
        var offsetTop = dragState.currentTop - dragState.startTop;
        var pageHeight = dragState.pageHeight;
        var index = this.index;
        var pageCount = this.pages.length;

        if (dragDuration < 300) {
          let fireTap = Math.abs(offsetLeft) < 5 && Math.abs(offsetTop) < 5;
          if (isNaN(offsetLeft) || isNaN(offsetTop)) {
            fireTap = true;
          }
          if (fireTap) {
            this.$children[this.index].$emit('tap');
          }
        }

        if (dragDuration < 300 && dragState.currentTop === undefined) return;

        if (dragDuration < 300 || Math.abs(offsetTop) > pageHeight / 2) {
          towards = offsetTop < 0 ? 'next' : 'prev';
        }

        if (!this.continuous) {
          if ((index === 0 && towards === 'prev') || (index === pageCount - 1 && towards === 'next')) {
            towards = null;
          }
        }

        if (this.$children.length < 2) {
          towards = null;
        }

        this.doAnimate(towards, {
          offsetTop: offsetTop,
          pageHeight: dragState.pageHeight,
          prevPage: dragState.prevPage,
          currentPage: dragState.dragPage,
          nextPage: dragState.nextPage
        });

        this.dragState = {};
      },

      dragStartEvent(event) {
        if (this.prevent) {
          event.preventDefault();
        }
        if (this.propagation) {
          event.stopPropagation();
        }
        if (this.animating) return;
        this.dragging = true;
        this.userScrolling = false;
        this.doOnTouchStart(event);
      },

      dragMoveEvent(event) {
        if (!this.dragging) return;
        this.doOnTouchMove(event);
      },

      dragEndEvent(event) {
        if (this.userScrolling) {
          this.dragging = false;
          this.dragState = {};
          return;
        }
        if (!this.dragging) return;
        this.doOnTouchEnd(event);
        this.dragging = false;
      }
    },

    destroyed() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
      if (this.reInitTimer) {
        clearTimeout(this.reInitTimer);
        this.reInitTimer = null;
      }
    },

    mounted() {
      this.ready = true;

      if (this.auto > 0) {
        this.timer = setInterval(() => {
          if (!this.dragging && !this.animating) {
            this.next();
          }
        }, this.auto);
      }

      this.reInitPages();

      var element = this.$el;

      // for mobile
      element.addEventListener('touchstart', this.dragStartEvent);
      element.addEventListener('touchmove', this.dragMoveEvent);
      element.addEventListener('touchend', this.dragEndEvent);
      // for pc
      element.addEventListener('mousedown', this.dragStartEvent);
      element.addEventListener('mousemove', this.dragMoveEvent);
      element.addEventListener('mouseup', this.dragEndEvent);
    }
  };
</script>
