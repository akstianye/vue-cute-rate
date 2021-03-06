<template>
  <div ref="cuteRate"
       :class="starDisable"
       class="star-main"
       @mouseleave="starMouseleave">
    <input :value="currentValue" type="hidden">
    <div v-for="i in count" :key="i"
         class="star-full"
         @mouseover="starMousemove(i)"
         @click="starClick(i)">
      <div v-if="customCharSlot">
        <slot v-if="i <= currentIndex()"
              name="customChar"
              computeClass="char" />
        <span v-else class="char">{{ inactiveChar }}</span>
      </div>
      <div v-else>
        <i v-if="!rateCharSlot"
           :class="computeFullTypeCls(i)"
           class="char iconfont icon-rate-1" />
        <slot :computeClass="computeFullCls(i, 'char')"
              name="rateChar" />
        <span class="star-half"
              @mouseover.stop="starHalfMousemove(i)">
          <i v-if="!rateCharSlot"
             :class="computeHalfTypeCls(i)"
             class="char iconfont icon-rate-2" />
          <slot :computeClass="computeHalfCls(i, 'char')"
                name="rateChar" />
        </span>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'StarRate',

    props: {
      count: {
        type: Number,
        default: 5
      },
      value: {
        type: Number,
        default: 0
      },
      starHalf: {
        type: Boolean,
        default: false
      },
      disabled: {
        type: Boolean,
        default: false
      },
      disabledCursor: {
        type: Boolean,
        default: false
      },
      type: {
        type: String,
        default: 'star'
      },
      allowClear: {
        type: Boolean,
        default: true
      },
      fontSize: {
        type: String,
        default: '22px'
      },
      activeColor: {
        type: String,
        default: '#fadb14'
      },
      inactiveColor: {
        type: String,
        default: '#e9e9e9'
      },
      hoverColor: {
        type: String,
        default: ''
      },
      inactiveChar: {
        type: String,
        default: '-'
      },
      hoverChange: {
        type: Boolean,
        default: false
      },
      onHoverChange: {
        type: Function,
        default: () => {}
      },
      onChange: {
        type: Function,
        default: () => {}
      },
      stopAndNotice: {
        type: Boolean,
        default: false
      }
    },

    data () {
      return {
        hoverIndex: -1,
        currentValue: this.value,
        isHalf: this.starHalf && this.value.toString().split('.').length > 1,
        rateCharSlot: false,
        customCharSlot: false,
        stopAndNoticeLocal: this.stopAndNotice
      }
    },
    computed: {
      starDisable () {
        return this.disabled && this.disabledCursor
          ? 'star-disable'
          : this.disabled && !this.disabledCursor
            ? 'star-disable-default'
            : 'star-able'
      }
    },

    watch:{
      value(val){
        this.currentValue = val;
      },
      stopAndNotice(val){
        this.stopAndNoticeLocal = val;
      }
    },

    mounted () {
      const style = this.$refs.cuteRate.style
      style.setProperty('--active-color', this.activeColor)
      style.setProperty('--inactive-color', this.inactiveColor)
      style.setProperty('--hover-color', this.hoverColor || this.activeColor)
      style.setProperty('--font-size', this.fontSize)
      this.rateCharSlot = this.$scopedSlots.rateChar
      this.customCharSlot = this.$scopedSlots.customChar
    },

    methods: {
      computeFullCls (i, j) {
        let starHalf = this.starHalf
        let currentIndex = this.currentIndex()
        const temp = starHalf
          ? i < currentIndex || (i === currentIndex && !this.isHalf)
          : i <= Math.ceil(currentIndex)
        return {
          'icon-full': temp,
          'char': j === 'char'
        }
      },
      computeFullTypeCls (i) {
        return Object.assign({}, this.computeFullCls(i), this.computeType())
      },
      computeHalfCls (i, j) {
        if (!this.starHalf && j === 'char') return 'icon-rate-2 char'
        let currentIndex = this.currentIndex()
        return {
          'icon-half': i === Math.ceil(currentIndex) && this.isHalf,
          'icon-rate-2 char': j === 'char'
        }
      },
      computeHalfTypeCls (i) {
        return Object.assign({}, this.computeHalfCls(i), this.computeType())
      },
      computeType () {
        return {
          'icon-star': this.type === 'star',
          'icon-star1': this.type === 'star1',
          'icon-heart': this.type === 'heart'
        }
      },
      currentIndex () {
        return this.hoverIndex === -1 ? this.currentValue : this.hoverIndex
      },
      starMousemove (i) {
        if (this.disabled) return
        if (this.stopAndNoticeLocal) {
          this.$emit('on-stop-and-notice')
          return
        }
        if (this.starHalf) {
          this.isHalf = false
        }
        this.hoverIndex = i
        this.changeValue(i)
      },
      starHalfMousemove (i) {
        if (this.disabled) return
        if (this.stopAndNoticeLocal) {
          this.$emit('on-stop-and-notice')
          return
        }
        if (this.starHalf) {
          this.isHalf = true
        }
        this.hoverIndex = i
        this.changeValue(i)
      },
      changeValue (i) {
        if (this.hoverChange) {
          let value = this.isHalf ? i - 0.5 : i
          this.$emit('input', value)
          this.onHoverChange(value)
        } else {
          this.onHoverChange(this.value)
        }
      },
      starClick (i) {
        if (this.disabled) return
        if (this.stopAndNoticeLocal) {
          this.$emit('on-stop-and-notice')
          return
        }
        const curValue = this.currentValue
        let value = this.isHalf ? i - 0.5 : i
        this.currentValue = value
        if (curValue === this.currentValue && this.allowClear) {
          this.currentValue = 0
          this.hoverIndex = -1
          value = 0
        }
        this.$emit('input', value)
        this.onChange(value)
      },
      starMouseleave () {
        if (this.disabled) return
        if (this.stopAndNoticeLocal) {
          this.$emit('on-stop-and-notice')
          return
        }
        this.hoverIndex = -1
        this.isHalf = this.starHalf && this.currentValue.toString().split('.').length > 1
        if (!this.hoverChange) return
        this.$emit('input', this.currentValue)
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "assets/iconfont.css";

  * {
    margin: 0;
    padding: 0;
  }

  .star-main {
    margin: 10px auto;
    padding: 0;
    text-align: center;
    display: inline-block;
  }
  .char {
    color: var(--inactive-color);
    font-size: var(--font-size);
    transition: all 0.3s ease-in-out;
  }
  .star-disable {
    .star-full {
      cursor: not-allowed;
      &:hover {
        transform: scale(1);
      }
    }
  }
  .star-disable-default {
    .star-full {
      cursor: default;
      &:hover {
        transform: scale(1);
      }
    }
  }
  .star-full {
    position: relative;
    display: inline-block;
    transition: all 0.2s ease;
    margin-right: 8px;
    cursor: pointer;
  }
  .star-able {
    .star-full {
      &:hover {
        transform: scale(1.1);
        .icon-full, .icon-half {
          color: var(--hover-color)
        }
      }
    }
  }
  .star-half {
    position: absolute;
    left: 0;
    top: 0;
    width: 50%;
    height: 100%;
    overflow: hidden;
  }
  .icon-rate-2 {
    color: transparent;
  }
  .icon-full, .icon-half {
    color: var(--active-color);
  }
  .star-char {
    padding: 5px;
  }
</style>
