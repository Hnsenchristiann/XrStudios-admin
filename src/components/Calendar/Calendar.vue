<template>
    <div ref="container" class="toastui-vue-calendar" />
</template>

<script>
import Calendar from '@toast-ui/calendar';
import '@toast-ui/calendar/dist/toastui-calendar.css';
export default{
  name: 'Calendar',
  props: {
    view: String,
    useFormPopup: {
      type: Boolean,
      default: () => undefined,
    },
    useDetailPopup: {
      type: Boolean,
      default: () => undefined,
    },
    isReadOnly: {
      type: Boolean,
      default: () => undefined,
    },
    usageStatistics: {
      type: Boolean,
      default: () => undefined,
    },
    week: Object,
    month: Object,
    gridSelection: {
      type: [Object, Boolean],
      default: () => undefined,
    },
    timezone: Array,
    theme: Object,
    template: Object,
    calendars: Array,
    events: Array,
  },
  data() {
    return {
      calendarInstance: null
    };
  },
  watch: {
    view(value) {
      this.calendarInstance.changeView(value + '');
    },
    useFormPopup(value) {
      this.calendarInstance.setOptions({ useFormPopup: !!value });
    },
    useDetailPopup(value) {
      this.calendarInstance.setOptions({ useDetailPopup: !!value });
    },
    isReadOnly(value) {
      this.calendarInstance.setOptions({ isReadOnly: !!value });
    },
    week(value) {
      this.calendarInstance.setOptions({ week: {...value} });
    },
    month(value) {
      this.calendarInstance.setOptions({ month: {...value} });
    },
    gridSelection(value) {
      this.calendarInstance.setOptions({ gridSelection: {...value} });
    },
    timezone(value) {
      this.calendarInstance.setOptions({ timezone: {...value} });
    },
    theme(value) {
      this.calendarInstance.setTheme({...value});
    },
    template(value) {
      this.calendarInstance.setOptions({ template: {...value} });
    },
    calendars(value) {
      this.calendarInstance.setCalendars({...value});
    },
    events(value) {
      this.calendarInstance.clear();
      this.calendarInstance.createEvents([...value]);
    },
  },
  mounted() {
    this.calendarInstance = new Calendar('.toastui-vue-calendar', {
      defaultView: this.view + '',
      useFormPopup: !!this.useFormPopup,
      useDetailPopup: !!this.useDetailPopup,
      isReadOnly: !!this.isReadOnly,
      usageStatistics: !!this.usageStatistics,
      week: {...this.week},
      month: {...this.month},
      gridSelection: {...this.gridSelection},
      timezone: {zones: [...this.timezone]},
      theme: {...this.theme},
      template: {...this.template},
      calendars: [...this.calendars],
    });
    this.addEventListeners();
    this.calendarInstance.createEvents([...this.events]);
  },
  beforeUnmount() {
    this.calendarInstance.off();
    this.calendarInstance.destroy();
  },
  methods: {
    addEventListeners() {
      Object.keys(this.$attrs).filter(v => v.startsWith('on')).forEach((eventName) => {
        this.calendarInstance.on(eventName, (...args) => this.$emit(eventName, ...args));
      });
    },
    getRootElement() {
      return this.$refs.container;
    },
    getInstance() {
      return this.calendarInstance;
    },
  },
}
</script>