<template>
  <component :value="date" :placeholder="placeholder"
    :inputClass="inputClass" :alignment="alignment"
    :is="wrap ? 'WrapperInput' : 'SingleInput'">
    <slot></slot>
  </component>
</template>

<script>
import BasicInput from './BasicInput'
import SingleInput from './SingleInput'
import WrapperInput from './WrapperInput'
import Flatpickr from 'flatpickr'

export default {
  mixins: [BasicInput],

  props: {
    alignment: String,
    config: {
      type: Object,
      default: () => ({})
    }
  },

  components: {
    SingleInput,
    WrapperInput
  },

  data () {
    return {
      datepicker: null,
      selectedDates: null
    }
  },

  mounted () {
    if (!this.datepicker) {
      this.config.onValueUpdate = this.dateUpdated
      this.datepicker = new Flatpickr(this.$el, this.config)
      this.initBlur()
      this.setDate(this.value)
    }
    this.$watch('config', this.redraw)
    this.$watch('value', this.setDate)
  },

  beforeDestroy () {
    if (this.datepicker && !this.static) {
      this.datepicker.destroy()
      this.datepicker = null
    }
  },

  methods: {
    redraw (newConfig) {
      this.datepicker.config = Object.assign(this.datepicker.config, newConfig)
      this.datepicker.redraw()
      this.datepicker.jumpToDate()
    },
    setDate (newDate) {
      if (newDate) {
        this.datepicker.setDate(newDate)
      } else {
        this.selectedDates = null
        this.datepicker.clear()
      }
    },
    dateUpdated (selectedDates, dateStr) {
        this.date = dateStr
    },
    initBlur () {
      let dp = this.datepicker
      this.datepicker._input.addEventListener("keydown", function(event) {
        if (!isNaN(event.key) && dp._input.value.length > 0 && !dp._input.value.includes(',')) {
          dp._input.value = 'Mo, ' + dp._input.value
        }
      }, true);
      let self = this
      this.datepicker._input.addEventListener("blur", function(event) {
        let enter = new KeyboardEvent('keydown', {
          bubbles : true,
          cancelable : true,
          key : "Enter"
        })
        if (dp._input.value.length > 0 && !dp._input.value.includes(',')) {
          dp._input.value = 'Mo, ' + dp._input.value
        }
        if (dp._input.value.length == 0) {
          self.selectedDates = null
          self.$emit('input', null)
          self.datepicker.clear()
        }
        dp._input.dispatchEvent(enter)
      }, true);
    }
  },

  computed: {
    wrap () {
      return !!this.config.wrap
    },
    static () {
      return !!this.config.static
    },
    name () {
      return this.wrap ? 'wrapperInput' : 'singleInput'
    },
    date: {
      get() {
        return this.selectedDates || this.value
      },
      set(newValue) {
        if (this.selectedDates !== newValue) {
          this.selectedDates = newValue
          this.$emit('input', newValue)
        }
      }
    }
  }
}
</script>

<style lang="stylus">
$calendar_background = #ffffff
$calendar_border_color = #d3d6db

$months_color = #111
$months_background = transparent

$weekdays_background = transparent

$day_text_color = #222324
$day_hover_background_color = #d3d6db

$today_color = #ed6c63
$selected_day_background = #1fc8db

@import '~flatpickr/src/style/flatpickr'

.flatpickr-calendar.hasWeeks
  width: auto !important
</style>
