<template>
  <div v-if="!isCopying">
    <div>
      <b-field label="Source">
        <b-select placeholder="---" v-model="sourceMountPoint">
          <option
            v-for="drive in driveList"
            :value="drive.mountPoint"
            :key="drive.mountPoint">
            {{ drive.description }}
          </option>
        </b-select>
      </b-field>
      <b-field label="Destination">
        <b-select placeholder="---" v-model="destinationMountPoint">
          <option
            v-for="drive in driveList"
            :value="drive.mountPoint"
            :key="drive.mountPoint">
            {{ drive.description }}
          </option>
        </b-select>
      </b-field>
      <b-field label="Destination Subfolder">
        <b-input v-model="destinationSubFolder"></b-input>
      </b-field>

      <b-button @click="startRsync">start Copying</b-button>
      <b-button @click="refreshDrives">Refresh Drives</b-button>
    </div>
  </div>

</template>

<script>
const driveListModule = require('drivelist')
const { exec } = require('child_process')

export default {
  name: 'Main.vue',

  mounted () {
    this.refreshDrives()
  },

  data () {
    return {
      driveList: [],
      isCopying: false,
      sourceMountPoint: null,
      destinationMountPoint: null,
      destinationSubFolder: 'default'
    }
  },

  methods: {
    async refreshDrives () {
      let allDrives = await driveListModule.list()

      this.driveList = []
      for (let drive of allDrives) {
        if (drive.isUSB === true && drive.mountpoints !== undefined && drive.mountpoints.length > 0) {
          this.driveList.push({
            description: drive.description,
            mountPoint: drive.mountpoints[0].path
          })
        }
      }
    },

    startRsync () {
      if (this.sourceMountPoint === null || this.destinationMountPoint === null) {
        return this.alert('one or more devices not set')
      }

      if (this.sourceMountPoint === this.destinationMountPoint) {
        return this.alert('source and destination are the same')
      }

      if (this.destinationSubFolder.length === 0) {
        return this.alert('destination subfolder must be set. standard value is <b>default</b>')
      }

      this.info('started terminal with generated rsync command')

      let execCommand = `lxterminal -e 'rsync -av --info=progress2 ${this.sourceMountPoint}/ ${this.destinationMountPoint}/${this.destinationSubFolder}/; read -p "RSYNC FINISHED"'`

      console.log(execCommand)
      exec(execCommand)
    },

    info (message) {
      this.$buefy.dialog.alert({
        title: 'Info',
        message,
        type: 'success',
        ariaRole: 'dialog',
        ariaModal: true
      })
    },

    alert (message) {
      this.$buefy.dialog.alert({
        title: 'Error',
        message,
        type: 'is-danger',
        ariaRole: 'alertdialog',
        ariaModal: true
      })
    }
  }
}
</script>

<style scoped>

</style>
