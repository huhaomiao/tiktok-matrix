<template>
  <div class="w-full">
    <Pagination :items="filter_tasks" :searchKeys="['id', 'device_index', 'script_name']" 
    :searchTermPlaceholder="$t('searchTaskPlaceholder')"
    @refresh="get_tasks">
      <template v-slot:buttons>
        <MyButton @click="retry_all_failed" label="retryAllFaied" icon="fa fa-repeat" />
        <MyButton @click="clearAll" label="clearAll" icon="fa fa-trash" />
        <select v-model="searchStatus" class="select select-md w-32 select-bordered ml-2">
          <option value="">{{ $t('allStatus') }}</option>
          <option value="0">{{ $t('waiting') }}</option>
          <option value="1">{{ $t('execing') }}</option>
          <option value="2">{{ $t('success') }}</option>
          <option value="3">{{ $t('failed') }}</option>
        </select>
      </template>
      <template v-slot:default="slotProps">
        <div class="overflow-x-auto">
          <table class="table table-md">
            <thead>
              <tr>
                <th>{{ $t('id') }}</th>
                <th>{{ $t('scriptName') }}</th>
                <th>{{ $t('scriptArgs') }}</th>
                <th>{{ $t('startTime') }}</th>
                <th>{{ $t('status') }}</th>
                <!-- <th>{{ $t('remark') }}</th> -->
                <th>{{ $t('device') }}</th>
                <th>{{ $t('actions') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(task, index) in slotProps.items" :key="index">
                <td>{{ task.id }}</td>
                <td><span class="badge badge-ghost badge-md">{{ task.script_name }}</span></td>
                <td>
                  <span class="badge badge-ghost badge-md">{{ truncateScriptArgs(task.script_args) }}</span>
                </td>
                <td>
                  <span class="badge badge-ghost badge-md">{{ task.start_time }}</span>
                </td>
                <td>
                  <div class="badge badge-neutral badge-md" v-if="task.status == '0'">{{ $t('waiting') }}</div>
                  <div class="badge badge-primary badge-md" v-else-if="task.status == '1'">{{ $t('execing') }}</div>
                  <div class="badge badge-success badge-md" v-else-if="task.status == '2'">{{ $t('success') }}</div>
                  <div class="badge badge-error badge-md" v-else-if="task.status == '3'">{{ $t('failed') }}</div>
                </td>
                <!-- <td><span class="badge badge-ghost badge-md">{{ task.remark }}</span></td> -->
                <td>
                  <a class="link link-primary" @click="show_device(task.serial)" v-if="task.device_index">{{
                    task.device_index }}</a>
                  <span v-else class="text text-error">{{ $t('offline') }}</span>
                </td>
                <td>
                  <div class="space-x-4">
                    <button class="btn btn-md btn-primary rounded" @click="retry(task)">{{
                      $t('retry') }}</button>
                    <button class="btn btn-md btn-error rounded" @click="deleteTask(task)">{{
                      $t('delete') }}</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </template>
    </Pagination>


  </div>
</template>
<script>
import Modal from '../Modal.vue'
import MyButton from '../Button.vue'
import Pagination from '../Pagination.vue'

export default {
  name: 'app',
  components: {
    Modal,
    MyButton,
    Pagination
  },
  props: {
    devices: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      tasks: [],
      currentDevice: null,
      searchStatus: ''
    }
  },
  computed: {
    filter_tasks() {
      let tasks = this.tasks
      if (this.searchStatus) {
        tasks = tasks.filter(task => task.status == this.searchStatus)
      }
      return tasks
    },

  },
  methods: {
    truncateScriptArgs(scriptArgs) {
      if (scriptArgs.length > 30) {
        return `${scriptArgs.slice(0, 10)}...${scriptArgs.slice(-10)}`;
      }
      return scriptArgs;
    },
    async clearAll() {
      this.$service
        .delete_all_tasks()
        .then(() => {
          this.get_tasks()
        })
    },

    async show_device(serial) {
      let mydevice = this.devices.find(d => d.serial === serial || d.real_serial === serial)
      await this.$emiter('openDevice', mydevice)
    },
    async get_tasks() {
      this.currentTask = null
      this.$service
        .get_tasks()
        .then(async (res) => {
          this.tasks = res.data
          this.tasks.forEach(task => {
            task.device_index = this.devices.find(device => device.serial === task.serial || device.real_serial === task.serial)?.key
          })
          await this.$emiter('reload_tasks', {})
        }).catch(async (err) => {
          await this.$emiter('NOTIFY', {
            type: 'error',
            message: err.message,
            timeout: 2000
          })
        })
    },

    async retry(task) {
      this.$service
        .update_task({
          id: task.id,
          status: 0,
        })
        .then(() => {
          this.get_tasks()
        })
    },
    async deleteTask(task) {
      this.$service
        .delete_task({
          id: task.id
        })
        .then(res => {
          console.log(res)
          this.get_tasks()
        })
    },

    async retry_all_failed() {
      this.$service
        .retry_all_failed_tasks()
        .then(() => {
          this.get_tasks()
        })
    }
  },
  async mounted() {
    this.get_tasks()
  }
}
</script>
