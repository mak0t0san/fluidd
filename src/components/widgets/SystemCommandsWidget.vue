<template>
  <div>
    <v-list-group
      prepend-icon="$host"
      no-action>
      <template v-slot:activator>
        <v-list-item-content>
          <v-list-item-title>{{ $t('Host') }}</v-list-item-title>
        </v-list-item-content>
      </template>

      <v-list-item
        @click="handleHostReboot"
        :disabled="printerPrinting">
        <v-list-item-title>{{ $t('Reboot') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="error">$powerCycle</v-icon>
        </v-list-item-icon>
      </v-list-item>

      <v-list-item
        @click="handleHostShutdown"
        :disabled="printerPrinting">
        <v-list-item-title>{{ $t('Shutdown') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="error">$power</v-icon>
        </v-list-item-icon>
      </v-list-item>
    </v-list-group>

    <v-list-group
      v-if="devicePowerPluginEnabled"
      prepend-icon="$power"
      no-action>
      <template v-slot:activator>
        <v-list-item-content>
          <v-list-item-title>{{ $t('Power Plugin') }}</v-list-item-title>
        </v-list-item-content>
      </template>

      <v-list-item
        v-for="(device, index) in powerDevices"
        :key="index"
        :disabled="(device.status === 'error' || device.status === 'init' || (printerPrinting && device.locked_while_printing))"
        @click="togglePowerDevice(device, `${waits.onDevicePowerToggle}${device.device}`)"
        :loading="hasWait(`${waits.onDevicePowerToggle}${device.device}`)"
      >
        <v-list-item-title>{{ getPowerButtonText(device) }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon>{{ getPowerIcon(device) }}</v-icon>
        </v-list-item-icon>
      </v-list-item>

    </v-list-group>

    <v-list-group
      prepend-icon="$restart"
      no-action>
      <template v-slot:activator>
        <v-list-item-content>
          <v-list-item-title>{{ $t('Services') }}</v-list-item-title>
        </v-list-item-content>
      </template>

      <v-list-item @click="serviceRestartMoonraker(); $emit('click')"
        :disabled="printerPrinting">
        <v-list-item-title class="text-wrap">{{ $t('Restart Moonraker') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="warning">$restart</v-icon>
        </v-list-item-icon>
      </v-list-item>

      <v-list-item
        v-if="!klippyConnected"
        @click="serviceRestartKlipper(); $emit('click')"
        :disabled="printerPrinting">
        <v-list-item-title class="text-wrap">{{ $t('Restart Klipper') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="error">$restartAlert</v-icon>
        </v-list-item-icon>
      </v-list-item>

      <v-list-item
        v-if="klippyConnected"
        @click="restartKlippy(); $emit('click')"
        :disabled="printerPrinting">
        <v-list-item-title class="text-wrap">{{ $t('Restart Klipper') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="error">$restartAlert</v-icon>
        </v-list-item-icon>
      </v-list-item>

      <v-list-item
        v-if="klippyConnected"
        @click="firmwareRestartKlippy(); $emit('click')"
        :disabled="printerPrinting">
        <v-list-item-title class="text-wrap">{{ $t('Firmware Restart Klipper') }}</v-list-item-title>
        <v-list-item-icon>
          <v-icon color="error">$restartAlert</v-icon>
        </v-list-item-icon>
      </v-list-item>
    </v-list-group>

  </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import ServicesMixin from '@/mixins/services'
import { SocketActions } from '@/socketActions'
import { Device } from '@/store/devicePower/types'

@Component({})
export default class SystemCommandsWidget extends Mixins(StateMixin, ServicesMixin) {
  confirmRebootDialog = {
    open: false
  }

  confirmShutdownDialog = {
    open: false
  }

  get serverInfo () {
    return this.$store.getters['server/getInfo']
  }

  get hosted () {
    return this.$store.state.config.hostConfig.hosted
  }

  get powerDevices () {
    return this.$store.state.devicePower.devices
  }

  get devicePowerPluginEnabled () {
    return this.$store.getters['server/pluginSupport']('power')
  }

  handleHostReboot () {
    this.$confirm('Are you sure? This will reboot your host system.')
      .then(res => {
        if (res) {
          this.$emit('click')
          this.hostReboot()
        }
      })
  }

  handleHostShutdown () {
    this.$confirm('Are you sure? This will shutdown your host system.')
      .then(res => {
        if (res) {
          this.$emit('click')
          this.hostShutdown()
        }
      })
  }

  togglePowerDevice (device: Device, wait?: string) {
    const state = (device.status === 'on') ? 'off' : 'on'
    SocketActions.machineDevicePowerToggle(device.device, state, wait)
  }

  getPowerIcon (device: Device) {
    switch (device.status) {
      case 'error': {
        return '$alert'
      }
      case 'init': {
        return '$dots'
      }
      case 'on': {
        return '$powerOn'
      }
      case 'off': {
        return '$powerOff'
      }
    }
  }

  getPowerButtonText (device: Device): string {
    switch (device.status) {
      case 'error': {
        return `${device.device} [error]`
      }
      default: {
        return `${device.device}`
      }
    }
  }
}
</script>
