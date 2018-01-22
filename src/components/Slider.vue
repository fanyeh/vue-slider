<template lang="pug">
  .container
    .slide-button(@click="right" class="slide-button--right")
      i(class="fa fa-chevron-right fa-2x" aria-hidden="true")
    .slide-button(@click="left" class="slide-button--left")
      i(class="fa fa-chevron-left fa-2x" aria-hidden="true")
    .showcase(
      ref="showcase"
      v-on:mouseout="hideShowcase"
      :class="{expand:expandShowcase}"
      )
    .slider-wrapper(ref="wrapper")
      transition-group(tag="div" class="slider" name="list")
        .slide-container(
          v-for="(container,slideContainerIndex) in slideContainer"
          :key="container"
          :class="[slideContainerIndex%3 === 1 ? 'middle' : '']"
          v-on:transitionend="containerTransition"
          )
          .slide(
            v-for="(content ,contentIndex) in contentContainer[container]"
            ref="slides"
            v-mouse:mouseover="{position: slideContainerIndex % 3,handler:selectSlide}"
            v-mouse:mouseout="{position: slideContainerIndex % 3,handler:unselectSlide}"
            :id="'slide-'+container+'-'+contentIndex"
            :data-container-index="container"
            :data-content-index="contentIndex"
          )
</template>


<script>
import _ from 'lodash';

export default {
  name: 'Slider',
  directives: {
    mouse: {
      bind(el, binding) {
        if (binding.value.position === 1) {
          el.addEventListener(binding.arg, binding.value.handler);
        }
      },
      update(el, binding) {
        if (binding.value.position === 1) {
          el.addEventListener(binding.arg, binding.value.handler);
        } else {
          el.removeEventListener(binding.arg, binding.value.handler);
        }
      },
    },
  },
  data() {
    return {
      defaultSize: 200,
      bodyMarginLeft: document.body.getBoundingClientRect().left,
      expandShowcase: false,
      timeoutID: '',
      ratioX: 1.6,
      ratioY: 1.6,
      ratioOffset: 0,
      selectedSlidePos: {},
      isSliding: false,
      slideContainer: [0, 1, 2],
      contentContainer: [],
      contentContainerSize: 6,
      contentData:
      [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24],
    };
  },
  created() {
    this.setContentContainer();
  },
  methods: {
    updateContentContainer() {
      this.contentContainer = _.chunk(this.contentData, this.contentContainerSize);
      this.slideContainer = [0, 1, 2];
      this.setColor(this.slideContainer[0]);
      this.setColor(this.slideContainer[1]);
      this.setColor(this.slideContainer[2]);
    },
    left: _.debounce(function slideLeft() {
      if (!this.expandShowcase) {
        this.isSliding = true;
        if (this.slideContainer[1] !== 0) {
          this.slideContainer.unshift(this.slideContainer[0] - 1);
          this.slideContainer.pop();
          this.setColor(this.slideContainer[0]);
        }
      }
    }, 500),
    right: _.debounce(function slideRight() {
      if (!this.expandShowcase) {
        this.isSliding = true;
        const containerID = _.last(this.slideContainer);
        if (containerID < this.contentContainer.length) {
          this.slideContainer.push(containerID + 1);
          this.setColor(_.last(this.slideContainer));
          this.slideContainer.shift();
        }
      }
    }, 500),
    selectSlide(event) {
      this.timeoutID = setTimeout(() => {
        if (!this.isSliding) {
          const selectedSlide = event.target;
          this.selectedSlidePos = this.slideIsFirstOrLast(selectedSlide);
          const transitionDistance = this.transitionDistance(selectedSlide);
          this.setShowcase(selectedSlide);
          const animationCallback = (currentSlide) => {
            const currentContainer = this.containerIndex(currentSlide);
            let direction = 0;
            if (this.selectedSlidePos.isFirst) {
              if (currentContainer >= 1) {
                direction = 1;
              }
            } else if (this.selectedSlidePos.isLast) {
              if (currentContainer <= 1) {
                direction = -1;
              }
            } else if (currentContainer === 1) {
              direction =
              this.contentIndex(currentSlide) < this.contentIndex(selectedSlide) ? 1 : -1;
            } else {
              direction = currentContainer < 1 ? 1 : -1;
            }
            this.setStyleProperty(currentSlide, { transform: `translateX(${transitionDistance * direction}px)` });
          };
          this.animateSlideTransition(animationCallback);
        }
      }, 500);
    },
    unselectSlide() {
      clearTimeout(this.timeoutID);
    },
    containerIndex(element) {
      return element.dataset.containerIndex * 1;
    },
    contentIndex(element) {
      return element.dataset.contentIndex * 1;
    },
    slideIsFirstOrLast(element) {
      return {
        isFirst: this.slideIsFirst(element),
        isLast: this.slideIsLast(element),
      };
    },
    slideIsFirst(element) {
      return this.contentIndex(element) === 0;
    },
    slideIsLast(element) {
      const containerIndex = this.containerIndex(element);
      return this.contentIndex(element) === this.contentContainer[containerIndex].length - 1;
    },
    transitionDistance(element) {
      if (this.selectedSlidePos.isFirst || this.selectedSlidePos.isLast) {
        return element.clientWidth;
      }
      return element.clientWidth * this.ratioOffset;
    },
    setShowcase(selectedSlide) {
      const selectedRect = selectedSlide.getBoundingClientRect();
      const showcaseWidth = selectedRect.left - this.bodyMarginLeft;
      const showcaseStyle = {
        left: `${showcaseWidth}px`,
        width: `${selectedRect.width}px`,
        height: `${selectedRect.height}px`,
        'background-color': `${selectedSlide.style.backgroundColor}`,
      };
      let transformOrigin = 'center center';
      if (this.selectedSlidePos.isFirst) {
        transformOrigin = 'center left';
      } else if (this.selectedSlidePos.isLast) {
        transformOrigin = 'center right';
      }
      Object.assign(showcaseStyle, { 'transform-origin': transformOrigin });
      this.setStyleProperty(this.$refs.showcase, showcaseStyle);
      this.expandShowcase = true;
    },
    hideShowcase(event) {
      if (event.currentTarget.classList.contains('expand')) {
        this.expandShowcase = false;
        this.animateSlideTransition((currentSlide) => {
          this.setStyleProperty(currentSlide, { transform: '' });
        });
      }
    },
    animateSlideTransition(callback) {
      this.$refs.slides.forEach((slide) => {
        callback(slide);
      });
    },
    setColor(containerIndex, callback) {
      if (containerIndex > -1 && containerIndex < this.contentContainer.length) {
        this.$nextTick(() => {
          this.contentContainer[containerIndex].forEach((content, contentIndex) => {
            const slideID = `#slide-${containerIndex}-${contentIndex}`;
            const slide = this.$el.querySelector(slideID);
            const offset = contentIndex * 7;
            const hue = (containerIndex * 20) % 360;
            this.setStyleProperty(slide, { 'background-color': `hsl(${hue},${40 + offset}%,${50 + offset}%)` });
          });
          if (callback) {
            callback();
          }
        });
      }
    },
    setStyleProperty(element, styles) {
      Object.assign(element.style, styles);
    },
    containerTransition() {
      this.isSliding = false;
    },
    resetSize() {
      this.setContentContainer();
    },
    setContentContainer() {
      if (window.matchMedia('(max-width: 480px)').matches) {
        this.contentContainerSize = 2;
      } else if (window.matchMedia('(max-width: 768px)').matches) {
        this.contentContainerSize = 3;
      } else if (window.matchMedia('(max-width:1024px)').matches) {
        this.contentContainerSize = 4;
      } else {
        this.contentContainerSize = 6;
      }
      this.updateContentContainer();
    },
  },
  mounted() {
    this.$el.style.setProperty('--blockSize', `${this.defaultSize}px`);
    this.$el.style.setProperty('--ratio-x', `${this.ratioX}`);
    this.$el.style.setProperty('--ratio-y', `${this.ratioY}`);
    this.slideContainer.forEach((container, index) => {
      this.setColor(index);
    });
    this.ratioOffset = (this.ratioX - 1) / -2;
    window.addEventListener('resize', _.debounce(this.resetSize, 150));
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

/* Medium Devices, Desktops */
@mixin md-screen {
  @media only screen and (max-width : 1024px) {
    @content;
  }
}

/* Small Devices, Tablets */
@mixin sm-screen {
  @media only screen and (max-width : 768px) {
    @content;
  }
}

/* Extra small devices */
@mixin xs-screen {
  @media only screen and (max-width : 480px) {
    @content;
  }
}

@function slide-width($numOfSlides) {
  @return (90vw - $numOfSlides * 0.1vw) / $numOfSlides;
}

@function slide-height($width) {
  @return $width * 0.6;
}

.container {
  --duration: 0.4s;
  --cubic-bezier: cubic-bezier( 0.5 , 0, 0.1 ,1);
  position: relative;
}

.container > * {
  box-sizing: border-box;
}

.slide-button {
  width: 5vw;
  height: slide-height(slide-width(6));
  background-color: black;
  opacity: 0.6;
  position: absolute;
  z-index: 10;
}

.slide-button.slide-button--left {
  left: 0;
}
.slide-button.slide-button--right {
  right: 0;
}

.slider-wrapper {
  overflow: hidden;
}

.slider {
  display: flex;
  width: 270vw;
  transform: translateX(-85vw);
}

.slide-container {
  display: flex;
  flex: 0 0 calc(90vw);
  will-change: transform;
}

.slide-container.middle {
  z-index: 1;
}

.slide {
  width:slide-width(6);
  height: slide-height(slide-width(6));
  transition: transform var(--duration) var(--cubic-bezier);
  will-change: transform;
  margin: auto;
}

@include md-screen {
  .slide {
    width:slide-width(4);
    height: slide-height(slide-width(4));
  }
  .slide-button {
    height:slide-height(slide-width(4));
  }
}

@include sm-screen {
  .slide {
    width:slide-width(3);
    height: slide-height(slide-width(3));
  }
  .slide-button {
    height:slide-height(slide-width(3));
  }
}

@include xs-screen {
  .slide {
    width:slide-width(2);
    height: slide-height(slide-width(2));
  }
  .slide-button {
    height:slide-height(slide-width(2));
  }
}

.showcase {
  position: absolute;
  visibility: hidden;
  transition: transform var(--duration) var(--cubic-bezier), visibility 0s calc(var(--duration));
  will-change: transform, opacity;
}

.expand {
  transform: scale( var(--ratio-x) , var(--ratio-y));
  z-index: 10;
  visibility: visible;
  transition: transform var(--duration) var(--cubic-bezier) ;
}

.left {
  transform-origin:center left;
}

// .list-enter-active,
// .list-leave-active {
//   transition: all 1s;
// }

.list-enter,
.list-leave-to {
  opacity: 0;
}

.list-leave-active{
  display: none;
  position: absolute;
}
.list-move {
  transition: all 1s;
}

.fa {
  color:grey;
  position: absolute;
  top:50%;
  left:50%;
  transform:translate(-50%,-50%);
}
</style>
