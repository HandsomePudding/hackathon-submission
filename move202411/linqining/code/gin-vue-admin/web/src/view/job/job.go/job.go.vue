
<template>
  <div>
    <div class="gva-search-box">
      <el-form ref="elSearchFormRef" :inline="true" :model="searchInfo" class="demo-form-inline" :rules="searchRule" @keyup.enter="onSubmit">

        <template v-if="showAllQuery">
          <!-- 将需要控制显示状态的查询条件添加到此范围内 -->
          <el-form-item label="薪资下限" prop="salaryBottom">

            <el-input v-model.number="searchInfo.salaryBottom" placeholder="搜索条件" />
          </el-form-item>
          <el-form-item label="薪资上限" prop="salaryBottom">
            <el-input v-model.number="searchInfo.salaryCeil" placeholder="搜索条件" />
          </el-form-item>
        </template>

        <el-form-item>
          <el-button type="primary" icon="search" @click="onSubmit">查询</el-button>
          <el-button icon="refresh" @click="onReset">重置</el-button>
          <el-button link type="primary" icon="arrow-down" @click="showAllQuery=true" v-if="!showAllQuery">展开</el-button>
          <el-button link type="primary" icon="arrow-up" @click="showAllQuery=false" v-else>收起</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="gva-table-box">
        <div class="gva-btn-list">
            <el-button  type="primary" icon="plus" @click="openDialog()">新增</el-button>
            <el-button  icon="delete" style="margin-left: 10px;" :disabled="!multipleSelection.length" @click="onDelete">删除</el-button>
            
        </div>
        <el-table
        ref="multipleTable"
        style="width: 100%"
        tooltip-effect="dark"
        :data="tableData"
        row-key="id"
        @selection-change="handleSelectionChange"
        >
        <el-table-column type="selection" width="55" />
          <el-table-column align="left" label="id" prop="id" width="120" />
          <el-table-column align="left" label="公司id" prop="companyId" width="120" />
          <el-table-column align="left" label="标题" prop="title" width="120" />
          <el-table-column align="left" label="职位描述" prop="description" width="120" />
          <el-table-column align="left" label="薪资下限" prop="salaryBottom" width="120" />
          <el-table-column align="left" label="薪资上限" prop="salaryCeil" width="120" />
          <el-table-column align="left" label="图片"  width="120" v-slot="scope">
            <div v-if="scope.row.mediaType && scope.row.mediaType.startsWith('image')">
<!--              <article>-->
<!--                <object :data="`https://aggregator.walrus-testnet.walrus.space/v1/`+scope.row.blobId" :type="scope.row.mediaType"/>-->
<!--              </article>-->
              <el-image :src="`https://aggregator.walrus-testnet.walrus.space/v1/`+scope.row.blobId"></el-image>
            </div>
            <div v-else>
              {{scope.row.blobId}}
            </div>
          </el-table-column>
          <el-table-column align="left" label="WalrusBlobId" prop="blobId" width="120" ></el-table-column>
          <el-table-column align="left" label="SuiObjectID" prop="objectID" width="120" />
          <el-table-column align="left" label="mediatype" prop="mediaType" width="120" />

          <el-table-column align="left" label="操作" fixed="right" :min-width="appStore.operateMinWith">
            <template #default="scope">
            <el-button  type="primary" link class="table-button" @click="getDetails(scope.row)"><el-icon style="margin-right: 5px"><InfoFilled /></el-icon>查看</el-button>
            <el-button  type="primary" link icon="edit" class="table-button" @click="updateJobFunc(scope.row)">编辑</el-button>
            <el-button   type="primary" link icon="delete" @click="deleteRow(scope.row)">删除</el-button>
            </template>
        </el-table-column>
        </el-table>
        <div class="gva-pagination">
            <el-pagination
            layout="total, sizes, prev, pager, next, jumper"
            :current-page="page"
            :page-size="pageSize"
            :page-sizes="[10, 30, 50, 100]"
            :total="total"
            @current-change="handleCurrentChange"
            @size-change="handleSizeChange"
            />
        </div>
    </div>
    <el-drawer destroy-on-close :size="appStore.drawerSize" v-model="dialogFormVisible" :show-close="false" :before-close="closeDialog">
       <template #header>
              <div class="flex justify-between items-center">
                <span class="text-lg">{{type==='create'?'新增':'编辑'}}</span>
                <div>
                  <el-button :loading="btnLoading" type="primary" @click="enterDialog">确 定</el-button>
                  <el-button @click="closeDialog">取 消</el-button>
                </div>
              </div>
            </template>

          <el-form :model="formData" label-position="top" ref="elFormRef" :rules="rule" label-width="80px">
            <el-form-item label="公司id:"  prop="companyId" >
              <el-input v-model.number="formData.companyId" :clearable="true" placeholder="请输入公司id" />
            </el-form-item>
            <el-form-item label="标题:"  prop="title" >
              <el-input v-model="formData.title" :clearable="true"  placeholder="请输入标题" />
            </el-form-item>
            <el-form-item label="职位描述:"  prop="description" >
              <el-input v-model="formData.description" :clearable="true" placeholder="请输入职位描述" />
            </el-form-item>
            <el-form-item label="薪资下限:"  prop="salaryBottom" >
              <el-input v-model.number="formData.salaryBottom" :clearable="true" placeholder="请输入薪资下限" />
            </el-form-item>
            <el-form-item label="薪资上限:"  prop="salaryCeil" >
              <el-input v-model.number="formData.salaryCeil" :clearable="true" placeholder="请输入薪资上限" />
            </el-form-item>

            <el-form-item label="SuiObjectID:"  prop="objectID" >
              <el-input v-model="formData.objectID" :clearable="true"  disabled />
            </el-form-item>
            <span>职位详情:</span>
            <el-upload
                ref="uploadFile"
                v-model:file-list="fileList"
                class="upload-demo"
                action="https://publisher.walrus-testnet.walrus.space/v1/store?epochs=1"
                method="PUT"
                :auto-upload="false"
            >
              <template #trigger>
                <el-button type="primary">选择文件</el-button>
              </template>
              <el-button class="ml-3" type="success" @click="submitUpload">
                上传文件
              </el-button>
              <template #tip>
                <div class="el-upload__tip">
                  不超过10m.
                </div>
              </template>
            </el-upload>

          </el-form>
    </el-drawer>

    <el-drawer destroy-on-close :size="appStore.drawerSize" v-model="detailShow" :show-close="true" :before-close="closeDetailShow" title="查看">
            <el-descriptions :column="1" border>
              <el-descriptions-item label="id">
                {{ detailFrom.id }}
              </el-descriptions-item>
                    <el-descriptions-item label="公司id">
                        {{ detailFrom.companyId }}
                    </el-descriptions-item>
              <el-descriptions-item label="标题">
                {{ detailFrom.title }}
              </el-descriptions-item>
                    <el-descriptions-item label="职位描述">
                        {{ detailFrom.description }}
                    </el-descriptions-item>
                    <el-descriptions-item label="薪资下限">
                      {{ detailFrom.salaryBottom }}
                    </el-descriptions-item>
                    <el-descriptions-item label="薪资上限">
                        {{ detailFrom.salaryCeil }}
                    </el-descriptions-item>
                    <el-descriptions-item label="WalrusBlobId">
                        {{ detailFrom.blobId }}
                    </el-descriptions-item>

              <el-descriptions-item label="SuiObjectID">
                {{ detailFrom.objectID }}
              </el-descriptions-item>
              <el-descriptions-item label="mediatype">
                {{ detailFrom.mediaType }}
              </el-descriptions-item>
            </el-descriptions>
        </el-drawer>

  </div>
</template>

<script setup>
import {
  createJob,
  deleteJob,
  deleteJobByIds,
  updateJob,
  findJob,
  getJobList
} from '@/api/job/job.go'

// 全量引入格式化工具 请按需保留
import { getDictFunc, formatDate, formatBoolean, filterDict ,filterDataSource, returnArrImg, onDownloadFile } from '@/utils/format'
import { ElMessage, ElMessageBox ,ElImage} from 'element-plus'
import { ref, reactive } from 'vue'
import { useAppStore } from "@/pinia"




defineOptions({
    name: 'Job'
})

// 提交按钮loading
const btnLoading = ref(false)
const appStore = useAppStore()

// 控制更多查询条件显示/隐藏状态
const showAllQuery = ref(false)

// 自动化生成的字典（可能为空）以及字段
const formData = ref({
            companyId: undefined,
            description: '',
            salaryCeil: undefined,
            blobId: '',
            title: '',
            salaryBottom: undefined,
            objectID: '',
            mediaType: '',
        })



// 验证规则
const rule = reactive({
})

const searchRule = reactive({
  createdAt: [
    { validator: (rule, value, callback) => {
      if (searchInfo.value.startCreatedAt && !searchInfo.value.endCreatedAt) {
        callback(new Error('请填写结束日期'))
      } else if (!searchInfo.value.startCreatedAt && searchInfo.value.endCreatedAt) {
        callback(new Error('请填写开始日期'))
      } else if (searchInfo.value.startCreatedAt && searchInfo.value.endCreatedAt && (searchInfo.value.startCreatedAt.getTime() === searchInfo.value.endCreatedAt.getTime() || searchInfo.value.startCreatedAt.getTime() > searchInfo.value.endCreatedAt.getTime())) {
        callback(new Error('开始日期应当早于结束日期'))
      } else {
        callback()
      }
    }, trigger: 'change' }
  ],
})

const elFormRef = ref()
const elSearchFormRef = ref()

// =========== 表格控制部分 ===========
const page = ref(1)
const total = ref(0)
const pageSize = ref(10)
const tableData = ref([])
const searchInfo = ref({})
// 重置
const onReset = () => {
  searchInfo.value = {}
  getTableData()
}

// 搜索
const onSubmit = () => {
  elSearchFormRef.value?.validate(async(valid) => {
    if (!valid) return
    page.value = 1
    getTableData()
  })
}

// 分页
const handleSizeChange = (val) => {
  pageSize.value = val
  getTableData()
}

// 修改页面容量
const handleCurrentChange = (val) => {
  page.value = val
  getTableData()
}

// 查询
const getTableData = async() => {
  const table = await getJobList({ page: page.value, pageSize: pageSize.value, ...searchInfo.value })
  if (table.code === 0) {
    console.log("resp",table.data)
    tableData.value = table.data.list
    total.value = table.data.total
    page.value = table.data.page
    pageSize.value = table.data.pageSize
  }
}

getTableData()

// ============== 表格控制部分结束 ===============

// 获取需要的字典 可能为空 按需保留
const setOptions = async () =>{
}

// 获取需要的字典 可能为空 按需保留
setOptions()


// 多选数据
const multipleSelection = ref([])
// 多选
const handleSelectionChange = (val) => {
    multipleSelection.value = val
}

// 删除行
const deleteRow = (row) => {
    ElMessageBox.confirm('确定要删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
    }).then(() => {
            deleteJobFunc(row)
        })
    }

// 多选删除
const onDelete = async() => {
  ElMessageBox.confirm('确定要删除吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async() => {
      const ids = []
      if (multipleSelection.value.length === 0) {
        ElMessage({
          type: 'warning',
          message: '请选择要删除的数据'
        })
        return
      }
      multipleSelection.value &&
        multipleSelection.value.map(item => {
          ids.push(item.id)
        })
      const res = await deleteJobByIds({ ids })
      if (res.code === 0) {
        ElMessage({
          type: 'success',
          message: '删除成功'
        })
        if (tableData.value.length === ids.length && page.value > 1) {
          page.value--
        }
        getTableData()
      }
      })
    }

// 行为控制标记（弹窗内部需要增还是改）
const type = ref('')

// 更新行
const updateJobFunc = async(row) => {
    const res = await findJob({ id: row.id })
    type.value = 'update'
    if (res.code === 0) {
        formData.value = res.data
        dialogFormVisible.value = true
    }
}


// 删除行
const deleteJobFunc = async (row) => {
    const res = await deleteJob({ id: row.id })
    if (res.code === 0) {
        ElMessage({
                type: 'success',
                message: '删除成功'
            })
            if (tableData.value.length === 1 && page.value > 1) {
            page.value--
        }
        getTableData()
    }
}

// 弹窗控制标记
const dialogFormVisible = ref(false)

// 打开弹窗
const openDialog = () => {
    type.value = 'create'
    dialogFormVisible.value = true
}

// 关闭弹窗
const closeDialog = () => {
    dialogFormVisible.value = false
    formData.value = {
        companyId: undefined,
        description: '',
        salaryBottom: undefined,
        salaryCeil: undefined,
        blobId: '',
        mediaType: '',
        title: '',
    }
}
// 弹窗确定
const enterDialog = async () => {
     btnLoading.value = true
     elFormRef.value?.validate( async (valid) => {
             if (!valid) return btnLoading.value = false
              let res
              switch (type.value) {
                case 'create':
                  res = await createJob(formData.value)
                  break
                case 'update':
                  res = await updateJob(formData.value)
                  break
                default:
                  res = await createJob(formData.value)
                  break
              }
              btnLoading.value = false
              if (res.code === 0) {
                ElMessage({
                  type: 'success',
                  message: '创建/更改成功'
                })
                closeDialog()
                getTableData()
              }
      })
}


const detailFrom = ref({})

// 查看详情控制标记
const detailShow = ref(false)


// 打开详情弹窗
const openDetailShow = () => {
  detailShow.value = true
}


// 打开详情
const getDetails = async (row) => {
  // 打开弹窗
  const res = await findJob({ id: row.id })
  if (res.code === 0) {
    detailFrom.value = res.data
    openDetailShow()
  }
}


// 关闭详情弹窗
const closeDetailShow = () => {
  detailShow.value = false
  detailFrom.value = {}
}

const uploadFile = ref()
const fileList = ref([])

import {storeBlob} from "@/api/walrus"
const submitUpload=()=>{
  console.log("uploadfile",uploadFile.value)
  console.log("filelist",fileList.value)
  if (fileList.value.length==0){
    return
  }
  storeBlob(fileList.value[0].raw).then((resp)=>{
    const storage_info = resp.info
    const media_type = resp.media_type
    console.log("upload resp",resp)
    var info = {};
  const SUI_NETWORK = "testnet";
  const SUI_VIEW_TX_URL = `https://suiscan.xyz/${SUI_NETWORK}/tx`;
  const SUI_VIEW_OBJECT_URL = `https://suiscan.xyz/${SUI_NETWORK}/object`;
    if ("alreadyCertified" in storage_info) {
        info = {
            status: "Already certified",
            blobId: storage_info.alreadyCertified.blobId,
            endEpoch: storage_info.alreadyCertified.endEpoch,
            suiRefType: "Previous Sui Certified Event",
            suiRef: storage_info.alreadyCertified.eventOrObject.Event.txDigest,
            suiBaseUrl: SUI_VIEW_TX_URL,
        };
    } else if ("newlyCreated" in storage_info) {
      info = {
        status: "Newly created",
        blobId: storage_info.newlyCreated.blobObject.blobId,
        endEpoch: storage_info.newlyCreated.blobObject.storage.endEpoch,
        suiRefType: "Associated Sui Object",
        suiRef: storage_info.newlyCreated.blobObject.id,
        suiBaseUrl: SUI_VIEW_OBJECT_URL,
      };
    }
    console.log(storage_info)
    console.log(info)
    console.log("upload file",fileList.value[0])
    formData.value.blobId = info.blobId
    formData.value.objectID = info.suiRef
    formData.value.mediaType = media_type
  })
}

</script>


