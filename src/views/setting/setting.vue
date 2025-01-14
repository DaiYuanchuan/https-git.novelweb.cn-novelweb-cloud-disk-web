<template>
  <div class="setting-panel">
    <!-- 修改用户头像 -->
    <el-card class="setting-panel-avatar box-card" shadow="always">
      <div slot="header" class="clearfix">
        <span>修改头像</span>
      </div>
      <!-- 用户头像上传组件 -->
      <div class="avatar-setting">
        <div class="form-setting">
          <div class="image-uploader__wrapper">
            <el-upload
              accept="image/*"
              class="image-uploader__content"
              :action="avatarUploader.action"
              :multiple="false"
              :drag="true"
              :with-credentials="true"
              :show-file-list="false"
              :auto-upload="true"
              :on-progress="avatarUploadProgressChange"
              :on-success="avatarUploadSuccessChange"
              :on-error="avatarUploadErrorChange"
              :on-change="avatarUploadChange"
              :before-upload="avatarBeforeUploadChange"
              :limit="1">
              <img class="image-uploader__image"
                   :src="avatarUploader.previewImage"
                   alt="username">
              <div class="image-uploader__overlay">
                <span class="image-uploader__dnd">拖拽</span>
                <span class="image-uploader__or">或</span>
                <span class="browse image-uploader-button img-item">
                  <div>浏览</div>
                </span>
              </div>
            </el-upload>
          </div>
        </div>
      </div>
    </el-card>
    <!-- 修改用户密码 -->
    <el-card class="setting-panel-password box-card" shadow="always">
      <div slot="header" class="clearfix">
        <span>重置密码</span>
      </div>
      <el-form :model="userForm" status-icon :rules="rules" ref="changePassword" label-width="100px">
        <el-form-item label="原密码" prop="sourcePassword">
          <el-input type="password" placeholder="原始密码" v-model="userForm.sourcePassword" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="新密码" prop="userPwd">
          <el-input type="password" placeholder="新密码" v-model="userForm.userPwd" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="reUserPwd">
          <el-input type="password" placeholder="确认密码" v-model="userForm.reUserPwd" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitUpdatePasswordForm('changePassword')">提交</el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <!-- 修改用户邮箱 -->
    <el-card class="setting-panel-email box-card" shadow="always">
      <div slot="header" class="clearfix">
        <span>重置邮箱</span>
      </div>
      <el-form :model="userForm" status-icon :rules="rules" ref="changeEmail" label-width="100px">
        <el-form-item label="新的邮箱" prop="userEmail">
          <el-input type="text" placeholder="请输入新的邮箱地址" v-model="userForm.userEmail" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="验证码" prop="securityCode">
          <el-input type="text" placeholder="请输入邮箱验证码" v-model="userForm.securityCode" autocomplete="off"/>
          <el-button
            @click="sendSecurityCode()"
            :disabled="!disabled"
            class="zmm"
          > {{ codeText }}
          </el-button>
        </el-form-item>
        <el-form-item class="setting-email-submit-form">
          <el-button class="setting-email-submit-btn" type="primary"
                     @click="submitUpdateEmailForm('changeEmail')">提交
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <!-- 用户容量、流量显示 -->
    <el-card class="setting-panel-capacity box-card" shadow="always">
      <div slot="header" class="clearfix">
        <span>我的容量</span>
      </div>
      <!-- 磁盘容量 -->
      <div class="content-bottom disk-capacity">
        <div class="storage-wrapper">
          <div class="usage-progress">
            <div class="text">
              <!-- 已用磁盘容量 -->
              网盘：{{ storageUnitFormatting(userInfo['userUsedDiskCapacity']) }}
              /
              <!-- 总磁盘容量 -->
              {{ storageUnitFormatting(userInfo['userTotalDiskCapacity']) }}
              <span class="expansion">
                  <a @click="$emit('openPayDialog', 0)">网盘扩容</a>
                </span>
            </div>
            <!-- 已用磁盘容量的百分比 -->
            <span class="progress disk-capacity-progress" :style="userCapacity.percentageCapacity"></span>
          </div>
        </div>
      </div>
      <!-- 流量 -->
      <div class="content-bottom traffic">
        <div class="storage-wrapper">
          <div class="usage-progress">
            <div class="text">
              <!-- 已用流量 -->
              流量：{{ storageUnitFormatting(userInfo['userUsedTraffic']) }}
              /
              <!-- 总流量 -->
              {{ storageUnitFormatting(userInfo['userTotalTraffic']) }}
              <span class="expansion">
                  <a @click="$emit('openPayDialog', 1)">流量扩容</a>
                </span>
            </div>
            <!-- 已用流量的百分比 -->
            <span class="progress disk-traffic-progress" :style="userCapacity.percentageTraffic"></span>
          </div>
        </div>
      </div>
    </el-card>
    <el-card class="setting-panel-order box-card" shadow="always">
      <div slot="header" class="clearfix">
        <span>支付订单信息</span>
      </div>
      <el-form ref="form" :model="paymentOrderSearchForm" label-width="80px" class="setting-panel-order-top-form">
        <el-select v-model="paymentOrderSearchForm.fieldSelectValue" placeholder="需要搜索的字段"
                   @change="paymentOrderSearchFormSelectChange">
          <el-option v-for="(item, index) in paymentOrderSearchForm.field"
                     :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
        <el-input
          v-show="paymentOrderSearchForm.fieldSelectValue !== 'orderTradeState' && paymentOrderSearchForm.fieldSelectValue !== 'packType'"
          type="text" placeholder="请输入需要查询的值"
          v-model="paymentOrderSearchForm.fieldSelectInputValue"></el-input>
        <el-select v-show="paymentOrderSearchForm.fieldSelectValue === 'orderTradeState'"
                   class="setting-select-order-trade-state"
                   v-model="paymentOrderSearchForm.orderTradeStateSelectValue" placeholder="请选择对应的状态">
          <el-option v-for="(item, index) in paymentOrderSearchForm.orderTradeStateSelect"
                     :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
        <el-select v-show="paymentOrderSearchForm.fieldSelectValue === 'packType'"
                   class="setting-select-order-trade-state"
                   v-model="paymentOrderSearchForm.orderPackTypeSelectValue" placeholder="请选择对应的状态">
          <el-option v-for="(item, index) in paymentOrderSearchForm.orderPackTypeSelect"
                     :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
        <el-date-picker
          v-model="paymentOrderSearchForm.time"
          type="daterange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          value-format="yyyy-MM-dd">
        </el-date-picker>
        <el-form-item class="payment-order-search-form">
          <el-button type="primary" @click="paymentOrderSearchBtn">查询</el-button>
        </el-form-item>
      </el-form>
      <el-table :data="paymentOrderInfo" style="width: 100%">
        <el-table-column prop="orderNumber" label="订单编号" min-width="240"></el-table-column>
        <el-table-column prop="orderSubject" label="订单标题" min-width="145"></el-table-column>
        <el-table-column prop="effectiveDuration" label="有效期" min-width="80"></el-table-column>
        <el-table-column prop="packType" label="资源包类型" min-width="90">
          <template slot-scope="scope">
            {{ scope.row.packType === 0 ? '扩容包' : '流量包' }}
          </template>
        </el-table-column>
        <el-table-column prop="orderTotalAmount" label="总金额" min-width="90"></el-table-column>
        <el-table-column prop="success" label="状态" min-width="90">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.success === '已支付' ? 'success' : 'info'"
              disable-transitions>{{ scope.row.success }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="orderSuccessTime" label="支付时间" min-width="185"></el-table-column>
        <el-table-column prop="createTime" label="下单时间" min-width="185"></el-table-column>
      </el-table>
      <el-pagination
        class="pagination-pc" background
        layout="sizes, prev, pager, next, ->, total"
        @size-change="paymentOrderSizeChange"
        @current-change="paymentOrderCurrentChange"
        :total="paymentOrderPagingInfo.toTal"
        :page-size="paymentOrderPagingInfo.pageSize"
        :current-page="paymentOrderPagingInfo.currentChange">
      </el-pagination>
      <el-pagination
        class="pagination-mobile" small
        layout="prev, pager, next"
        @size-change="paymentOrderSizeChange"
        @current-change="paymentOrderCurrentChange"
        :total="paymentOrderPagingInfo.toTal"
        :page-size="paymentOrderPagingInfo.pageSize"
        :current-page="paymentOrderPagingInfo.currentChange">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
import cookies from 'js-cookie'
import {updateUserInfo} from '@/api/user'
import {sendSecurityCode} from '@/api/login'
import {paymentOrderSearch} from '@/api/order'
import {validEmail, validUsername} from '@/utils/validate'
import {storageUnitConversion} from '@/utils/utils'

export default {
  name: 'setting',
  props: {
    // 路径信息的面包屑导航(数组中的第一位元素为主页)
    breadcrumbs: {
      type: Array,
      default () {
        return []
      }
    },
    // 文件列表(用来渲染的文件列表数据)
    fileList: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    // 密码校验
    const validatePassword = (rule, value, callback) => {
      if (validUsername(value)) {
        callback()
      } else {
        callback(new Error('密码 4 ~ 16位，不包含特殊字符'))
      }
    }
    // 两次密码一次性校验
    const validateRePassword = (rule, value, callback) => {
      if (!validUsername(value)) {
        callback(new Error('密码 4 ~ 16位，不包含特殊字符'))
      } else if (this.userForm.userPwd !== value) {
        callback(new Error('两次密码不相等'))
      } else {
        callback()
      }
    }
    // 邮箱校验
    const validateEmail = (rule, value, callback) => {
      if (validEmail(value)) {
        callback()
      } else {
        callback(new Error('邮箱地址不规范'))
      }
    }
    // 验证码长度校验
    const validateSecurityCode = (rule, value, callback) => {
      if (value.length === 4) {
        callback()
      } else {
        callback(new Error('验证码错误'))
      }
    }
    return {
      // 当前登录的用户对象信息
      userInfo: {},
      // 用户头像上传请求数据
      avatarUploader: {
        // 头像上传的请求地址
        action: process.env.BASE_API + '/uploader/uploader-avatar',
        // 预览的图片
        previewImage: '/static/img/default.png'
      },
      // 用户基本信息的表单对象
      userForm: {
        sourcePassword: '',
        userPwd: '',
        reUserPwd: '',
        userEmail: '',
        securityCode: ''
      },
      // 控制验证码按钮是否可用
      disabled: true,
      codeText: '获取验证码',
      // 表单验证，需要在 el-form-item 元素中增加 prop 属性
      rules: {
        sourcePassword: [{required: true, trigger: 'blur', validator: validatePassword}],
        userPwd: [{required: true, trigger: 'blur', validator: validatePassword}],
        reUserPwd: [{required: true, trigger: 'blur', validator: validateRePassword}],
        userEmail: [{required: true, trigger: 'blur', validator: validateEmail}],
        securityCode: [{required: true, trigger: 'blur', validator: validateSecurityCode}]
      },
      // 支付订单表格的分页信息
      paymentOrderPagingInfo: {
        // 总数
        toTal: 0,
        // 每页显示个数
        pageSize: 10,
        // 当前页码
        currentChange: 1
      },
      // 支付订单表格的搜索表单组件信息
      paymentOrderSearchForm: {
        // 当前被选中的值
        fieldSelectValue: 'orderNumber',
        field: [{
          label: '商户订单编号',
          value: 'orderNumber'
        }, {
          label: '交易编号',
          value: 'orderTradeNo'
        }, {
          label: '订单标题',
          value: 'orderSubject'
        }, {
          label: '交易状态',
          value: 'orderTradeState'
        }, {
          label: '资源包类型',
          value: 'packType'
        }],
        // 当前被选中状态的对应值
        fieldSelectInputValue: '',
        // 订单资源包类型被选中状态的对应值
        orderPackTypeSelectValue: 0,
        // 订单资源包类型选择器
        orderPackTypeSelect: [{
          label: '扩容包',
          value: 0
        }, {
          label: '流量包',
          value: 1
        }],
        // 订单状态选择器被选中状态的对应值
        orderTradeStateSelectValue: 'TRADE_SUCCESS',
        // 订单状态选择器
        orderTradeStateSelect: [{
          label: '已支付',
          value: 'TRADE_SUCCESS'
        }, {
          label: '已过期',
          value: 'ORDER_EXPIRED'
        }, {
          label: '待付款',
          value: ''
        }, {
          label: '已关闭',
          value: 'ORDER_CLOSED'
        }],
        // 订单交易状态
        orderTradeState: {
          'WAIT_BUYER_PAY': '待付款',
          '': '待付款',
          'TRADE_CLOSED': '已关闭',
          'ORDER_CLOSED': '已关闭',
          'TRADE_SUCCESS': '已支付',
          'TRADE_FINISHED': '交易结束',
          'ORDER_EXPIRED': '已过期'
        },
        // 时间
        time: ''
      },
      // 当前用户的支付订单信息
      paymentOrderInfo: [],
      // 用户磁盘容量、流量信息的展示
      userCapacity: {
        // 已用容量的百分比
        percentageCapacity: {},
        // 已用流量的百分比
        percentageTraffic: {}
      }
    }
  },
  // 钩子函数：页面加载完成后执行
  mounted: function () {
    // 获取cookie缓存中的用户信息
    let userInfo = cookies.get('userInfo')
    if (userInfo === undefined) {
      // 如果在未登录的情况下使用，则跳转登录页面
      this.$router.push({name: 'login'})
      return
    }
    // 重置头像预览图片
    this.avatarUploader.previewImage = JSON.parse(userInfo).userAvatar
    this.userInfo = JSON.parse(userInfo)
    this.userCapacity = {
      // 已用容量的百分比
      percentageCapacity: {
        width: ((this.userInfo['userUsedDiskCapacity'] / this.userInfo['userTotalDiskCapacity']) * 100) + '%',
        maxWidth: '100%'
      },
      percentageTraffic: {
        width: ((this.userInfo['userUsedTraffic'] / this.userInfo['userTotalTraffic']) * 100) + '%',
        maxWidth: '100%'
      }
    }
    // 获取当前用户的订单列表
    this.getPaymentOrderSearch(false)
  },
  methods: {
    /**
     * 头像上传进度的change
     * @param event
     * @param file
     */
    avatarUploadProgressChange: function (event, file) {
      console.log(event, file)
    },
    /**
     * 文件上传成功时的change
     * @param response 请求结果数据
     * @param file 当前上传成功的文件
     * @param fileList 文件列表
     */
    avatarUploadSuccessChange: function (response, file, fileList) {
      // 请求成功时返回统一状态码
      if (response.code !== '200') {
        this.$message({
          message: response.message || 'error',
          showClose: true,
          type: 'error',
          duration: 2 * 1000
        })
      } else {
        this.avatarUploader.previewImage = response.data + '?' + new Date().getTime()
        this.$emit('userAvatar')
      }
      console.log(response, file, fileList)
    },
    /**
     * 文件上传失败时的change
     * @param err
     * @param file
     * @param fileList
     */
    avatarUploadErrorChange: function (err, file, fileList) {
      console.log(err, file, fileList)
    },
    /**
     * el-upload 文件状态改变时的钩子
     * 添加文件、上传成功和上传失败时都会被调用
     * @param file
     * @param fileList
     */
    avatarUploadChange: function (file, fileList) {
      // 添加文件时
      if (file.status === 'ready') {
        // 规定文件上传大小 最大为 2 MB
        let maxFileSize = 2 * 1024 * 1024
        if (file.size > maxFileSize) {
          this.$message({
            showClose: true,
            message: '文件最大支持2MB',
            type: 'error'
          })
          return
        }

        let URL = window.URL || window.webkitURL
        this.avatarUploader.previewImage = URL.createObjectURL(file.raw)
        console.log(file, this.avatarUploader.previewImage)
      }
    },
    /**
     * 上传文件之前的钩子，参数为上传的文件，若返回 false 或者返回 Promise 且被 reject，则停止上传。
     * @param file
     */
    avatarBeforeUploadChange: function (file) {
      // 规定文件上传大小 最大为 2 MB
      let maxFileSize = 2 * 1024 * 1024
      return file.size <= maxFileSize
    },
    /**
     * 提交修改用户密码的表单数据
     * @param formName 表单名称
     */
    submitUpdatePasswordForm: function (formName) {
      // 为表单绑定验证功能
      this.$refs[formName].validate((valid) => {
        if (valid) {
          // 更新用户密码
          updateUserInfo({
            userId: this.userInfo.userId,
            userAvatar: this.userInfo.userAvatar.split('?')[0],
            userName: this.userInfo.userName,
            userEmail: this.userInfo.userEmail,
            sourcePassword: this.userForm.sourcePassword,
            userPwd: this.userForm.userPwd
          }).then((response) => {
            console.log(response)
            this.$message({
              showClose: true,
              message: '修改成功',
              type: 'success',
              duration: 2 * 1000
            })
          }).catch((err) => {
            console.log(err)
          })
        }
      })
    },
    /**
     * 提交修改用户邮箱的表单数据
     *
     * @param formName 表单名称
     */
    submitUpdateEmailForm: function (formName) {
      // 为表单绑定验证功能
      this.$refs[formName].validate((valid) => {
        if (valid) {
          // 更新用户邮箱
          updateUserInfo({
            userId: this.userInfo.userId,
            userAvatar: this.userInfo.userAvatar.split('?')[0],
            userName: this.userInfo.userName,
            userEmail: this.userForm.userEmail,
            securityCode: this.userForm.securityCode
          }).then((response) => {
            console.log(response)
            this.$message({
              showClose: true,
              message: '修改成功',
              type: 'success',
              duration: 2 * 1000
            })
          }).catch((err) => {
            console.log(err)
          })
        }
      })
    },
    /**
     * 获取邮箱验证码
     */
    sendSecurityCode: function () {
      if (validEmail(this.userForm.userEmail)) {
        // 发送验证码
        sendSecurityCode(this.userForm.userEmail, false).then(() => {
          this.$message({
            showClose: true,
            message: '验证码已发送',
            type: 'success',
            duration: 2 * 1000
          })
          // 验证码倒计时60s
          this.countdown(60)
        }).catch((err) => {
          console.log(err)
        })
      }
    },
    /**
     * 验证码倒计时方法
     *
     * @param time 时间
     */
    countdown: function (time) {
      if (time === 0) {
        this.disabled = true
        this.codeText = '重新发送'
        return
      } else {
        this.disabled = false
        this.codeText = `重新发送(${time})`
        time--
      }
      setTimeout(() => {
        this.countdown(time)
      }, 1000)
    },
    /**
     * 支付订单信息检索、获取当前用户的订单列表
     *
     * @param {Boolean} loading 是否需要加载loading面板
     */
    getPaymentOrderSearch: function (loading) {
      // 需要搜索的key值
      let key = this.paymentOrderSearchForm.fieldSelectValue
      // 需要搜索的value(默认为 input 的值)
      let value = this.paymentOrderSearchForm.fieldSelectInputValue
      if (this.paymentOrderSearchForm.fieldSelectValue === 'orderTradeState') {
        // 订单状态选中值
        value = this.paymentOrderSearchForm.orderTradeStateSelectValue
      }
      if (this.paymentOrderSearchForm.fieldSelectValue === 'packType') {
        // 资源包类型选中值
        value = this.paymentOrderSearchForm.orderPackTypeSelectValue
      }
      // 需要执行搜索的参数
      let params = {
        page: this.paymentOrderPagingInfo.currentChange,
        pageSize: this.paymentOrderPagingInfo.pageSize,
        userId: this.userInfo.userId
      }
      params[key] = value
      if (this.paymentOrderSearchForm.time !== null &&
        this.paymentOrderSearchForm.time !== '' &&
        this.paymentOrderSearchForm.time[0] !== undefined &&
        this.paymentOrderSearchForm.time[1] !== undefined) {
        params.startTime = this.paymentOrderSearchForm.time[0]
        params.endTime = this.paymentOrderSearchForm.time[1]
      }
      // 获取订单列表详情
      paymentOrderSearch(params, loading).then(response => {
        this.paymentOrderPagingInfo.toTal = response.data.toTal
        console.log(response)
        this.paymentOrderInfo = []
        this.formattedList(response.data['diskPaymentOrder'])
      }).catch((err) => {
        console.log(err)
      })
    },
    /**
     * 格式化订单列表响应信息
     * @param {Array} diskPaymentOrder 订单列表响应信息
     */
    formattedList: function (diskPaymentOrder) {
      diskPaymentOrder.forEach(res => {
        // 总的持续时间
        let totalDuration = res['packMonth'] * res['orderQuantity']
        // 有效时间
        let effectiveDuration
        // 创建时间转成的Date对象
        let endTime = new Date(res['createTime'].replace(new RegExp('-', 'gm'), '/'))
        // 重置月份为第一个月
        endTime.setMonth(0)
        // 设置新的月份 + (资源包的月份 * 数量)
        endTime.setMonth((endTime.getMonth() + totalDuration) - 1)
        let createTime = new Date(res['createTime'].replace(new RegExp('-', 'gm'), '/'))
        // 重置月份为第一个月
        createTime.setMonth(0)
        if (endTime.getFullYear() - createTime.getFullYear() > 0) {
          effectiveDuration = `${endTime.getFullYear() - createTime.getFullYear()}`
          let month = endTime.getMonth() - createTime.getMonth()
          if (month !== 11) {
            effectiveDuration = `${(totalDuration / 12).toFixed(2)}年`
          } else if (month === 11) {
            effectiveDuration = (parseInt(effectiveDuration) + 1) + '年'
          } else {
            effectiveDuration = effectiveDuration + '年'
          }
        } else {
          let month = (endTime.getMonth() - createTime.getMonth()) + 1
          effectiveDuration = month === 12 ? '1年' : `${month}个月`
        }

        if (totalDuration > 12) {
          res['orderSubject'] = res['packName'] + '-' + totalDuration + '个月'
        } else {
          res['orderSubject'] = res['packName'] + (res['packType'] === 0 ? '资源扩容包' : '流量包')
        }
        res['effectiveDuration'] = effectiveDuration
        res['orderTotalAmount'] = `￥${res['orderTotalAmount'] / 100}`
        // 支付方式: 1:微信 2:支付宝 3:系统赠送
        res['orderPaymentType'] = this.formattingOrderPaymentType(res['orderPaymentType'])
        res['success'] = this.formattingOrderTradeState(res['orderTradeState'])
        if (res['orderSuccessTime'] === null || res['orderSuccessTime'] === undefined) {
          res['orderSuccessTime'] = ''
        }
        this.paymentOrderInfo.push(res)
      })
    },
    /**
     * pageSize 改变时会触发
     * @param size 每页条数
     */
    paymentOrderSizeChange: function (size) {
      this.paymentOrderPagingInfo.pageSize = size
      // 重置当前页为第一页
      this.paymentOrderPagingInfo.currentChange = 1
      // 重置表格数据
      this.getPaymentOrderSearch(true)
    },
    /**
     * currentPage 改变时会触发
     * @param current 当前页
     */
    paymentOrderCurrentChange: function (current) {
      this.paymentOrderPagingInfo.currentChange = current
      // 重置表格数据
      this.getPaymentOrderSearch(true)
    },
    /**
     * 支付订单列表上方select选中值发生变化时触发
     * @param value 目前的选中值
     */
    paymentOrderSearchFormSelectChange: function (value) {
      console.log(this.paymentOrderSearchForm.fieldSelectValue)
    },
    /**
     * 格式化订单支付类型
     * 当前已定义的支付类型为
     * 1:微信 2:支付宝 3:系统赠送
     * @param paymentType 支付类型枚举
     */
    formattingOrderPaymentType: function (paymentType) {
      // 支付方式: 1:微信 2:支付宝 3:系统赠送
      switch (paymentType) {
        case '1':
          return '微信'
        case '2':
          return '支付宝'
        case '3':
          return '赠送'
        default:
          return '未定义'
      }
    },
    /**
     * 格式化当前订单的交易状态枚举
     * @param tradeState 交易状态枚举
     */
    formattingOrderTradeState: function (tradeState) {
      if (tradeState === '') {
        return '待付款'
      }

      // 交易状态格式化
      let state = this.paymentOrderSearchForm.orderTradeState[tradeState]
      if (state === undefined) {
        return '未定义'
      }
      return state
    },
    /**
     * 支付订单列表 搜索、查询 的按钮
     */
    paymentOrderSearchBtn: function () {
      this.paymentOrderPagingInfo.currentChange = 1
      this.getPaymentOrderSearch(true)
    },
    /**
     * 存储单位格式化
     * @param size 字节数
     */
    storageUnitFormatting: function (size) {
      if (size > 0) {
        return storageUnitConversion(size)
      }
      return '0'
    }
  }
}
</script>

<style scoped lang="scss">
.share-panel {
  display: block;
  outline: none;
  box-sizing: border-box;
  -moz-user-select: none; /*火狐*/
  -webkit-user-select: none; /*webkit浏览器*/
  -ms-user-select: none; /*IE10*/
  -khtml-user-select: none; /*早期浏览器*/
  user-select: none;
}

.zmm {
  position: absolute;
  right: 1px;
  top: 2px;
  font-size: 12px;
  padding: 12px 10px;
  border: none;
  border-left: 1px solid #dcdfe6;
  border-radius: 0;
}

/deep/ .el-upload-dragger {
  background: #373737;
  border: 0;
  width: 100%;
  height: 100%;
  position: initial;
}

.avatar-setting {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.avatar-setting .form-setting {
  width: 70%;
  max-width: 300px;
  margin: 0 auto;
}

.form-setting {
  width: 100%;
  box-sizing: border-box;
  padding: 4px 6px;
  margin: 5px 0 0;
  font-size: .933em
}

.disk-traffic-progress {
  color: #ff1d12;
}

.progress {
  height: 6px;
  position: absolute;
  bottom: 0;
  left: 0;
  transition-duration: 0.274778s;
  z-index: 1;
  background-color: rgba(28, 175, 253, 1);
  border-radius: 100px;
}

.usage-progress::after {
  content: "";
  display: block;
  position: absolute;
  bottom: 0;
  left: 0;
  background-color: rgba(222, 223, 235, .5);
  width: 100%;
  height: 6px;
  border-radius: 100px;
}

.form-setting {
  margin-top: 0;
}

.image-uploader__wrapper {
  position: relative;
  width: 100%;
  padding-bottom: 100%
}

.image-uploader__content {
  opacity: 1;
  transition: .2s opacity;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  position: absolute;
  width: 100%;
  height: 100%;
  background: #373737;
  overflow: hidden
}

.image-uploader__content:hover .image-uploader__overlay {
  opacity: 1;
  bottom: 0
}

.image-uploader__image {
  max-width: 100%;
  max-height: 100%;
}

img {
  border: none;
  vertical-align: middle;
}

.setting-panel {
  display: flex;
  flex-wrap: wrap;
}

.image-uploader__overlay {
  transition: opacity .2s, bottom .2s;
  background: #f1f1f2;
  padding: 1.4rem 2rem;
  text-align: center;
  z-index: 2;
  position: absolute;
  width: 100%;
  box-sizing: border-box;
  background: rgba(0, 0, 0, 0.6);
  color: #f1f1f2;
  opacity: 0;
  left: 0;
  bottom: -2rem
}

.setting-panel-avatar {
  width: 49%;
  margin-top: 1%;
}

.setting-select-order-trade-state {
  margin-right: 1%;
  margin-left: 1%;
  width: 30%;
}

.setting-panel-order-top-form {
  display: flex;
}

.setting-panel-order-top-form .el-input {
  width: 30%;
  margin-right: 1%;
  margin-left: 1%;
}

.setting-panel-password {
  width: 49%;
  margin-left: 1.2%;
  margin-top: 1%;
}

.setting-panel-email {
  width: 49%;
  margin-top: 1%;
  position: relative;
}

.setting-panel-capacity {
  width: 49%;
  margin-left: 1.2%;
  margin-top: 1%;
  position: relative;
}

.usage-progress {
  height: 36px;
  position: relative;
  margin-bottom: 32px;
}

.expansion {
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  user-select: none;
  color: #09AAFF;
  text-align: right;
  float: right;
}

.expansion a, .expansion a:hover {
  color: inherit;
  cursor: pointer;
  text-align: right;
  border: 0;
}

.text {
  font-size: 11px;
  line-height: 160%;
  color: var(--context_primary);
  margin-bottom: 12px;
}

.setting-panel-order {
  width: 100%;
}

/deep/ .el-form-item__content {
  text-align: right;
}

.image-uploader__dnd {
  font-size: 1.3rem;
  display: none;
  font-weight: bold;
  margin-bottom: 1.2rem
}

.item {
  padding: 18px 0;
}

/deep/ .el-table--fit {
  margin-bottom: 15px;
}

.setting-panel-order {
  margin-top: 1%;
}

@media screen and (max-width: 1350px) {

  .setting-panel-avatar {
    width: 100%;
    margin-top: 1%;
  }

  .setting-panel-password {
    width: 49%;
    margin-left: 0;
  }

  .setting-panel-email {
    width: 49%;
    margin-left: 1.5%;
  }

  .setting-panel-order {
    margin-top: 1.5%;
  }
}

@media screen and (max-width: 1120px) {

  .setting-panel-avatar {
    width: 100%;
  }

  .setting-panel-password {
    width: 100%;
    margin-left: 0;
  }

  .setting-panel-capacity {
    width: 100%;
    margin-left: 0;
  }

  .setting-panel-email {
    width: 100%;
    margin-left: 0;
    margin-top: 1.5%;
  }

  .setting-email-submit-form {
    position: relative;
    bottom: 0;
    right: 0;
  }
}

@media screen and (min-width: 1150px) {
  .image-uploader__or, .image-uploader__dnd {
    display: block
  }

  /deep/ .el-input {
    width: 96%;
  }
}

.image-uploader__or {
  font-size: 1rem;
  display: none;
  margin-bottom: 1.1rem
}

@media screen and (min-width: 1150px) {
  .image-uploader__or, .image-uploader__dnd {
    display: block
  }
}

.browse {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border-radius: 3px;
  display: -webkit-box;
  display: -moz-box;
  display: -ms-inline-flexbox;
  display: -webkit-inline-flex;
  display: inline-flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -webkit-align-items: center;
  -ms-flex-align: center;
  align-items: center;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -webkit-justify-content: center;
  -ms-flex-pack: center;
  justify-content: center;
  font-size: 13px;
  width: 100%;
  height: 2.2em;
  margin: .5em 0;
  padding: 0 1em;
  border: 0;
  font-family: "CoreSans", Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  font-weight: 700;
  color: #fff;
  fill: #fff;
  background: #94d500;
  text-transform: uppercase;
  text-align: center;
  transition: background .2s, width .2s, min-width .2s;
  min-width: 150px;
  line-height: 2em;
  box-sizing: border-box;
  cursor: pointer;
  white-space: nowrap;
  position: relative;
  text-shadow: 0 0 5px #71a200;
  vertical-align: bottom;
}

.pagination-mobile {
  display: none;
}

@media screen and (min-width: 768px) {
  .browse {
    width: auto;
    margin-left: 0;
    margin-right: 0;
  }
}

@media screen and (max-width: 736px) {
  .pagination-pc {
    display: none;
  }

  .pagination-mobile {
    display: block;
  }

  /deep/ .el-form-item__label {
    display: none;
  }

  /deep/ .el-form-item__content {
    margin-left: 0 !important;
  }

  .setting-panel-order-top-form {
    flex-wrap: wrap;
    justify-content: flex-end;
  }

  .setting-panel-order-top-form .el-input {
    margin-bottom: 15px;
    width: 100%;
  }

  /deep/ .setting-panel-order-top-form .el-input input {
    width: calc(100% + 6px);
    margin-left: -3px;
  }

  /deep/ .setting-panel-order-top-form .el-input.el-input--suffix .el-input__inner {
    width: calc(100% + 0px);
    margin-left: 0;
  }

  .el-select {
    width: 100%;
  }

  /deep/ .el-select > .el-input {
    margin-bottom: 15px;
  }

  /deep/ .el-range-editor.el-input__inner {
    display: none;
  }

  .image-uploader__overlay {
    padding: 1.4rem 0;
  }

  .browse {
    width: 50%;
    min-width: 100px;
  }
}

</style>
