<template>
  <div ref="conBox" style="height: 100%">
    <div class="content-all">
      <div class="title-box">
        <!-- <el-input
          v-model.trim="content"
          suffix-icon="el-icon-search"
          placeholder="搜索内容"
          clearable
          @change="getOrderList"
          style="width: 300px; margin-left: 10px"
        ></el-input> -->
      </div>

      <!-- 表格 -->
      <div class="record-table">
        <el-table
          :data="orderDate"
          border
          size="mini"
          stripe
          :max-height="conHeight"
          @expand-change="expandChange"
          :expand-row-keys="expands"
          :row-key="getRowkeys"
          :header-cell-style="{ 'background-color': '#e3e3e3' }"
        >
          <el-table-column type="expand">
            <el-table
              class="table-style"
              size="mini"
              border
              stripe
              height="auto"
              :data="materielList"
              :max-height="conHeight"
              :header-cell-style="{ 'background-color': '#e3e3e3' }"
            >
              <el-table-column
                type="index"
                label="序号"
                width="50"
              ></el-table-column>
              <el-table-column
                property="code"
                label="物料编码"
              ></el-table-column>
              <el-table-column
                property="name"
                label="物料名称"
              ></el-table-column>
              <el-table-column
                property="potting"
                label="封装"
              ></el-table-column>
              <el-table-column property="model" label="型号"></el-table-column>
              <el-table-column property="brand" label="品牌"></el-table-column>
              <el-table-column
                property="factoryModel"
                label="厂家型号"
              ></el-table-column>
              <el-table-column property="price" label="单价"></el-table-column>
              <el-table-column
                property="supplier"
                label="供应商信息"
                show-overflow-tooltip
              ></el-table-column>
              <el-table-column
                property="remarks"
                label="描述(规格)"
                show-overflow-tooltip
              ></el-table-column>
              <!-- <el-table-column
                property="quantity"
                label="库存数量"
              ></el-table-column> -->
              <el-table-column
                label="入库数量"
                property="inNum"
              ></el-table-column>
              <el-table-column
                property="note"
                label="备注"
                show-overflow-tooltip
              ></el-table-column>
            </el-table>
          </el-table-column>
          <el-table-column prop="name" label="订单名称"></el-table-column>
          <el-table-column prop="remarks" label="订单描述"></el-table-column>
          <el-table-column prop="inTime" label="入库时间"></el-table-column>
          <el-table-column label="操作" width="180">
            <template slot-scope="scope">
              <el-tooltip
                class="item"
                effect="dark"
                offset="-20"
                content="详情"
                placement="top-start"
              >
                <el-button
                  size="mini"
                  type="primary"
                  plain
                  @click="openDialog(scope.row)"
                  >详情</el-button
                >
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <!-- 分页 -->
      <div class="block" ref="pageBox" style="margin-top: 20px; float: right">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[10, 20, 50]"
          :page-size="currPageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </div>

      <el-dialog title="订单详情" :visible.sync="orderDetail" width="500px">
        <el-form
          ref="orderForm"
          :model="orderForm"
          label-width="120px"
          class="order-form"
        >
          <el-form-item
            prop="name"
            label="订单名称"
            :rules="[
              { required: true, message: '请输入订单名称', trigger: 'blur' },
            ]"
          >
            <el-input
              readonly
              style="width: 80%"
              v-model.trim="orderForm.name"
            ></el-input>
          </el-form-item>
          <el-form-item prop="remarks" label="订单描述">
            <el-input
              readonly
              type="textarea"
              style="width: 80%"
              v-model.trim="orderForm.remarks"
            ></el-input>
          </el-form-item>
          <el-form-item prop="remarks" label="入库时间">
            <span>{{ orderForm.inTime }}</span>
          </el-form-item>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      currPageSize: 20,
      conHeight: 0,
      orderDate: [],
      currentPage: 1,
      total: 0, //总量
      content: "", //搜索框内容
      orderForm: {
        id: "",
        name: "",
        remarks: "",
        inTime: "",
      },
      orderDetail: false,
      materielList: [], //订单下物料数组
      expands: [],
    };
  },
  computed: {},
  watch: {},
  mounted() {
    this.$nextTick(() => {
      this.conHeight = this.$refs.conBox.offsetHeight - 150;
    });
    window.addEventListener("resize", () => {
      this.$nextTick(() => {
        this.conHeight = this.$refs.conBox.offsetHeight - 150;
      });
    });
  },
  methods: {
    // 分页大小改变
    handleSizeChange(val) {
      this.currPageSize = val;
      this.getOrderList();
    },
    // 分页改变
    handleCurrentChange(val) {
      this.currentPage = val;
      this.getOrderList();
    },
    //查询已完成入库订单
    getOrderList() {
      let param = {
        type: 1,
        status: 1,
        content: this.content,
        page: this.currentPage - 1,
        size: this.currPageSize,
      };
      this.orderDate = [];
      this.$get("/orderRecords/pageList", param).then((res) => {
        if (res.code == 0) {
          this.total = res.data.total;
          this.orderDate = res.data.orderRecords;
        }
      });
    },
    getOrderInfo(id) {
      let param = {
        id: id,
      };
      this.$get("/orderRecords/detail", param)
        .then((res) => {
          if (res.code == 0) {
            this.orderForm = res.data;
          }
        })
        .catch((errr) => {
          this.$notity.error("");
        });
    },
    openDialog(row) {
      this.orderDetail = true;
      this.getOrderInfo(row.id);
    },
    // getMaterielList(row) {
    //   let param = {
    //     orderId:row.id,
    //     content:"",
    //   }
    //   this.$get("/materielRecords/findCurOrder",param).then(res => {
    //     if (res.code == 0) {
    //       this.materielList = []
    //       for (let i = 0; i < res.data.materielRecords.length; i++) {
    //         let obj = res.data.materielRecords[i];
    //         this.materielList.push(obj)
    //       }
    //       console.log(this.materielList)
    //     }
    //   })
    // },
    getRowkeys(row) {
      return row.id;
    },
    expandChange(row, expandedRows) {
      this.expands = [];
      if (expandedRows.length) {
        this.materielList = [];
        if (row) {
          this.expands.push(row.id);
          this.materielList = row.materielRecords;
        }
      }
    },
  },
  created() {
    this.getOrderList();
  },
};
</script>

<style lang="less" scoped>
@table-border: #8fc7ba;

.content-all {
  height: 100%;
  overflow-y: auto;
  .record-table {
    background-color: rgba(67, 92, 112);
    width: 99%;
    // margin-top: 20px;
  }
}

.title-box {
  height: 2rem;
  position: relative;
  left: 0;
  right: 0;
  top: 0;
  // border-bottom: 1px solid @table-border;
  line-height: 1rem;
  text-align: left;
}

.order-form /deep/ .el-form-item__content {
  text-align: left;
}
</style>
