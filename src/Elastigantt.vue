<template>
<elastigantt-main :tasks="tasks" :options="options" @calculateTaskListColumnsWidths="calculateTaskListColumnsWidths"></elastigantt-main>
</template>

<script>
function getOptions(userOptions) {
  return {
    debug: false,
    width: 0,
    height: 0,
    svgElement: null,
    scope: {
      before: 1,
      after: 1
    },
    times: {
      timeScale: 60 * 1000,
      timeZoom: 17,
      timePerPixel: 0,
      fistDate: null,
      firstTime: null, // firstDate getTime()
      lastDate: null,
      lastTime: null, // last date getTime()
      totalViewDurationMs: 0,
      totalViewDurationPx: 0,
      stepMs: 24 * 60 * 60 * 1000,
      stepPx: 0,
      steps: 0
    },
    row: {
      height: 24,
      styles: {
        bar: {
          'fill': '#ff0000a0',
          'stroke': '#ff0000',
          'stroke-width': '1'
        },
        text: {
          'fill': '#ffffff',
          'font-family': 'sans-serif',
          'font-size': '12px'
        }
      },
      showText: true
    },
    treeText: {
      offset: 0,
      styles: {
        text: {
          'font-family': 'sans-serif',
          'font-size': '12px',
          'font-weight': 'normal',
          'fill': '#000000a0'
        },
        background: {
          fill: '#ffffffb0'
        }
      },
      xPadding: 10
    },
    dependencyLines: {
      style: {
        'fill': 'transparent',
        'stroke': '#FFa00090',
        'stroke-width': 2
      }
    },
    progress: {
      width: 20,
      height: 6,
      styles: {
        line: {
          'stroke': '#ffffff85',
          'stroke-width': 20
        },
        bar: {
          'fill': "url(#diagonalHatch)",
          'transform': 'translateY(0.1) scaleY(0.8)'
        }
      }
    },
    horizontalGrid: {
      gap: 6,
      strokeWidth: 1,
      style: 'stroke:#00000010;strokeWidth:1',
      lines: []
    },
    verticalGrid: {
      strokeWidth: 1,
      style: 'stroke:#00000010;strokeWidth:1',
      lines: []
    },
    info: {
      style: 'fill:#000000a0',
      textStyle: 'fill:#fff',
      fontFamily: 'sans-serif',
      fontSize: '12px',
      fontWeight: 'bold',
      display: false
    },
    taskList: {
      display: true,
      styles: {
        row: {
          'background': 'transparent',
          'border-color': '#00000010'
        },
        column: {
          'border-color': '#00000010',
          'height': null,
          'width': null,
          'line-height': null
        },
        header: {
          'background': 'linear-gradient(to bottom,#fff,#f5f5f5)',
          'border-color': '#00000010'
        },
        label: {
          'display': 'inline-block',
          'margin': 'auto 6px',
          'color': '#404040'
        },
        value: {
          'margin': 'auto 6px',
          'overflow': 'hidden',
          'text-overflow': 'ellipsis',
          'line-height': '1.5em',
          'word-break': 'keep-all',
          'font-family': 'sans-serif',
          'font-size': '12px',
          'white-space': 'nowrap',
          'color': '#606060'
        },
        expander: {
          stroke: '#909090',
          strokeWidth: 1,
          'fill': '#ffffffa0'
        }
      },
      columns: [{
        id: 0,
        label: 'ID',
        value: 'id',
        width: 40,
        styles: {
          label: {}
        }
      }],
      resizerWidth: 0,
      percent: 100,
      width: 0,
      finalWidth: 0,
      expander: {
        size: 16,
        columnWidth: 24
      }
    },
    calendar: {
      hours: [],
      days: [],
      months: [],
      gap: 6,
      height: 0,
      styles: {
        wrapper: {
          'width': '100%',
          'height': '100%',
          'background': 'linear-gradient(to bottom,#fff,#f5f5f5)',
          'border-color': '#00000010'
        },
        row: {
          fill: 'transparent',
          stroke: '#00000010'
        },
        column: {
          'stroke': '#00000010',
          'stroke-width': 1,
          'fill': 'transparent'
        },
        text: {
          fontFamily: 'sans-serif',
          'fill': '#404040'
        }
      },
      hour: {
        height: 20,
        display: true,
        fontSize: '12px',
        format: {
          short(date) {
            return dayjs(date).locale(userOptions.locale.code).format('HH');
          },
          medium(date) {
            return dayjs(date).locale(userOptions.locale.code).format('HH:mm');
          },
          long(date) {
            return dayjs(date).locale(userOptions.locale.code).format('HH:mm');
          }
        }
      },
      day: {
        height: 20,
        display: true,
        fontSize: '12px',
        format: {
          short(date) {
            return dayjs(date).locale(userOptions.locale.code).format('DD');
          },
          medium(date) {
            return dayjs(date).locale(userOptions.locale.code).format('DD ddd');
          },
          long(date) {
            return dayjs(date).locale(userOptions.locale.code).format('DD dddd');
          }
        }
      },
      month: {
        height: 20,
        display: true,
        fontSize: '12px',
        format: {
          short(date) {
            return dayjs(date).locale(userOptions.locale.code).format('MM');
          },
          medium(date) {
            return dayjs(date).locale(userOptions.locale.code).format('\'YY MMM');
          },
          long(date) {
            return dayjs(date).locale(userOptions.locale.code).format('YYYY MMMM (MM)');
          }
        }
      }
    },
    defs: []
  };
}

import Main from './components/Main.vue';

export default {
  components: {
    'elastigantt-main': Main,
  },
  props: [
    'tasks', 'options'
  ],
  provide() {
    const provider = {};
    const self = this;
    Object.defineProperty(provider, 'root', {
      enumerable: true,
      get: () => self
    });
    return provider;
  },
  data() {
    return {
      state: {}
    };
  },
  methods: {
    isObject(item) {
      return item && typeof item === 'object' && !Array.isArray(item);
    },
    mergeDeep(target, ...sources) {
      if (!sources.length) {
        return target;
      }
      const source = sources.shift();
      if (this.isObject(target) && this.isObject(source)) {
        for (const key in source) {
          if (this.isObject(source[key])) {
            if (!target[key])
              Object.assign(target, {
                [key]: {}
              });
            this.mergeDeep(target[key], source[key]);
          } else {
            Object.assign(target, {
              [key]: source[key]
            });
          }
        }
      }
      return this.mergeDeep(target, ...sources);
    },
    initialize() {
      this.state = this.mergeDeep(getOptions(this.options), this.options, {
        tasks: this.tasks
      });
      dayjs.locale(options.locale, null, true);
      this.state.taskList.columns = this.state.taskList.columns.map(column => {
        column.finalWidth = (column.width / 100) * this.state.taskList.percent;
        column.styles = this.mergeDeep({}, this.state.taskList.styles, column.styles);
        if (typeof column.style === 'undefined') {
          column.style = {
            'height': 0 + "px",
            'line-height': 0 + "px",
            'width': 0 + "px"
          };
        }
        column.style = this.mergeDeep({}, this.state.taskList.styles.column, column.style);
        return column;
      });
      // initialize observer
      this.state.tasks = this.state.tasks.map(task => {
        task.x = 0;
        task.y = 0;
        task.width = 0;
        task.height = 0;
        task.tooltip = {
          visible: false
        };
        task.mouseOver = false;
        task.dependencyLines = [];
        if (typeof task.visible === 'undefined') {
          task.visible = true;
        }
        if (typeof task.collapsed === 'undefined') {
          task.collapsed = false;
        }
        if (typeof task.dependencyLines === 'undefined') {
          task.dependencyLines = [];
        }
        if (typeof task.parentId === 'undefined') {
          task.parentId = null;
        }
        task.children = [];
        task.allChildren = [];
        task.parents = [];
        task.parent = null;
        return task;
      });
      this.state.rootTask = {
        id: null,
        label: 'root',
        children: [],
        allChildren: [],
        parents: [],
        parent: null
      };
      this.state.taskTree = this.makeTaskTree(this.state.rootTask);
      this.state.ctx = document.createElement('canvas').getContext('2d');
    },
    calculateCalendarDimensions() {
      this.state.calendar.height = 0;
      if (this.state.calendar.hour.display) {
        this.state.calendar.height += this.state.calendar.hour.height;
      }
      if (this.state.calendar.day.display) {
        this.state.calendar.height += this.state.calendar.day.height;
      }
      if (this.state.calendar.month.display) {
        this.state.calendar.height += this.state.calendar.month.height;
      }
    },
    calculateTaskListColumnsWidths() {
      console.log('calculate')
      let final = 0;
      this.state.taskList.columns.forEach(column => {
        column.finalWidth = (column.width / 100) * this.state.taskList.percent;
        final += column.finalWidth;
        let height = this.state.row.height + this.state.horizontalGrid.gap * 2 - this.state.horizontalGrid.strokeWidth;
        column.style.height = height + "px";
        column.style['line-height'] = height + "px";
        column.style.width = column.finalWidth + "px";
        return column;
      });
      this.state.taskList.finalWidth = final + this.state.taskList.expander.columnWidth;
      console.log(this.state.taskList.columns)
    },
    resetTaskTree() {
      this.state.rootTask.children = [];
      this.state.rootTask.allChildren = [];
      this.state.rootTask.parent = null;
      this.state.rootTask.parents = [];
      for (let i = 0, len = this.state.tasks.length; i < len; i++) {
        let current = this.tasks[i];
        current.children = [];
        current.allChildren = [];
        current.parent = null;
        current.parents = [];
      }
    },
    makeTaskTree(task) {
      for (let i = 0, len = this.state.tasks.length; i < len; i++) {
        let current = this.tasks[i];
        if (current.parentId === task.id) {
          if (task.parents.length) {
            task.parents.forEach(parent => current.parents.push(parent));
          }
          if (task !== this.root) {
            current.parents.push(task);
            current.parent = task;
          } else {
            current.parents = [];
            current.parent = null;
          }
          current = this.makeTaskTree(current);
          task.allChildren.push(current);
          task.children.push(current);
          current.allChildren.forEach(child => task.allChildren.push(child));
        }
      }
      return task;
    },
    getTask(taskId) {
      return this.tasksById[taskId];
    },
    getChildren(taskId) {
      return this.state.tasks.filter(task => task.parent === taskId);
    },
    getSVG() {
      return this.svgElement.outerHTML;
    },
    getImage(type = 'image/png') {
      return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => {
          const canvas = document.createElement('canvas');
          canvas.width = this.svgElement.clientWidth;
          canvas.height = this.svgElement.clientHeight;
          canvas.getContext('2d').drawImage(img, 0, 0);
          resolve(canvas.toDataURL(type));
        };
        img.src = 'data:image/svg+xml,' + encodeURIComponent(this.getSVG());
      });
    },
  },
  computed: {
    visibleTasks() {
      const firstDate = this.state.times.firstTaskDate.toISOString().split('T')[0] + 'T00:00:00';
      const lastDate = this.state.times.lastTaskDate.toISOString().split('T')[0] + 'T23:59:59.999';
      this.state.times.firstDate = dayjs(firstDate).locale(this.locale).subtract(this.state.scope.before, 'days').toDate();
      this.state.times.lastDate = dayjs(lastDate).locale(this.locale).add(this.state.scope.after, 'days').toDate();
      this.state.times.firstTime = this.state.times.firstDate.getTime();
      this.state.times.lastTime = this.state.times.lastDate.getTime();
      this.state.times.totalViewDurationMs = this.state.times.lastTime - this.state.times.firstTime;
      this.state.taskList.width = this.state.taskList.columns.reduce((prev, current) => {
        return {
          width: prev.width + current.width
        };
      }, {
        width: 0
      }).width;
      let max = this.state.times.timeScale * 60;
      let min = this.state.times.timeScale;
      let steps = max / min;
      let percent = this.state.times.timeZoom / 100;
      this.state.times.timePerPixel = this.state.times.timeScale * steps * percent + Math.pow(2, this.state.times.timeZoom);
      this.state.times.totalViewDurationPx = this.state.times.totalViewDurationMs / this.state.times.timePerPixel;
      this.state.times.stepPx = this.state.times.stepMs / this.state.times.timePerPixel;
      this.state.width = this.state.times.totalViewDurationPx + this.state.verticalGrid.strokeWidth;
      this.state.times.steps = Math.ceil(this.state.times.totalViewDurationPx / this.state.times.stepPx);
      this.calculateCalendarDimensions();
      this.calculateTaskListColumnsWidths();
      this.resetTaskTree();
      this.state.tasks = this.makeTaskTree(this.state.rootTask).allChildren;
      const visibleTasks = this.state.tasks.filter(task => task.visible);
      this.state.height = visibleTasks.length * (this.state.row.height + this.state.horizontalGrid.gap * 2) + this.state.horizontalGrid.gap + this.state.calendar.height + this.state.calendar.styles.column['stroke-width'] + this.state.calendar.gap;
      for (let index = 0, len = visibleTasks.length; index < len; index++) {
        let task = visibleTasks[index];
        task.width = task.durationMs / this.state.times.timePerPixel - this.state.verticalGrid.strokeWidth;
        if (task.width < 0) {
          task.width = 0;
        }
        task.height = this.state.row.height;
        let x = task.startTime - this.state.times.firstTime;
        if (x) {
          x = x / this.state.times.timePerPixel;
        }
        task.x = x + this.state.verticalGrid.strokeWidth;
        task.y = (this.state.row.height + this.state.horizontalGrid.gap * 2) * index + this.state.horizontalGrid.gap + this.state.calendar.height + this.state.calendar.styles.column['stroke-width'] + this.state.calendar.gap;
      }
      return visibleTasks;
    },

  },
  created() {
    this.initialize();
    this.tasksById = {};
    this.state.tasks.forEach(task => (this.tasksById[task.id] = task));
    let tasks = this.tasks;
    let firstTaskTime = Number.MAX_SAFE_INTEGER;
    let lastTaskTime = 0;
    let firstTaskDate,
      lastTaskDate;
    for (let index = 0, len = this.state.tasks.length; index < len; index++) {
      let task = this.tasks[index];
      task.startDate = new Date(task.start);
      task.startTime = task.startDate.getTime();
      task.durationMs = task.duration * 1000;
      if (task.startTime < firstTaskTime) {
        firstTaskTime = task.startTime;
        firstTaskDate = task.startDate;
      }
      if (task.startTime + task.durationMs > lastTaskTime) {
        lastTaskTime = task.startTime + task.durationMs;
        lastTaskDate = new Date(task.startTime + task.durationMs);
      }
    }
    this.state.times.firstTaskTime = firstTaskTime;
    this.state.times.lastTaskTime = lastTaskTime;
    this.state.times.firstTaskDate = firstTaskDate;
    this.state.times.lastTaskDate = lastTaskDate;
  },
}
</script>