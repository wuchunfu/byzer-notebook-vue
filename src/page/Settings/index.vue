<template>
  <div class="settings-page page-outer-padding">
    <div class="page-inner-header">{{$t('menu.settings')}}</div>
    <div class="container">
      <div class="setting-item">
        <el-form ref="timeoutForm" label-position="top" :model="timeoutRuleForm" :rules="timeoutRules">
          <el-form-item :label="$t('settings.timeoutSetting')" prop="time">
            {{$t('settings.timeoutDesc1')}}
            <el-input style="width: 85px;" v-model.trim="timeoutRuleForm.time"></el-input>
            {{$t('settings.timeoutDesc2')}}
          </el-form-item>
          <div class="btns">
            <el-button type="primary" size="mini" @click="saveConfig('timeout')" :disabled="timeoutRuleForm.time === defaultTime">{{$t('save')}}</el-button>
            <el-button size="small" @click="resetForm('timeout')">{{$t('reset')}}</el-button>
          </div>
        </el-form>
      </div>
      <div class="setting-item">
        <el-form ref="engineForm" label-position="top" :model="engineRuleForm" :rules="engineRules">
          <el-form-item :label="$t('settings.defaultEngine')" prop="name">
            <p class="tip">{{$t('settings.defaultEngineDesc')}}</p>
            <el-select style="width: 240px;" v-model="engineRuleForm.name" :placeholder="$t('settings.engineSelectTip')">
              <el-option :label="engine" :value="engine" v-for="engine in engineList" :key="engine"></el-option>
            </el-select>
          </el-form-item>
          <div class="btns">
            <el-button type="primary" size="small" @click="saveConfig('engine')" :disabled="engineRuleForm.name === defaultEngine">{{$t('save')}}</el-button>
          </div>
        </el-form>
      </div>
      <div class="setting-item">
        <div class="setting-item-title">{{$t('settings.externalDatasource')}}</div>
        <div class="setting-item-btn">
          <el-button size="medium" @click="handleCreateConnection()">{{$t('add')}}</el-button>
        </div>
        <el-table
          :data="connectionList"
          style="width: 100%"
          >
          <el-table-column
            prop="staus"
            label=""
            width="60">
             <template slot-scope="scope">
              <i class="el-ksd-icon-state_16 font16'" :class="scope.row.status==='success' ? 'txt-success' : 'txt-danger'" />
            </template>
          </el-table-column>
          <el-table-column
            prop="name"
            :label="$t('settings.connectionName')"
            width="180">
          </el-table-column>
          <el-table-column
            prop="datasource"
            :label="$t('settings.dataSource')"
            width="180">
          </el-table-column>
          <el-table-column
            :label="$t('settings.action')">
            <template slot-scope="scope">
              <icon-btn icon="el-ksd-icon-edit_22" class="mr-10" :text="$t('edit')" :handler="() => handleCreateConnection(scope.row)" />
              <icon-btn icon="el-ksd-icon-delete_22" :text="$t('delete')" :handler="() => handleDeleteConnection(scope.row)" />
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'
import { Component } from 'vue-property-decorator'
import { mapActions } from 'vuex'

@Component({
  computed: {
    timeoutRules () {
      return {
        time: [{required: false, validator: this.validateTime, trigger: 'blur'}]
      }
    },
    engineRules () {
      return {
        name: [{required: false, validator: this.validateEngine, trigger: 'change'}]
      }
    }
  },
  methods: {
    ...mapActions('CreateConnectionModal', {
      callCreateConnectionModal: 'CALL_MODAL'
    }),
    ...mapActions({
      saveConfiguration: 'SAVE_CONFIG',
      getEngins: 'GET_ENGINE_LIST',
      getDefaultConfig: 'GET_CONFIGS',
      resetTimout: 'RESET_TIMEOUT',
      deleteConnection: 'DELETE_CONNECTION',
      getConnectionList: 'GET_CONNECTION_LIST'
    })
  }
})

export default class SettingsPage extends Vue {
  defaultTime = 2880
  defaultEngine = ''
  timeoutRuleForm = { time: '' }
  engineRuleForm = { name: '' }
  showReset = true
  engineList = []
  connectionList = []

  created () {
    this.getConfig()
    this.getEngineList()
    this.getConnections()
  }

  initData ({ timeout, engine }) {
    if (timeout) {
      this.defaultTime = timeout
      this.timeoutRuleForm.time = timeout
    }
    if (engine) {
      this.defaultEngine = engine
      this.engineRuleForm.name = engine
    }
  }

  async getConfig (type) {
    try {
      const res = await this.getDefaultConfig()
      if (type === 'timeout') {
        this.initData({ timeout: res.data.timeout})
      } else if (type === 'engine') {
        this.initData({ engine: res.data.engine})
      } else {
        this.initData(res.data)
      }
    } catch (e) {
      console.log(e)
    }
  }

  async getEngineList () {
    try {
      const res = await this.getEngins()
      this.engineList = res.data?.list || []
      this.engineRuleForm.name = this.engineList.length ? this.engineList[0] : ''
    } catch (e) {
      console.log(e)
    }
  }

  validateTime (rule, value, callback) {
    if (!value) {
      return callback(new Error(this.$t('settings.validTimout1')))
    } else if (!/^[0-9]+$/.test(value)) {
      return callback(new Error(this.$t('settings.validTimout1')))
    } else if (Number(value) < 1) {
      return callback(new Error(this.$t('settings.validTimout1')))
    } else {
      return callback()
    }
  }
  validateEngine (rule, value, callback) {
    if (!value) {
      return callback(new Error(this.$t('settings.engineSelectTip')))
    } else {
      return callback()
    }
  }

  async saveConfig (type) {
    const res = await this.$refs[`${type}Form`].validate()
    if (res) {
      try {
        let params = {
          timeout: this.timeoutRuleForm.time,
          engine: this.engineRuleForm.name
        }
        if (type === 'timeout') {
          params.engine = this.defaultEngine
        } else {
          params.timeout = this.defaultTime
        }
        await this.saveConfiguration(params)
        this.getConfig(type)
      } catch (e) {
        console.log(e)
      }
    }
  }
  async resetForm (type) {
    try {
      await this.resetTimout()
      this.getConfig('timeout')
      if (type === 'timeout') {
        this.$refs.timeoutForm.resetFields()
      }
    } catch (e) {
      console.log(e)
    }
  }
  async getConnections () {
    try {
      const res = await this.getConnectionList()
      this.connectionList = res.data?.connection_list ?? []
    } catch (e) {
      console.log(e)
    }
  }

  async handleCreateConnection (item) {
    const type = item ? 'edit' : 'add'
    const payload = item || {}
    if (item && (item.parameter.length === 0)) {
      item.parameter = [{name: '', value: '', key: String(Date.now())}]
    }
    const { isSubmit } = await this.callCreateConnectionModal({ payload, type })
    if (isSubmit) {
      this.getConnections()
    }
  }
  deleteConnectionConfirm () {
    return this.$confirm(this.$t('settings.deleteText'), this.$t('settings.deleteTitle'), {
      confirmButtonText: this.$t('ok'),
      cancelButtonText: this.$t('cancel'),
      type: 'warning',
      customClass: 'centerButton'
    })
  }
  async handleDeleteConnection (item) {
    try {
      await this.deleteConnectionConfirm()
      await this.deleteConnection(item.id)
      this.$message.success(this.$t('actionSuccess'))
      this.getConnections()
    } catch (e) {
      console.log(e)
    }
    
  }
}
</script>

<style lang="scss">
.settings-page {
  .setting-item {
    width: 544px;
    margin-top: 25px;
    &-title {
      font-weight: 600;
      font-size: 16px;
      line-height: 24px;
    }
    &-btn {
      padding-top: 16px;
      padding-bottom: 8px;
    }
    .tip {
      font-size: 12px;
      line-height: 16px;
      margin-bottom: 10px;
    }
  }
}
</style>