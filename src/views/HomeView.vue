<template>
  <LayoutAuthenticated>
    <!-- <SectionMain>
      <SectionTitleLineWithButton
        :icon="mdiChartTimelineVariant"
        title="Overview"
        main
      >
        <BaseButton
          href="https://github.com/justboil/admin-one-vue-tailwind"
          target="_blank"
          :icon="mdiGithub"
          label="Star on GitHub"
          color="contrast"
          rounded-full
          small
        />
      </SectionTitleLineWithButton>

      <div class="grid grid-cols-1 gap-6 lg:grid-cols-3 mb-6">
        <CardBoxWidget
          trend="12%"
          trend-type="up"
          color="text-emerald-500"
          :icon="mdiAccountMultiple"
          :number="512"
          label="Clients"
        />
        <CardBoxWidget
          trend="12%"
          trend-type="down"
          color="text-blue-500"
          :icon="mdiCartOutline"
          :number="7770"
          prefix="$"
          label="Sales"
        />
        <CardBoxWidget
          trend="Overflow"
          trend-type="alert"
          color="text-red-500"
          :icon="mdiChartTimelineVariant"
          :number="256"
          suffix="%"
          label="Performance"
        />
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        <div class="flex flex-col justify-between">
          <CardBoxTransaction
            v-for="(transaction, index) in transactionBarItems"
            :key="index"
            :amount="transaction.amount"
            :date="transaction.date"
            :business="transaction.business"
            :type="transaction.type"
            :name="transaction.name"
            :account="transaction.account"
          />
        </div>
        <div class="flex flex-col justify-between">
          <CardBoxClient
            v-for="client in clientBarItems"
            :key="client.id"
            :name="client.name"
            :login="client.login"
            :date="client.created"
            :progress="client.progress"
          />
        </div>
      </div>

      <SectionBannerStarOnGitHub class="mt-6 mb-6" />

      <SectionTitleLineWithButton :icon="mdiChartPie" title="Trends overview">
        <BaseButton
          :icon="mdiReload"
          color="whiteDark"
          @click="fillChartData"
        />
      </SectionTitleLineWithButton>

      <CardBox class="mb-6">
        <div v-if="chartData">
          <line-chart :data="chartData" class="h-96" />
        </div>
      </CardBox>

      <SectionTitleLineWithButton :icon="mdiAccountMultiple" title="Clients" />

      <NotificationBar color="info" :icon="mdiMonitorCellphone">
        <b>Responsive table.</b> Collapses on mobile
      </NotificationBar>

      <CardBox has-table>
        <TableSampleClients />
      </CardBox>
    </SectionMain> -->

    <div class="flex py-3 px-3">
      <div>
        <select
          v-model="selectedView"
          class="view-select"
        >
          <option
            v-for="view in viewOptions"
            :key="view.value"
            :value="view.value"
          >
            {{ view.title }}
          </option>
        </select>
      </div>

      <div class="flex items-justify-center px-2">
        <button
            class="px-1 bg-yellow-300 mx-1 px-5 rounded-lg"
            type="button"
            @click="onClickTodayButton"
          >
            Today
          </button>
          <button
            class="px-1 bg-red-300 mx-1 px-5 rounded-lg"
            type="button"
            @click="onClickMoveButton(-1)"
          >
            Prev
          </button>
          <button
            class="px-1 bg-blue-300 mx-1 px-5 rounded-lg"
            type="button"
            @click="onClickMoveButton(1)"
          >
            Next
        </button>
      </div>

      <div class="date-range">{{ dateRangeText }}</div>
    </div>

    <calendar
      ref="calendar"
      style="height: 800px"
      :view="'month'"
      :use-form-popup="true"
      :use-detail-popup="true"
      :week="{
        showTimezoneCollapseButton: true,
        timezonesCollapsed: false,
        eventView: true,
        taskView: true,
      }"
      :month="{ startDayOfWeek: 2 }"
      :timezone="zones"
      :theme="theme"
      :template="{
        milestone: getTemplateForMilestone,
        allday: getTemplateForAllday,
      }"
      :grid-selection="true"
      :calendars="calendars"
      :events="events"
      @selectDateTime="onSelectDateTime"
      @beforeCreateEvent="onBeforeCreateEvent"
      @beforeUpdateEvent="onBeforeUpdateEvent"
      @beforeDeleteEvent="onBeforeDeleteEvent"
      @afterRenderEvent="onAfterRenderEvent"
      @clickDayName="onClickDayName"
      @clickEvent="onClickEvent"
      @clickTimezonesCollapseBtn="onClickTimezonesCollapseBtn"
    /> 

    <div>
      <button @click="demoViewMode('day')">Day View Mode</button>
      <button @click="demoViewMode('week')">Week View Mode</button>
      <button @click="demoViewMode('month')">Month View Mode</button>
    </div>

    <gantt-chart
      :view-mode="mode"
      :tasks="tasks"
      @task-updated="debugEventLog.push($event)"
      @task-date-updated="debugEventLog.push($event)"
      @task-progress-change="debugEventLog.push($event)" 
    />

    <button @click="addRandomTask">
      Add
    </button>

    <div>
      <h5>These are the events being emitted by the Vue.js component wrapper for Frappe Gantt</h5>
      <ul>
        <li
          v-for="event in debugEventLog"
          :key="event.id">
          {{ event }}
        </li>
      </ul>
    </div>
  </LayoutAuthenticated>
</template>

<script>
import Calendar from '../components/Calendar/Calendar.vue';
import GanttChart from '../components/GanttChart/GanttChart.vue';
import { events } from '../components/Calendar/mock-data';
import { theme  } from '../components/Calendar/theme';
import '@toast-ui/calendar/dist/toastui-calendar.css';
import 'tui-date-picker/dist/tui-date-picker.min.css';
import 'tui-time-picker/dist/tui-time-picker.min.css';

export default {
  name: 'HomeView',
  components: {
    Calendar,
    GanttChart
  },
  data() {
    return {
      calendars: [
        {
          id: '0',
          name: 'Private',
          backgroundColor: '#9e5fff',
          borderColor: '#9e5fff',
          dragBackgroundColor: '#9e5fff',
        },
        {
          id: '1',
          name: 'Company',
          backgroundColor: '#00a9ff',
          borderColor: '#00a9ff',
          dragBackgroundColor: '#00a9ff',
        },
      ],
      events,
      zones: [
        {
          timezoneName: 'Asia/Seoul',
          displayLabel: 'Seoul',
          tooltip: 'UTC+09:00',
        },
        {
          timezoneName: 'Pacific/Guam',
          displayLabel: 'Guam',
          tooltip: 'UTC+10:00',
        },
      ],
      theme,
      selectedView: 'month',
      viewOptions: [
        {
          title: 'Monthly',
          value: 'month',
        },
        {
          title: 'Weekly',
          value: 'week',
        },
        {
          title: 'Daily',
          value: 'day',
        },
      ],
      dateRangeText: '',
      mode: 'week',
      tasks: [
        {
          id: '1',
          name: 'Hello',
          start: '2019-01-01',
          end: '2019-01-05',
          progress: 10,
        },
        {
          id: '2',
          name: 'World',
          start: '2019-01-05',
          end: '2019-01-10',
          progress: 20,
          dependencies: '1'
        },
        {
          id: '3',
          name: 'From',
          start: '2019-01-10',
          end: '2019-01-15',
          progress: 30,
          dependencies: '2'
        },
        {
          id: '4',
          name: 'Lloyd',
          start: '2019-01-15',
          end: '2019-01-20',
          progress: 40,
          dependencies: '3'
        }
      ],
      debugEventLog: []
    };
  },
  computed: {
    calendarInstance() {
      return this.$refs.calendar.getInstance();
    },
  },
  watch: {
    selectedView(newView) {
      this.calendarInstance.changeView(newView);
      this.setDateRangeText();
    },
  },
  mounted() {
    this.setDateRangeText();
  },
  methods: {
    getTemplateForMilestone(event) {
      return `<span style="color: #fff; background-color: ${event.backgroundColor};">${event.title}</span>`;
    },
    getTemplateForAllday(event) {
      return `[All day] ${event.title}`;
    },
    onSelectDateTime({ start, end }) {
      console.group('onSelectDateTime');
      console.log(`Date : ${start} ~ ${end}`);
      console.groupEnd();
    },
    onBeforeCreateEvent(eventData) {
      const event = {
        calendarId: eventData.calendarId || '',
        id: String(Math.random()),
        title: eventData.title,
        isAllday: eventData.isAllday,
        start: eventData.start,
        end: eventData.end,
        category: eventData.isAllday ? 'allday' : 'time',
        dueDateClass: '',
        location: eventData.location,
        state: eventData.state,
        isPrivate: eventData.isPrivate,
      };
      this.calendarInstance.createEvents([event]);
    },
    onBeforeUpdateEvent(updateData) {
      console.group('onBeforeUpdateEvent');
      console.log(updateData);
      console.groupEnd();
      const targetEvent = updateData.event;
      const changes = { ...updateData.changes };
      this.calendarInstance.updateEvent(targetEvent.id, targetEvent.calendarId, changes);
    },
    onBeforeDeleteEvent({ title, id, calendarId }) {
      console.group('onBeforeDeleteEvent');
      console.log('Event Info : ', title);
      console.groupEnd();
      this.calendarInstance.deleteEvent(id, calendarId);
    },
    onAfterRenderEvent({ title }) {
      console.group('onAfterRenderEvent');
      console.log('Event Info : ', title);
      console.groupEnd();
    },
    onClickDayName({ date }) {
      console.group('onClickDayName');
      console.log('Date : ', date);
      console.groupEnd();
    },
    onClickEvent({ nativeEvent, event }) {
      console.group('onClickEvent');
      console.log('MouseEvent : ', nativeEvent);
      console.log('Event Info : ', event);
      console.groupEnd();
    },
    onClickTimezonesCollapseBtn(timezoneCollapsed) {
      console.group('onClickTimezonesCollapseBtn');
      console.log('Is Timezone Collapsed?: ', timezoneCollapsed);
      console.groupEnd();
      const newTheme = {
        'week.daygridLeft.width': '100px',
        'week.timegridLeft.width': '100px',
      };
      this.calendarInstance.setTheme(newTheme);
    },
    onClickTodayButton() {
      this.calendarInstance.today();
      this.setDateRangeText();
    },
    onClickMoveButton(offset) {
      this.calendarInstance.move(offset);
      this.setDateRangeText();
    },
    setDateRangeText() {
      const date = this.calendarInstance.getDate();
      const start = this.calendarInstance.getDateRangeStart();
      const end = this.calendarInstance.getDateRangeEnd();
      const startYear = start.getFullYear();
      const endYear = end.getFullYear();
      switch (this.selectedView) {
        case 'month':
          this.dateRangeText = `${date.getFullYear()}.${date.getMonth() + 1}`;
          return;
        case 'day':
          this.dateRangeText = `${date.getFullYear()}.${date.getMonth() + 1}.${date.getDate()}`;
          return;
        default:
          this.dateRangeText = `${startYear}.${start.getMonth() + 1}.${start.getDate()} - ${
            startYear !== endYear ? `${endYear}.` : ''
          }${end.getMonth() + 1}.${end.getDate()}`;
      }
    },
    addRandomTask () {
      const id = uuidv4();
      this.tasks.push({
          id: id,
          name: id,
          start: '2019-01-01',
          end: '2019-01-05',
          progress: Math.random() * 100,
      });
    },
    demoViewMode (viewMode) {
      this.mode = viewMode;
    }
  },
};
</script>

<script setup>
import { computed, ref, onMounted } from "vue";
import { useMainStore } from "@/stores/main";
import {
  mdiAccountMultiple,
  mdiCartOutline,
  mdiChartTimelineVariant,
  mdiMonitorCellphone,
  mdiReload,
  mdiGithub,
  mdiChartPie,
} from "@mdi/js";
import * as chartConfig from "@/components/Charts/chart.config.js";
import LineChart from "@/components/Charts/LineChart.vue";
import SectionMain from "@/components/SectionMain.vue";
import CardBoxWidget from "@/components/CardBoxWidget.vue";
import CardBox from "@/components/CardBox.vue";
import TableSampleClients from "@/components/TableSampleClients.vue";
import NotificationBar from "@/components/NotificationBar.vue";
import BaseButton from "@/components/BaseButton.vue";
import CardBoxTransaction from "@/components/CardBoxTransaction.vue";
import CardBoxClient from "@/components/CardBoxClient.vue";
import LayoutAuthenticated from "@/layouts/LayoutAuthenticated.vue";
import SectionTitleLineWithButton from "@/components/SectionTitleLineWithButton.vue";
import SectionBannerStarOnGitHub from "@/components/SectionBannerStarOnGitHub.vue";

const chartData = ref(null);

const fillChartData = () => {
  chartData.value = chartConfig.sampleChartData();
};

onMounted(() => {
  fillChartData();
});

const mainStore = useMainStore();

const clientBarItems = computed(() => mainStore.clients.slice(0, 4));

const transactionBarItems = computed(() => mainStore.history);
</script>
