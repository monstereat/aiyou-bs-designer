<template>
  <div class="wrapper">
    <el-dialog title="数据统计" :visible.sync="analyseVisible" width="400px">
      <el-table :data="analyseData" :show-header="false">
        <el-table-column property="key" />
        <el-table-column property="value" />
      </el-table>
    </el-dialog>
    <el-row>
      <el-col :span="6">
        <el-card
          :body-style="{ padding: '0px' }"
          shadow="hover"
          calss="my-add-card"
          @click.native="addNewChart"
        >
          <div class="add-card">
            <i class="el-icon-plus" />
            <div class="add-card-title">
              添加大屏
            </div>
          </div>
        </el-card>
      </el-col>
      <el-col v-for="item in chartList" :key="item._id" :span="6">
        <el-card
          :body-style="{ padding: '0px' }"
          shadow="hover"
          @click.native="editChart(item._id)"
        ><img class="image" :src="item.img">
          <div style="padding: 14px;">
            <span>{{ item.title }}</span>
            <el-dropdown
              style="float: right;"
            ><i class="el-icon-more" />
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item
                  @click.native="editChart(item._id)"
                >编辑</el-dropdown-item>
                <el-dropdown-item
                  @click.native="renameChart(item)"
                >重命名</el-dropdown-item>
                <el-dropdown-item
                  @click.native="copyChart(item)"
                >复制</el-dropdown-item>
                <el-dropdown-item
                  @click.native="deleteChart(item._id)"
                >删除</el-dropdown-item>
                <el-dropdown-item
                  divided
                  @click.native="viewChart(item._id)"
                >访问</el-dropdown-item>
                <el-dropdown-item
                  @click.native="openChartAnalyse(item)"
                >查看统计</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import {
  getChartList,
  addChartItem,
  updateChartItem,
  updateChartItemTitle,
  cloneChartItem,
  removeChartItem
} from '@/api/chart'

export default {
  props: ['user'],
  data() {
    return {
      chartList: [],
      analyseData: [],
      analyseVisible: false
    }
  },
  mounted() {
    this.getData()
  },
  methods: {
    getData() {
      console.log('this.user.uid', this.user.uid)
      getChartList(this.user.uid)
        .then(res => {
          const { errno, data } = res
          if (errno === 0) {
            this.chartList = data.chartList
          }
        })
        .catch(() => {})
    },
    editChart(id) {
      const url = window.location.protocol + '//' + window.location.host + window.location.pathname + `#/my-visual-manager/visual-editor/${id}`
      window.open(url)
    },
    addNewChart() {
      const userId = this.user.uid
      this.$prompt('输入大屏标题', '创建大屏项目', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      })
        .then(({ value }) => {
          console.log(userId, '=====', value)
          addChartItem(userId, value)
            .then(res => {
              debugger
              const { errno, data } = res
              if (errno === 0) {
                this.$message({
                  type: 'success',
                  message: '创建成功'
                })
                // this.getData();
                this.editChart(data._id)
              }
            })
            .catch(() => {})
        })
        .catch(() => {})
    },
    renameChart(row) {
      this.$prompt('输入大屏标题', '重命名', {
        inputValue: row.title,
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      })
        .then(({ value }) => {
          updateChartItemTitle(row._id, value)
            .then(res => {
              const { errno, data } = res
              if (errno === 0) {
                this.$message({
                  type: 'success',
                  message: '保存成功'
                })
                setTimeout(() => {
                  this.getData()
                }, 100)

                // this.editChart(data._id);
              }
            })
            .catch(() => {})
        })
        .catch(() => {})
    },
    copyChart(row) {
      const userId = this.user.uid
      this.$prompt('输入大屏标题', '复制大屏项目', {
        inputValue: row.title + '_复制',
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      })
        .then(({ value }) => {
          cloneChartItem(row._id, userId, value)
            .then(res => {
              const { errno, data } = res
              if (errno === 0) {
                this.$message({
                  type: 'success',
                  message: '创建成功'
                })
                this.getData()
                // this.editChart(data._id);
              }
            })
            .catch(() => {})
        })
        .catch(() => {})
    },
    deleteChart(id) {
      this.$confirm('是否删除大屏项目？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          removeChartItem(id).then(res => {
            const { errno, data } = res
            if (errno === 0) {
              this.$message({
                type: 'success',
                message: '已删除'
              })
              setTimeout(() => {
                this.getData()
              }, 100)
              // this.editChart(data._id);
            }
          })
        })
        .catch(() => {})
    },
    viewChart(id) {
      const url = window.location.protocol + '//' + window.location.host + window.location.pathname + `#/my-visual-manager/visual-view/${id}`
      window.open(url)
    },
    openChartAnalyse(row) {
      this.analyseVisible = true
      this.analyseData = [
        {
          key: '创建时间',
          value: this.$dayjs(row.createdAt).format('YYYY-MM-DD HH:mm')
        },
        {
          key: '修改时间',
          value: this.$dayjs(row.updatedAt).format('YYYY-MM-DD HH:mm')
        },
        {
          key: '访问人数',
          value: row.view
        }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  ::v-deep .el-card {
    margin: 20px;
  }
}

.page-header-slot {
  position: fixed;
  top: 56px;
  left: 240px;
  right: 0;
  padding: 0 40px 20px;
  z-index: 8;
  background-color: rgba(255, 255, 255, 0.94);

  .search {
    width: 240px;
    float: right;
  }
}

.el-card {
  margin-bottom: 36px;

  .image {
    width: 100%;
    height: 150px;
    display: block;
    opacity: 0.6;
    transition: opacity 0.3s ease;
  }

  &:hover {
    cursor: pointer;
    .image {
      opacity: 0.8;
    }
  }
}

.el-card .add-card {
  height: 198px;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  transition: all 0.5s;
  &:hover {
      background-color: #4cc9f0;
      -webkit-box-shadow: 10px 10px 99px 2px rgba(76, 201, 240, 1);
      -moz-box-shadow: 10px 10px 99px 6px rgba(76, 201, 240, 1);
      box-shadow: 10px 10px 99px 6px rgba(76, 201, 240, 1);

  }
  &:hover .el-icon-plus {
    font-size: 45px;
    color: white;
  }

  &:hover .add-card-title {
    font-size: 20px;
    color: white;
  }

  .el-icon-plus {
    font-size: 25px;
    padding-bottom: 10px;
    transition: all 0.5s;
  }

  .add-card-title {
    font-size: 16px;
    transition: all 0.5s;
  }
}
</style>
