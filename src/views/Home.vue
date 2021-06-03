<template>
  <el-container class="home-container" v-loading.fullscreen.lock="!client.connected">
    <el-header>
      <h1>Driverbot</h1>
    </el-header>
    <el-card>
      <h1>Drive</h1>
      <el-row>
        <el-col :span="2" offset="9">
          <el-button @click="setDirection(1, -1)" :type="getTurn(1, -1)" icon="el-icon-top-left" circle></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(1, 0)" :type="getTurn(1, 0)" icon="el-icon-top" circle></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(1, 1)" :type="getTurn(1, 1)" icon="el-icon-top-right" circle></el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="2" offset="9">
          <el-button @click="setDirection(0, -1)" :type="getTurn(0, -1)" icon="el-icon-back" circle></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(0, 0)" :type="getTurn(0, 0)" icon="el-icon-close" circle></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(0, 1)" :type="getTurn(0, 1)" icon="el-icon-right" circle></el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="2" offset="9">
          <el-button
            @click="setDirection(-1, -1)"
            :type="getTurn(-1, -1)"
            icon="el-icon-bottom-left"
            circle
          ></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(-1, 0)" :type="getTurn(-1, 0)" icon="el-icon-bottom" circle></el-button>
        </el-col>
        <el-col :span="2">
          <el-button @click="setDirection(-1, 1)" :type="getTurn(-1, 1)" icon="el-icon-bottom-right" circle></el-button>
        </el-col>
      </el-row>
    </el-card>
    <el-card>
      <h1>Info</h1>
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="name"> </el-table-column>
        <el-table-column prop="value"> </el-table-column>
      </el-table>
    </el-card>
    <el-card>
      <h1>Logger</h1>
    </el-card>
    <el-footer><p>Adam Nord 2021</p></el-footer>
  </el-container>
</template>

<script>
import mqtt from 'mqtt'

export default {
  name: 'Home',

  data() {
    return {
      connection: {
        host: 'broker.hivemq.com',
        port: 8000,
        endpoint: '/mqtt',
        clean: true,
        connectTimeout: 4000,
        reconnectPeriod: 0,
        clientId: 'mqttjs_3be2c321',
        username: 'fladam',
        password: 'euYB9zYt9jDKKb6',
      },
      subscription: {
        topic: 'drive',
        qos: 0,
      },
      publish: {
        topic: 'drive',
        qos: 0,
        payload: 'ok',
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
      tableData: [
        {
          name: 'Broker',
          value: 'broker.hivemq.com',
        },
        {
          name: 'Port',
          value: '8000',
        },
        {
          name: 'Username',
          value: 'fladam',
        },
        {
          name: 'Password',
          value: 'euYB9zYt9jDKKb6',
        },
      ],
      driveDirection: 0,
      turnDirection: 0,
      driveValue: 0,
      turnValue: 90,
    }
  },
  methods: {
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
      this.client.on('error', error => {
        console.log('Connection failed', error)
      })
      this.client.on('message', (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message)
        console.log(`Received message ${message} from topic ${topic}`)
      })
    },
    doPublish() {
      const { topic, qos, payload } = this.publish
      this.client.publish(topic, payload, qos, error => {
        if (error) {
          console.log('Publish error', error)
        }
      })
    },
    setDirection(drive, turn) {
      this.driveDirection = drive
      this.turnDirection = turn
      this.updateDirection()
    },
    updateDirection() {
      let map = (value, x1, y1, x2, y2) => ((value - x1) * (y2 - x2)) / (y1 - x1) + x2
      this.driveValue = map(this.driveDirection, -1, 1, -1023, 1023)
      this.turnValue = map(this.turnDirection, -1, 1, 0, 180)
    },
    getTurn(drive, turn) {
      if (drive == this.driveDirection && turn == this.turnDirection) return 'primary'
      else return ''
    },
  },
  mounted() {
    this.$nextTick(function() {
      this.createConnection()
    })
  },
}
</script>

<style lang="scss">
@import url('../assets/style/home.scss');

.home-container {
  max-width: 800px;
  margin: 0 auto;

  .el-card {
    margin-bottom: 10px;
    &:last-child {
      margin-bottom: 0;
    }
  }

  .el-row {
    margin-bottom: 20px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  .el-col {
    text-align: center;
  }
}
</style>
