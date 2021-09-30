<template>
    <div class="content-wrapper">
      <div id="Schedule"></div>
    </div>
</template>

<script  lang="ts">
import { enableRipple } from '@syncfusion/ej2-base';
enableRipple(true);
import {
  Schedule,
  ScheduleModel,
  TimelineViews,
  TimelineMonth,
  Agenda,
  Resize,
  DragAndDrop,
  EventSettingsModel
} from '@syncfusion/ej2-schedule'
import * as dataSource from '../../models/datasource.json';
import { extend } from '@syncfusion/ej2-base';
import Vue from 'vue';
import { Component, Watch, Prop } from 'vue-property-decorator'
Schedule.Inject(TimelineViews, TimelineMonth, Agenda, Resize, DragAndDrop);
import { Employee } from '~/models/employee'
import { Event } from '~/models/event'

@Component
export default class Scheduler extends Vue {
  data : Event[] = <Event[]>extend([], (dataSource as any).resourceData.concat((dataSource as any).timelineResourceData), undefined, true);

  categories : Employee[] = [
    { text: 'Nancy', id: 1, groupId: 1, color: '#df5286' },
    { text: 'Steven', id: 2, groupId: 1, color: '#7fa900' },
    { text: 'Robert', id: 3, groupId: 2, color: '#ea7a57' },
    { text: 'Smith', id: 4, groupId: 2, color: '#5978ee' },
    { text: 'Michael', id: 5, groupId: 3, color: '#df5286' },
    { text: 'Root', id: 6, groupId: 3, color: '#00bdae' }
  ]

  projects = [
    { text: 'PROJECT 1', id: 1, color: '#cb6bb2' },
    { text: 'PROJECT 2', id: 2, color: '#56ca85' },
    { text: 'PROJECT 3', id: 3, color: '#df5286' }
  ]

  private schedule : Schedule = new Schedule()

  get unassignedId() : number {
    let lastId = 0
    this.categories.forEach(el => {
      if (el.id > lastId) lastId = el.id
    })
    return lastId + 1
  }

  get events() : Event[] {
    return this.data.map(e => ({
      ...e,
      TaskId: e.TaskId ? e.TaskId : this.unassignedId
    }))
  }

  get employees() : Employee[] {
    const unassignedRows = this.projects.map(p => ({
      text: 'N/A', id: this.unassignedId, groupId: p.id, color: '#b4b3b4'
    }))

    return [ ...this.categories, ...unassignedRows]
  }

  get scheduleOptions() : ScheduleModel {
    const opts = <ScheduleModel>{
      width: '100%',
      height: '650px',
      selectedDate: new Date(2018, 3, 4),
      views: ['TimelineDay', 'TimelineWeek', 'TimelineWorkWeek', 'TimelineMonth', 'Agenda'],
      currentView: 'TimelineDay',
      group: {
        resources: ['Projects', 'Categories']
      },
      resources: [
        {
          field: 'ProjectId', title: 'Choose Project', name: 'Projects',
          dataSource: [],
          textField: 'text', idField: 'id', colorField: 'color'
        }, {
          field: 'TaskId', title: 'Category',
          name: 'Categories',
          dataSource: [],
          textField: 'text', idField: 'id', groupIDField: 'groupId', colorField: 'color'
        }, {}
      ],
      eventSettings: {},
      rowAutoHeight: true,
      dragStop: this.dragStop
    }

    if (!opts.resources) opts.resources = []

    const project = opts.resources.find(r => r.field === 'ProjectId')

    if (project) {
      project.dataSource = this.projects
    }

    const category = opts.resources.find(r => r.field === 'TaskId')

    if (category) {
      category.dataSource = this.employees
    }

    if (!opts.eventSettings) {
      opts.eventSettings = <EventSettingsModel>{ dataSource : undefined }
    }
    opts.eventSettings.dataSource = this.events

    return opts
  }

  mounted (): void {
    this.schedule = new Schedule(this.scheduleOptions, '#Schedule');
  }

  dragStop (event: any) : void {
    if (event.data && !event.data.TaskId) {
      const idx = this.data.findIndex(e => e.Id === event.data.Id)
      if (idx > -1) {
        this.data[idx] =  { ...event.data, TaskId : this.unassignedId }
        this.data = [...this.data]
      }
    }
  }

  @Watch('data', {deep: true})
  dataChanged() {
    this.schedule.eventSettings = {
      dataSource: this.events
    }
  }
}
</script>

<style scoped>

</style>
