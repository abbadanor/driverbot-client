<template>
  <div class="home-container" v-loading.fullscreen.lock="!client.connected">
    <el-button :disabled="!client.connected" type="success" size="small" class="publish-btn" @click="doPublish">
      Publish
    </el-button>
  </div>
</template>

<script>
import mqtt from 'mqtt'

export default {
  name: 'Home',

  data() {
    return {
      connection: {
        host: 'maqiatto.com',
        port: 8883,
        endpoint: '/mqtt',
        clean: true,
        connectTimeout: 4000,
        reconnectPeriod: 0,
        clientId: 'mqttjs_3be2c321',
        username: 'adam.nord@abbindustrigymnasium.se',
        password: 'password',
      },
      subscription: {
        topic: 'adam.nord@abbindustrigymnasium.se/drive',
        qos: 0,
      },
      publish: {
        topic: 'adam.nord@abbindustrigymnasium.se/drive',
        qos: 0,
        payload: 'd1023',
      },
      receiveNews: '',
      qosList: [
        { label: 0, value: 0 },
        { label: 1, value: 1 },
        { label: 2, value: 2 },
      ],
      client: {
        connected: false,
      },
      subscribeSuccess: false,
      fullScreenLoading: false,
    }
  },
  methods: {
    openFullScreen1() {
      console.log('hej')
      this.fullScreenLoading = true
      setTimeout(() => {
        this.fullScreenLoading = false
      }, 2000)
    },
    createConnection() {
      const { host, port, endpoint, ...options } = this.connection
      const connectUrl = `ws://${host}:${port}${endpoint}`
      try {
        this.client = mqtt.connect(connectUrl, options)
      } catch (error) {
        console.log('mqtt.connect error', error)
      }
      this.client.on('connect', () => {
        console.log('Connection succeeded!')
      })
      this.client.on('error', (error) => {
        console.log('Connection failed', error)
      })
      this.client.on('message', (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message)
        console.log(`Received message ${message} from topic ${topic}`)
      })
    },
    doPublish() {
      const { topic, qos, payload } = this.publish
      this.client.publish(topic, payload, qos, (error) => {
        if (error) {
          console.log('Publish error', error)
        }
      })
    },
  },
    mounted() {
    this.$nextTick(function () {
      this.createConnection()
    })
  },
}
</script>

<style lang="scss">
@import url('../assets/style/home.scss');

.home-container {
  max-width: 1100px;
  margin: 0 auto;

  .conn-btn {
    color: #fff;
    background-color: #00b173;
    font-size: 14px;
  }

  .publish-btn {
    margin-bottom: 20px;
    float: left;
  }

  .el-button--success {
    background-color: #34c388 !important;
    border-color: #34c388 !important;
    font-size: 14px !important;
  }

  .el-button--danger {
    background-color: #f5222d !important;
    border-color: #f5222d !important;
  }

  .el-form-item {
    &.is-error {
      .el-input__inner,
      .el-textarea__inner {
        box-shadow: 0 0 0 2px rgba(245, 34, 45, 0.2);
      }
    }
    &.is-success {
      .el-input__inner,
      .el-textarea__inner {
        border-color: #34c388 !important;
      }
    }
  }
}
</style>
