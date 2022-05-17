<template>
  <div style="display: inline-block;position: relative;" ref="effect_button">
    <canvas id="canvas-confetti" ref="canvas-confetti" :width="canvasWidth" :height="canvasHeight"/>
    <transition mode="in-out" name="extrusion">
      <div style="overflow: hidden;" class="button" :class="'button-'+currentState" @click="handleClick">
        <transition-group name="fade" tag="main" class="content-wrapper">
          <div class="content-submit content" key="submit" v-if="currentState === 'submit'">
            <div>
              <slot>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:0ms;">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 12 12">
                    <line stroke="currentColor" x1="6" y1="1" x2="6" y2="11"/>
                    <polyline stroke="currentColor" points="1.5,6 6,11 10.5,6"/>
                  </svg>
                </span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:30ms;--y:6px;">C</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:60ms;--y:6px;">l</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:90ms;--y:6px;">i</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:120ms;--y:6px;">c</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:150ms;--y:6px;">k</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:180ms;--y:6px;">&nbsp;</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:210ms;--y:6px;">M</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:240ms;--y:6px;">e</span>
                <span v-if="submitTextVisible" :class="{jump}" style="--d:270ms;--y:6px;">!</span>
              </slot>
            </div>
          </div>
          <div class="content-loading content" key="loading" v-if="currentState === 'loading'">
            <div>
              <slot name="loading">
                <div class="dotList">
                  <span class="dot" style="--y: -6px;"/>
                  <span class="dot" style="--y: -6px;"/>
                  <span class="dot" style="--y: -6px;"/>
                </div>
              </slot>
            </div>
          </div>
          <div class="content-success content" key="success" v-if="currentState === 'success'">
            <div>
              <slot name="success">
                <svg :class="{successful:svg_success}" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 13 11">
                  <polyline stroke="currentColor" points="1.4,5.8 5.1,9.5 11.6,2.1 "/>
                </svg>
                <span>{{ successText }}</span>
              </slot>
            </div>
          </div>
          <div class="content-fail content" key="fail" v-if="currentState === 'fail'">
            <div>
              <slot name="fail">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 12 12">
                  <line stroke="currentColor" x1="11" y1="11" x2="1" y2="1"/>
                  <line stroke="currentColor" x1="11" y1="1" x2="1" y2="11"/>
                </svg>
                <span>{{ failText }}</span>
              </slot>
            </div>
          </div>
        </transition-group>
      </div>
    </transition>
  </div>
</template>

<script>

export default {
  name: "rx-button",
  props: {
    onClick: {
      type: Function,
      required: true
    },
    colors: {
      default: () => [
        {front: '#7b5cff', back: '#6245e0'}, // Purple
        {front: '#f9ff0c', back: '#ffc507'}, // Light Blue
        {front: '#5c86ff', back: '#345dd1'}  // Darker Blue
      ],
    },
    successText: {
      default: 'Success'
    },
    failText: {
      default: 'Fail'
    }
  },
  data() {
    return {
      currentState: 'submit',
      disable: false,
      jump: false,
      svg_success: false,

      animationId: '',
      intervalId: '',
      // "render" variables
      confettiCount: 30,
      sequinCount: 20,
      confetti: [],
      sequins: [],
      // "physics" variables
      gravityConfetti: 0.3,
      gravitySequins: 0.55,
      dragConfetti: 0.075,
      dragSequins: 0.02,
      terminalVelocity: 3,

      position: {
        buttonWidth: 0,
        buttonHeight: 0,
        screenWidth: 0,
        screenHeight: 0,
      },
      canvasWidth: 0,
      canvasHeight: 0,
    }
  },
  computed: {
    submitTextVisible() {
      return this.currentState === 'submit'
    }
  },
  methods: {
    handleClick() {
      if (!this.disable) {
        this.jump = true
        this.disable = true
        setTimeout(() => {
          this.$emit('loading')
          this.jump = false
          this.currentState = 'loading'

          let cb = this.onClick()
          const afterSuccess = () => {
            // 获得成功结果后，进入成功状态
            this.initBurst()
            this.currentState = 'success'
            this.$emit('success')
            setTimeout(() => {
              this.svg_success = true
            }, 0)
          }
          const afterFail = () => {
            // 获得失败结果后，进入失败状态
            this.currentState = 'fail'
            this.$emit('fail')
          }
          if (cb && cb.then) {
            cb.then(() => {
              afterSuccess()
            }, () => {
              afterFail()
            })
          } else {
            if (typeof cb === 'boolean') {
              cb ? afterSuccess() : afterFail()
            } else {
              // onClick 的返回值不是 boolean 或者 Promise 则报错
              throw '😱 Oh~ Attribute onClick needs return a Boolean or Promise object'
            }
          }
        }, 500)
      }
    },

    // helper function to pick a random number within a range
    randomRange(min, max) {
      return Math.random() * (max - min) + min
    },
    // add elements to arrays to be drawn
    initBurst() {
      for (let i = 0; i < this.confettiCount; i++) {
        this.confetti.push(this.createConfetti());
      }
      for (let i = 0; i < this.sequinCount; i++) {
        this.sequins.push(this.createSequin());
      }
    },
    initConfettoVelocity(xRange, yRange) {
      const randomRange = this.randomRange
      const x = randomRange(xRange[0], xRange[1]);
      const range = yRange[1] - yRange[0] + 1;
      let y = yRange[1] - Math.abs(randomRange(0, range) + randomRange(0, range) - range);
      if (y >= yRange[1] - 1) {
        // Occasional confetto goes higher than the max
        y += (Math.random() < .25) ? randomRange(1, 3) : 0;
      }
      return {x: x, y: -y};
    },
    createConfetti() {
      let randomRange = this.randomRange
      let that = this

      function Confetti() {
        this.randomModifier = randomRange(0, 99);
        this.color = that.colors[Math.floor(randomRange(0, that.colors.length))];
        this.dimensions = {
          x: randomRange(5, 9),
          y: randomRange(8, 15),
        };
        this.position = {
          x: randomRange(200, 280),
          y: randomRange(300, 340),
        };
        this.rotation = randomRange(0, 2 * Math.PI);
        this.scale = {
          x: 1,
          y: 1,
        };
        this.velocity = that.initConfettoVelocity([-9, 9], [6, 11]);
      }

      Confetti.prototype.update = function () {
        // apply forces to velocity
        this.velocity.x -= this.velocity.x * that.dragConfetti;
        this.velocity.y = Math.min(this.velocity.y + that.gravityConfetti, that.terminalVelocity);
        this.velocity.x += Math.random() > 0.5 ? Math.random() : -Math.random();
        // set position
        this.position.x += this.velocity.x;
        this.position.y += this.velocity.y;

        // spin confetto by scaling y and set the color, .09 just slows cosine frequency
        this.scale.y = Math.cos((this.position.y + this.randomModifier) * 0.09);
      }
      return new Confetti()
    },
    createSequin() {
      let randomRange = this.randomRange
      let that = this

      function Sequin() {
        this.color = that.colors[Math.floor(randomRange(0, that.colors.length))].back
        this.radius = randomRange(1, 2)
        this.position = {
          x: randomRange(190, 320),
          y: randomRange(300, 340),
        }
        this.velocity = {
          x: randomRange(-6, 6),
          y: randomRange(-10, -15)
        }
      }

      Sequin.prototype.update = function () {
        // apply forces to velocity
        this.velocity.x -= this.velocity.x * that.dragSequins;
        this.velocity.y = this.velocity.y + that.gravitySequins;

        // set position
        this.position.x += this.velocity.x;
        this.position.y += this.velocity.y;
      }

      return new Sequin()
    },

    render() {
      // const canvas = document.querySelector('#canvas-confetti')
      const canvas = this.$refs['canvas-confetti']
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      this.confetti.forEach(confetto => {
        let width = (confetto.dimensions.x * confetto.scale.x);
        let height = (confetto.dimensions.y * confetto.scale.y);

        // move canvas to position and rotate
        ctx.translate(confetto.position.x, confetto.position.y);
        ctx.rotate(confetto.rotation);

        // update confetto "physics" values
        confetto.update();

        // get front or back fill color
        ctx.fillStyle = confetto.scale.y > 0 ? confetto.color.front : confetto.color.back;

        // draw confetti
        ctx.fillRect(-width / 2, -height / 2, width, height);

        // reset transform matrix
        ctx.setTransform(1, 0, 0, 1, 0, 0);

        // clear rectangle where button cuts off
        if (confetto.velocity.y < 0) {
          // ctx.clearRect(canvas.width / 2 - 28 / 2, canvas.height / 2 + 10 / 2, 28, 10);
        }
      })
      this.sequins.forEach(sequin => {
        // move canvas to position
        ctx.translate(sequin.position.x, sequin.position.y);

        // update sequin "physics" values
        sequin.update();

        // set the color
        ctx.fillStyle = sequin.color;

        // draw sequin
        ctx.beginPath();
        ctx.arc(0, 0, sequin.radius, 0, 2 * Math.PI);
        ctx.fill();

        // reset transform matrix
        ctx.setTransform(1, 0, 0, 1, 0, 0);

        // clear rectangle where button cuts off
        if (sequin.velocity.y < 0) {
          // ctx.clearRect(canvas.width / 2 - 28 / 2, canvas.height / 2 + 10 / 2, 28, 10);
        }
      })

      // remove confetti and sequins that fall off the screen
      // must be done in seperate loops to avoid noticeable flickering
      this.confetti.forEach((confetto, index) => {
        if (confetto.position.y >= 550 + Math.random() * 50) this.confetti.splice(index, 1);
      });
      this.sequins.forEach((sequin, index) => {
        if (sequin.position.y >= 550 + Math.random() * 50) this.sequins.splice(index, 1);
      });

      this.animationId = window.requestAnimationFrame(this.render);
    },

    getElementPosition(el) {
      let position = {
        top: 0,
        left: 0
      }
      let actualTop = el.offsetTop
      let actualLeft = el.offsetLeft
      let currentEl = el.offsetParent
      while (currentEl !== null) {
        actualTop += currentEl.offsetTop
        actualLeft += currentEl.offsetLeft
        currentEl = currentEl.offsetParent
      }
      position.top = actualTop
      position.left = actualLeft
      return position
    },

    resetButton() {
      if (this.currentState === 'success' || this.currentState === 'fail') {
        this.currentState = 'submit'
        this.svg_success = false
        this.disable = false
      }
    },
    changeButtonState(stateName) {
      // 手动切换按钮状态
      if (this.currentState === 'loading') {
        return false
      }
      switch (stateName) {
        case 'submit':
          this.resetButton()
          break
        case 'success':
          this.resetButton()
          this.currentState = 'success'
          setTimeout(() => {
            this.svg_success = true
          }, 0)
          this.disable = true
          break
        case 'fail':
          this.resetButton()
          this.currentState = 'fail'
          this.disable = true
          break
        default:
          throw '😵 An unknown param was passed to function setButtonState'
      }
    }
  },
  mounted() {
    this.render()
    this.canvasWidth = this.$refs['canvas-confetti'].clientWidth
    this.canvasHeight = this.$refs['canvas-confetti'].clientHeight
    this.position.buttonWidth = this.getElementPosition(this.$refs['effect_button']).left
    this.position.buttonHeight = this.getElementPosition(this.$refs['effect_button']).top
    window.onresize = () => {
      return (() => {
        this.position.buttonWidth = this.getElementPosition(this.$refs['effect_button']).left
        this.position.buttonHeight = this.getElementPosition(this.$refs['effect_button']).top
      })()
    }
  },
  beforeDestroy() {
    cancelAnimationFrame(this.animationId)
    clearInterval(this.intervalId)
    window.onresize = () => {
    }
  }
}
</script>

<style scoped lang="scss">
* {
  box-sizing: border-box;
  text-align: left;
}

.fade-enter-active, .fade-leave-active {
  transition: all .6s;
}

.fade-enter {
  transform: translateY(-100%) !important;
  opacity: 0;
}

.fade-leave-to {
  transform: translateY(100%) !important;
  opacity: 0;
}

@keyframes jump {
  0% {
    transform: translateY(0px);
  }
  20% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(var(--y));
  }
  80% {
    transform: translateY(0px);
  }
  100% {
    transform: translateY(0px);
  }
}

.button {
  margin: 0 auto;
  height: 40px;
  border-radius: 200px;
  background: #1f2336;
  color: white;
  cursor: pointer;
  transition: width .3s;

  &.button-submit, &.button-success, &.button-fail {
    width: 170px;
  }

  &.button-loading {
    width: 130px;
  }

  .content-wrapper {
    height: 100%;
    position: relative;

    .content {
      //border: 1px solid red;
      width: 100%;
      height: 100%;
      position: absolute;
    }
  }
}

.content-submit {
  @keyframes fall {
    0% {
      opacity: 1;
      transform: translateY(0)
    }
    100% {
      opacity: 0;
      transform: translateY(100%)
    }
  }

  > div {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    span.jump {
      animation: fall 1 900ms var(--d);
    }

    span:first-child {
      display: flex;
      justify-content: center;
      align-items: center;
    }

    svg {
      color: #5c86ff;
      width: 14px;
      display: inline-block;
      fill: none;
      margin-right: 5px;
      stroke-linecap: round;
      stroke-linejoin: round;
      stroke-width: 2;
    }
  }

}

.content-loading {
  > div {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    .dotList {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 26%;
      height: 100%;
      margin: 0 auto;

      .dot {
        background: white;
        width: 6px;
        height: 6px;
        border-radius: 100px;
        animation: jump 1000ms infinite ease-in-out;

        &:nth-child(2) {
          animation-delay: 100ms;
        }

        &:last-child {
          animation-delay: 200ms;
        }
      }
    }
  }
}

.content-success {
  > div {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    svg {
      color: #5cffa1;
      stroke-dasharray: 20;
      stroke-dashoffset: 20;
      transition: stroke-dashoffset .3s ease-in-out;
      width: 14px;
      display: inline-block;
      fill: none;
      margin-right: 5px;
      stroke-linecap: round;
      stroke-linejoin: round;
      stroke-width: 2;
    }

    svg.successful {
      stroke-dashoffset: 0;
      transition: stroke-dashoffset .3s ease-in-out .5s;
      opacity: 1;
    }
  }

}

.content-fail {
  @keyframes shake {
    0% {
      transform: translateX(0);
    }
    45% {
      transform: translateX(1%);
    }
    55% {
      transform: translateX(-1%);
    }
    100% {
      transform: translateX(0%);
    }
  }

  > div {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    animation: shake 0.2s 1 0.5s;

    svg {
      color: #cb4110;
      transition: stroke-dashoffset .3s ease-in-out;
      width: 14px;
      display: inline-block;
      fill: none;
      margin-right: 5px;
      stroke-linecap: round;
      stroke-linejoin: round;
      stroke-width: 2;
    }
  }

}

#canvas-confetti {
  height: 620px;
  width: 500px;
  position: absolute;
  z-index: 10;
  pointer-events: none;
  top: -300px;
  left: calc(-50% - 85px);
}
</style>
