<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 订单列表数据 -->
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.row.pay_status === '0'">未付款</el-tag>
            <el-tag type="success" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
            <el-button type="success" icon="el-icon-location" size="mini" @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryIndfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryIndfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 修改地址的对话框 -->
    <el-dialog title="修改地址" :visible.sync="addressDialogVisible" width="50%" @close="addressDialogClose">
      <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
            <!-- 选择地址的级联选择框 -->
            <el-cascader v-model="addressForm.address1" :options="cityData" 
            :props="{ expandTrigger: 'hover' }" @change="handleChange">
            </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 展示物流信息的对话框 -->
    <el-dialog title="物流信息" :visible.sync="progressDialogVisible" width="50%">
      <!-- 时间线 -->
       <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressInfo"
          :key="index" :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
  import cityData from './citydata.js'

  export default {
    data() {
      return {
        // 订单列表信息
        queryIndfo: {
          query: '',
          pagenum: 1,
          pagesize: 5
        },
        // 订单总数
        total: 0,
        // 订单列表
        orderlist: [],
        // 展示修改地址对话框的显示与隐藏
        addressDialogVisible: false,
        // 修改地址对话框的表单对象
        addressForm: {
          address1: [],
          address2: ''
        },
        // 修改地址对话框的表单验证规则
        addressFormRules: {
          address1: [
            { required:'ture', message: '请选择省市区/县', trigger: 'blur' }
          ],
          address2: [
            { required:'ture', message: '请填写详细地址', trigger: 'blur' }
          ]
        },
        cityData,
        // 展示物流信息对话框的显示与隐藏
        progressDialogVisible: false,
        // 物流信息
        progressInfo: []
      }
    },
    created() {
      this.getOrderList()
    },
    methods: {
      // 获取订单列表
      async getOrderList() {
        const { data:res } = await this.$http.get('orders', { params: this.queryIndfo })

        if(res.meta.status !== 200) {
          return this.$message.error('获取订单列表失败')
        }

        // console.log(res);
        this.orderlist = res.data.goods
        this.total = res.data.total
      },
      // 分页 刷新每页条数
      handleSizeChange(newSize) {
        this.queryIndfo.pagesize = newSize
        this.getOrderList()
      },
      // 分页 刷新页码
      handleCurrentChange(newPage) {
        this.queryIndfo.pagenum = newPage
        this.getOrderList()
      },
      // 展示修改地址的对话框
      showBox() {
        this.addressDialogVisible = true
      },
      handleChange() {

      },
      // 监听修改地址对话框的关闭事件
      addressDialogClose() {
        this.$refs.addressFormRef.resetFields()
      },
      // 展示物流信息对话框的显示与隐藏
      async showProgressBox() {
        // const { data:res } = await this.$http.get('order/1106975712662')

        // if(res.meta.status !== 200) {
        //   return this.$message.error('获取物流信息失败')
        // }

        // this.progressInfo = res.data

        // this.progressDialogVisible = true
        // console.log(this.progressInfo);
        // console.log('123');
      }
    }
  }
</script>

<style lang="less" scoped>
@import '../../plugins/timeline/timeline.css';
@import '../../plugins/timeline-item/timeline-item.css';

.el-cascader{
  width: 100%;
}
</style>