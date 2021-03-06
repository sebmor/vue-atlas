<template>
  <div :class="classObj" :style="styleObj">
    <div :class="`${prefixCls}-topbar-inner`">
      <div :class="`${prefixCls}-topbar-left`">
        <slot name="left"/>
      </div>
      <div :class="`${prefixCls}-topbar-center`">
        <slot name="center"/>
      </div>
      <div :class="`${prefixCls}-topbar-right`">
        <slot name="right"/>
      </div>
    </div>
  </div>
</template>

<script>
import events from '../utils/events'

export default {
  name: 'VaTopbar',
  mixins: [events],
  props: {
    /**
    * The theme validator is being removed to allow you to
    * create and use your own themes.
    */
    theme: {
      type: String,
      default: 'blue',
      required: false
      // validator (v) {
      //   return [
      //     'default',
      //     'white',
      //     'blue',
      //     'dark',
      //     'darker',
      //     'purple'
      //   ]
      // }
    },
    prefixCls: {
      type: String,
      default: 'va'
    }
  },
  data () {
    let t = this.theme
    return {
      currentDesktopMinimumWidth: 0,
      currentDesktopMargin: 0,
      currentContentWidth: 0,
      currentTopbarHeight: 0,
      currentMinibarWidth: 0,
      currentSidebarWidth: 0,
      sidebarPriority: false,
      minibarPriority: false,
      currentWindowWidth: 0,
      topbarPriority: false,
      topbarPadded: false,
      isReverse: false,
      isMobile: false,
      isSplit: false,
      isRTL: false,
      th: t
    }
  },
  mounted () {
    /**
     * This needs to be wrapped in a short setTimeout to give
     * LayoutManager time to call enableReceivers().
     */
    setTimeout(() => {
      this.dispatch('VaLayoutManager', 'Va@configTopbarThemeChange', this.th)
    }, 10)
  },
  created () {
    this.$on('Va@topbarPresenceCheck', (val) => { this.dispatch('VaLayoutManager', 'Va@topbarPresenceReply', true) })
    this.$on('Va@desktopMinimumWidthChange', (val) => { this.currentDesktopMinimumWidth = val })
    this.$on('Va@desktopMarginChange', (val) => { this.currentDesktopMargin = val })
    this.$on('Va@minibarWidthChange', (val) => { this.currentMinibarWidth = val })
    this.$on('Va@topbarHeightChange', (val) => { this.currentTopbarHeight = val })
    this.$on('Va@contentWidthChange', (val) => { this.currentContentWidth = val })
    this.$on('Va@sidebarWidthChange', (val) => { this.currentSidebarWidth = val })
    this.$on('Va@sidebarPriorityChange', (val) => { this.sidebarPriority = val })
    this.$on('Va@minibarPriorityChange', (val) => { this.minibarPriority = val })
    this.$on('Va@windowWidthChange', (val) => { this.currentWindowWidth = val })
    this.$on('Va@topbarPriorityChange', (val) => { this.topbarPriority = val })
    this.$on('Va@topbarPaddedChange', (val) => { this.topbarPadded = val })
    this.$on('Va@reverseChange', (val) => { this.isReverse = val })
    this.$on('Va@topbarIsMobile', (val) => { this.isMobile = val })
    this.$on('Va@topbarThemeChange', (val) => { this.th = val })
    this.$on('Va@splitChange', (val) => { this.isSplit = val })
    this.$on('Va@rtlChange', (val) => { this.isRTL = val })
  },
  beforeDestroy () {
    this.dispatch('VaLayoutManager', 'Va@topbarDisconnect', true)
  },
  watch: {
    theme (val) {
      this.th = val
      this.dispatch('VaLayoutManager', 'Va@configTopbarThemeChange', this.th)
    }
  },
  computed: {
    classObj () {
      let {prefixCls, th} = this
      let klass = {}

      klass[prefixCls + '-topbar'] = true
      klass[prefixCls + '-topbar--theme-' + th] = true

      return klass
    },
    styleObj () {
      let dmw = parseInt(this.currentDesktopMinimumWidth)
      let dm = parseInt(this.currentDesktopMargin)
      let th = parseInt(this.currentTopbarHeight)
      let sw = parseInt(this.currentSidebarWidth)
      let cw = parseInt(this.currentContentWidth)
      let mw = parseInt(this.currentMinibarWidth)
      let sp = this.sidebarPriority
      let mp = this.minibarPriority
      let tp = this.topbarPriority
      let tpad = this.topbarPadded
      let reverse = this.isReverse
      let mobile = this.isMobile
      let split = this.isSplit
      let rtl = this.isRTL
      let style = {}

      style['height'] = th + 'px'
      style['left'] = '0px'
      style['right'] = '0px'

      /**
       * Adjust the margins if content width is smaller than
       * desktop minimum width.
       */
      if (!mobile) {
        if (cw < dmw) {
          let x = dmw - cw
          dm = dm - (x / 2)
        }
      } else {
        dm = 0
        dmw = 0
      }

      /**
       * If a minimum desktop width is set
       */
      if (dmw !== 0) {
        style['min-width'] = (dmw - mw - sw) + 'px'
      } else {
        style['min-width'] = '0px'
      }

      /**
       * Ready to be thoroughly confused?
       */
      if (sp) {
        if (split) {
          if (reverse) {
            if (mp) {
              style['left'] = (dm + sw) + 'px'
              style['right'] = (dm + mw) + 'px'
            } else {
              style['left'] = (dm + sw) + 'px'
              style['right'] = dm + 'px'
            }
          } else {
            if (mp) {
              style['left'] = (dm + mw) + 'px'
              style['right'] = (dm + sw) + 'px'
            } else {
              style['left'] = dm + 'px'
              style['right'] = (dm + sw) + 'px'
            }
          }
        } else {
          if (reverse) {
            if (mp) {
              style['left'] = (sw + mw + dm) + 'px'
              style['right'] = dm + 'px'
            } else {
              style['left'] = (dm + sw) + 'px'
              style['right'] = dm + 'px'
            }
          } else {
            if (mp) {
              style['left'] = (sw + mw + dm) + 'px'
              style['right'] = dm + 'px'
            } else {
              style['left'] = (sw + mw + dm) + 'px'
              style['right'] = dm + 'px'
            }
          }
        }
      }

      if (!sp) {
        if (split) {
          if (reverse) {
            if (mp) {
              style['left'] = dm + 'px'
              style['right'] = (dm + mw) + 'px'
            } else {
              style['left'] = dm + 'px'
              style['right'] = dm + 'px'
            }
          } else {
            if (mp) {
              style['left'] = (dm + mw) + 'px'
              style['right'] = dm + 'px'
            } else {
              style['left'] = dm + 'px'
              style['right'] = dm + 'px'
            }
          }
        } else {
          if (reverse) {
            if (mp) {
              style['left'] = (sw + mw + dm) + 'px'
              style['right'] = dm + 'px'
            } else {
              style['left'] = dm + 'px'
              style['right'] = dm + 'px'
            }
          } else {
            if (mp) {
              style['left'] = (dm + mw) + 'px'
              style['right'] = dm + 'px'
            } else {
              style['left'] = dm + 'px'
              style['right'] = dm + 'px'
            }
          }
        }
      }

      if (rtl) {
        // rtl only effective when not split
        if (!split) {
          if (reverse) {
            if (sp) {
              style['left'] = dm + 'px'
              style['right'] = (sw + mw + dm) + 'px'
            } else {
              if (mp) {
                style['left'] = dm + 'px'
                style['right'] = (dm + mw) + 'px'
              }
            }
          } else {
            if (sp) {
              if (mp) {
                style['right'] = (mw + sw + dm) + 'px'
                style['left'] = dm + 'px'
              } else {
                style['right'] = (dm + sw) + 'px'
                style['left'] = dm + 'px'
              }
            } else {
              if (mp) {
                style['right'] = (mw + sw + dm) + 'px'
                style['left'] = dm + 'px'
              }
            }
          }
        }
      }

      if (tp) {
        style['left'] = '0px'
        style['right'] = '0px'

        if (tpad) {
          style['padding-left'] = (dm + 8) + 'px'
          style['padding-right'] = (dm + 8) + 'px'
          /**
           * Why 8px?
           * Because a 50px Minibar looks best, and if the
           * first or last element in a padded topbar is
           * an icon, this lines it up nicely with the
           * icons in the Minibar.
           */
        }
      }

      // Less than 40 and we don't want overflow.
      // Greater than 40 and we do, because we don't want
      // to hide dropdown menus.
      // That means..
      // Dropdown menus in the topbar won't appear when the
      // topbar has a height of less than 40px.
      if (th < 40) {
        style['overflow'] = 'hidden'
      } else {
        style['overflow'] = 'visible'
      }

      return style
    }
  }
}
</script>
