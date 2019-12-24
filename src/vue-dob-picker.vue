<template>
  <div class="vue-dob-picker">
    <label :class="labelClass">
      <div v-if="showLabels !== 'false'">{{ labels[0] }}</div>
      <select v-model="day" :class="dayClass" @blur="onBlur" @focus="onFocus">
        <option v-if="placeholders[0]" value="null" disabled="disabled">{{ placeholders[0] }}</option>
        <option v-for="(item, index) in new Array(28)" :key="index" :value="index + 1">{{ index + 1 }}</option>
        <option value="29" v-if="daysInMonth >= 29 || isLeapYear">29</option>
        <option value="30" v-if="daysInMonth >= 30">30</option>
        <option value="31" v-if="daysInMonth >= 31">31</option>
      </select>
    </label>
    <label :class="labelClass">
      <div v-if="showLabels !== 'false'">{{ labels[1] }}</div>
      <select v-model="month" :class="monthClass" @blur="onBlur" @focus="onFocus">
        <option v-if="placeholders[1]" value="null" disabled="disabled">{{ placeholders[1] }}</option>
        <option v-for="(item, index) in new Array(12)" :key="index" :value="index">{{ getDisplayedMonth(index) }}</option>
      </select>
    </label>
    <label :class="labelClass">
      <div v-if="showLabels !== 'false'">{{ labels[2] }}</div>
      <select v-model="year" :class="yearClass" @blur="onBlur" @focus="onFocus">
        <option v-if="placeholders[2]" value="null" disabled="disabled">{{ placeholders[2] }}</option>
        <option v-for="(item, index) in new Array(100)" :key="index" :value="currentYear - index">{{ currentYear - index }}</option>
      </select>
    </label>
  </div>
</template>

<script>
const datesInMonths = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];

export default {
  name: 'vue-dob-picker',
  props: {
    value: {
      type: Date,
    },
    selectClass: String,
    selectPlaceholderClass: String,
    labelClass: String,
    showLabels: String,
    locale: {
      type: String,
      default: navigator.language,
    },
    monthFormat: {
      type: String,
      default: '2-digit',
    },
    labels: {
      type: Array,
      default: () => ['Date', 'Month', 'Year'],
    },
    placeholders: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      day: null,
      month: null,
      year: null,
      currentYear: (new Date()).getUTCFullYear(),
      blurTimeout: null,
    };
  },
  computed: {
    date: {
      get() {
        let daysInMonth = this.daysInMonth;
        if (this.isLeapYear && this.month === 1) {
          daysInMonth += 1;
        }
        if (this.day > daysInMonth) {
          this.day = null;
        }
        if (this.day !== null && this.month !== null && this.year !== null) {
          return new Date(this.year, this.month, this.day, 0, 0, 0, 0);
        }
        return null;
      },
      set(val) {
        if (val) {
          this.day = val.getUTCDate();
          this.month = val.getUTCMonth();
          this.year = val.getUTCFullYear();
        }
      },
    },
    daysInMonth() {
      return datesInMonths[this.month] || 31;
    },
    isLeapYear() {
      return ((this.year % 4 === 0) && (this.year % 100 !== 0)) || (this.year % 400 === 0);
    },
    dayClass() {
      if (this.selectPlaceholderClass && this.day === null) {
        return this.selectPlaceholderClass;
      }
      return this.selectClass;
    },
    monthClass() {
      if (this.selectPlaceholderClass && this.month === null) {
        return this.selectPlaceholderClass;
      }
      return this.selectClass;
    },
    yearClass() {
      if (this.selectPlaceholderClass && this.year === null) {
        return this.selectPlaceholderClass;
      }
      return this.selectClass;
    },
  },
  watch: {
    date() {
      if (this.date) {
        this.$emit('input', this.date);
      }
    },
  },
  methods: {
    getDisplayedMonth(monthNum) {
      const monthDateObj = new Date(2000, monthNum, 1);
      const locale = this.locale || navigator.language;
      return monthDateObj.toLocaleString(locale, { month: this.monthFormat });
    },
    onBlur() {
      this.blurTimeout = window.setTimeout(() => {
        this.$emit('blur');
      }, 50);
    },
    onFocus() {
      window.clearTimeout(this.blurTimeout);
    },
  },
  created() {
    this.date = this.value;
  },
};
</script>
