<template>
<div class="elastigantt__header">
  <div class="elastigantt__header-dates">{{root.state.scroll.tree.dateTime.left}} - {{root.state.scroll.tree.dateTime.right}}</div>
  <div class="elastigantt__header-options">
    <button class="elastigantt__btn-recenter btn btn-primary mr-4" @click="recenterPosition">{{root.state.locale.Now}}</button>
    <label class="mr-2">{{root.state.locale['X-Scale']}}<input type="range" v-model="scale" max="24" min="2"></label>
    <label class="mr-2">{{root.state.locale['Y-Scale']}}<input type="range" v-model="height" max="100" min="6"></label>
    <label class="mr-2">{{root.state.locale['Before/After']}}<input type="range" v-model="scope" max="100" min="0"></label>
    <label class="mr-2">{{root.state.locale['Task list width']}}<input type="range" v-model="divider" max="100" min="0"></label>
    <label class="mr-2"><input type="checkbox" class="mr-1" v-model="root.state.taskList.display">{{root.state.locale['Display task list']}}</label>
    <!--<button class="elastigantt__btn-img" @click="getImage">Get image</button>-->
  </div>
</div>
</template>
<script>
export default {
  inject: ['root'],
  data() {
    return {};
  },
  methods: {
    getImage() {
      const code = this.root.getImage('image/png').then(imgB64 => {
        const link = document.createElement('a');
        link.href = imgB64;
        link.download = 'Elastigantt.png';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      });
    },
    recenterPosition() {
      this.root.$emit('recenterPosition')
    }
  },
  computed: {
    scale: {
      get() {
        return this.root.state.times.timeZoom;
      },
      set(value) {
        this.root.state.times.timeZoom = Number(value);
      }
    },
    height: {
      get() {
        return this.root.state.row.height;
      },
      set(value) {
        this.root.state.row.height = Number(value);
        this.root.calculateTaskListColumnsWidths();
      }
    },
    scope: {
      get() {
        return this.root.state.scope.before;
      },
      set(value) {
        this.root.state.scope.before = Number(value);
        this.root.state.scope.after = Number(value);
      }
    },
    divider: {
      get() {
        return this.root.state.taskList.percent;
      },
      set(value) {
        this.root.state.taskList.percent = Number(value);
        this.root.calculateTaskListColumnsWidths();
      }
    }
  }
}
</script>
