<template>
  <!-- <div class="title-card-container2">
    <div style="flex-grow: 1">
      <PiXiuYaml :refresh="getNamespaces" title="命名空间" :display-namespace="false"></PiXiuYaml>
    </div>
  </div> -->
  <Description
    :description="'Namespace 提供一种机制，将同一集群中的资源划分为相互隔离的组。同一名字空间内的资源名称要唯一。'"
  />
  <div style="margin-top: 5px">
    <el-row>
      <el-col>
        <button class="pixiu-two-button" @click="createNamespace">新建</button>
        <button class="pixiu-two-button2" style="margin-left: 10px" @click="getNamespaces">
          刷新
        </button>

        <el-input
          v-model="data.pageInfo.search.searchInfo"
          placeholder="名称搜索关键字"
          style="width: 480px; float: right"
          clearable
          @clear="getNamespace"
          @input="searchNamespace"
        >
          <template #suffix>
            <el-icon class="el-input__icon" @click="getNamespaces">
              <component :is="'Search'" />
            </el-icon>
          </template>
        </el-input>
      </el-col>
    </el-row>
    <el-card class="box-card">
      <el-table
        v-loading="data.loading"
        :data="data.tableData"
        stripe
        style="margin-top: 2px; width: 100%"
        :cell-style="{
          'font-size': '12px',
          color: '#191919',
        }"
        header-row-class-name="pixiu-table-header"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="30" />
        <el-table-column prop="metadata.name" sortable label="命名空间">
          <template #default="scope">
            <el-link
              class="global-table-world"
              type="primary"
              @click="jumpNamespaceRoute(scope.row)"
            >
              {{ scope.row.metadata.name }}
            </el-link>
          </template>
        </el-table-column>

        <el-table-column label="状态" prop="status" sortable :formatter="formatStatus">
        </el-table-column>

        <el-table-column
          prop="metadata.labels"
          label="Labels"
          sortable
          :formatter="formatterLabelsBackup2"
          width="380px"
        />

        <el-table-column
          label="创建时间"
          prop="metadata.creationTimestamp"
          sortable
          :formatter="formatterTime"
        >
        </el-table-column>

        <el-table-column label="描述" prop="-"> <span>-</span> </el-table-column>

        <el-table-column fixed="right" label="操作" width="200px">
          <template #default="scope">
            <el-button
              size="small"
              type="text"
              style="margin-right: -25px; margin-left: -10px; color: #006eff"
              @click="handleQuotaDialog(scope.row)"
            >
              管理配额
            </el-button>

            <el-button
              type="text"
              size="small"
              style="margin-right: -2px; color: #006eff"
              @click="handleDeleteDialog(scope.row)"
            >
              删除
            </el-button>

            <el-dropdown>
              <span class="cluster-dropdown">
                更多
                <div style="margin-left: 2px"></div>
                <pixiu-icon name="icon-xiala" size="12px" type="iconfont" color="#006eff" />
              </span>
              <template #dropdown>
                <el-dropdown-menu class="dropdown-buttons">
                  <el-dropdown-item class="dropdown-item-buttons" @click="editYaml(scope.row)">
                    编辑YAML
                  </el-dropdown-item>
                  <el-dropdown-item class="dropdown-item-buttons"> 强制删除 </el-dropdown-item>
                </el-dropdown-menu>
              </template>
            </el-dropdown>
          </template>
        </el-table-column>

        <template #empty>
          <div class="table-inline-word">命名空间的列表为空</div>
        </template>
      </el-table>
      <pagination :total="data.pageInfo.total" @on-change="onChange"></pagination>
    </el-card>
  </div>

  <pixiuDialog
    :close-event="data.deleteDialog.close"
    :object-name="data.deleteDialog.objectName"
    :delete-name="data.deleteDialog.deleteName"
    @confirm="confirm"
    @cancel="cancel"
  ></pixiuDialog>

  <el-dialog
    :model-value="data.quotaData.close"
    style="color: #191919; font: 14px"
    width="42%"
    align-center
    center
    @close="cleanQuota"
  >
    <template #header>
      <div class="header-docs">配额管理</div>
    </template>

    <el-card class="app-docs" style="margin-top: -0px; height: 40px; margin-left: 5px">
      <el-icon
        style="vertical-align: middle; font-size: 16px; margin-left: -25px; margin-top: -50px"
        ><WarningFilled
      /></el-icon>
      <div style="vertical-align: middle; margin-top: -40px">
        配额可以限制命名空间下的资源使用，支持以命名空间为粒度的资源划分。
      </div>
    </el-card>
    <div style="margin-top: -10px" />

    <el-table
      :data="data.quotaData.data"
      stripe
      style="margin-top: 2px; margin-left: 8px"
      header-row-class-name="pixiu-table-header"
      :cell-style="{
        'font-size': '12px',
        color: '#191919',
      }"
    >
      <el-table-column prop="name" label="应用资源">
        <template #default="scope"> {{ scope.row.name }} </template>
      </el-table-column>
      <el-table-column prop="value" label="配额">
        <template #default="scope">
          <el-input-number v-model="scope.row.value" size="small" :min="0" style="width: 115px" />
        </template>
      </el-table-column>
    </el-table>

    <template #footer>
      <span class="dialog-footer">
        <el-button class="pixiu-delete-cancel-button" @click="cancelQuota">取消</el-button>
        <el-button type="primary" class="pixiu-delete-confirm-button" @click="confirmQuota"
          >确认</el-button
        >
      </span>
      <div style="margin-bottom: 10px" />
    </template>
  </el-dialog>

  <PiXiuViewOrEdit
    :yaml-dialog="data.editYamlDialog"
    title="编辑Yaml"
    :yaml="data.yaml"
    :read-only="false"
    :confirm="confirmEditYaml"
  ></PiXiuViewOrEdit>
</template>

<script setup lang="jsx">
import { useRouter } from 'vue-router';
import { reactive, getCurrentInstance, onMounted } from 'vue';
import {
  getNamespaceList,
  getNamespace,
  deleteNamespace,
  getQuotaList,
  patchQuota,
  deleteQuota,
  createQuota,
} from '@/services/kubernetes/namespaceService';
import { getTableData, searchData } from '@/utils/utils';
import Description from '@/components/description/index.vue';
import Pagination from '@/components/pagination/index.vue';
import pixiuDialog from '@/components/pixiuDialog/index.vue';
import { formatterTime, formatterIcon, formatterLabelsBackup2 } from '@/utils/formatter';
import PiXiuViewOrEdit from '@/components/pixiuyaml/viewOrEdit/index.vue';

const { proxy } = getCurrentInstance();
const router = useRouter();

const data = reactive({
  cluster: '',
  pageInfo: {
    page: 1,
    total: 0,
    limit: 10,
    search: {
      field: 'name',
      searchInfo: '',
    },
  },
  tableData: [],
  loading: false,
  editYamlDialog: false,
  yaml: '',
  namespaceList: [],

  // 删除对象属性
  deleteDialog: {
    close: false,
    objectName: '命名空间',
    deleteName: '',
  },

  quotaForm: {
    metadata: {
      name: '',
      namespace: '',
    },
    spec: {
      hard: {},
    },
  },
  quotaData: {
    close: false,
    exists: false,
    needDelete: true,
    name: '',
    namespaceName: '',
    oldSpec: {},
    data: [
      {
        name: 'CPU(核)',
        value: null,
      },
      {
        name: '内存(GiB)',
        value: null,
      },
      {
        name: '无状态负载 Deployment',
        value: null,
      },
      {
        name: '有状态负载 StatefulSet',
        value: null,
      },
      {
        name: '普通任务 Job',
        value: null,
      },
      {
        name: '定时任务 CronJob',
        value: null,
      },
      {
        name: '容器组 Pod',
        value: null,
      },
    ],
  },
});

onMounted(() => {
  data.cluster = proxy.$route.query.cluster;

  getNamespaces();
});

const handleQuotaDialog = async (row) => {
  const [quotas, err] = await getQuotaList(data.cluster, row.metadata.name);
  if (err) {
    proxy.$notify.error(err.response.data.message);
    return;
  }

  if (quotas.items.length !== 0) {
    data.quotaData.exists = true;
    const quota = quotas.items[0];
    data.quotaData.name = quota.name;
    data.quotaData.oldSpec = quota.spec;
    initQuotaData(quota);
  }

  data.quotaData.namespaceName = row.metadata.name;
  data.quotaData.close = true;
};

const cancelQuota = () => {
  data.quotaData.close = false;
  setTimeout(() => {
    cleanQuota();
  }, 100);
};

const initQuotaData = (quota) => {
  const hard = quota.spec.hard;
  for (let d of data.quotaData.data) {
    if (d.name === 'CPU(核)') {
      if (hard['limits.cpu']) {
        d.value = hard['limits.cpu'];
      }
    }
    if (d.name === '内存(GiB)') {
      if (hard['limits.memory']) {
        if (hard['limits.memory'].includes('Gi')) {
          d.value = hard['limits.memory'].split('Gi')[0];
        } else {
          d.value = hard['limits.memory'];
        }
      }
    }
    if (d.name === '无状态负载 Deployment') {
      if (hard['count/deployments.apps']) {
        d.value = hard['count/deployments.apps'];
      }
    }
    if (d.name === '有状态负载 StatefulSet') {
      if (hard['count/statefulsets.apps']) {
        d.value = hard['count/statefulsets.apps'];
      }
    }
    if (d.name === '普通任务 Job') {
      if (hard['count/jobs.batch']) {
        d.value = hard['count/jobs.batch'];
      }
    }
    if (d.name === '定时任务 CronJob') {
      if (hard['count/cronjobs.batch']) {
        d.value = hard['count/cronjobs.batch'];
      }
    }
    if (d.name === '容器组 Pod') {
      if (hard['count/pods']) {
        d.value = hard['count/pods'];
      }
    }
  }
};

const getQuotaNameForNamespace = (ns) => {
  return 'pixiu-' + ns + '-quota';
};

const initQuotaForm = (quotaData) => {
  data.quotaForm.metadata.name = getQuotaNameForNamespace(quotaData.namespaceName);
  data.quotaForm.metadata.namespace = quotaData.namespaceName;

  // 默认设置成true，根据直接情况修改
  data.quotaData.needDelete = true;
  let newHard = {};

  for (let d of quotaData.data) {
    if (d.name === 'CPU(核)') {
      if (d.value !== null && d.value !== 0) {
        newHard['limits.cpu'] = d.value;
        data.quotaData.needDelete = false;
      }
    }
    if (d.name === '内存(GiB)') {
      if (d.value !== null && d.value !== 0) {
        newHard['limits.memory'] = d.value + 'Gi';
        data.quotaData.needDelete = false;
      }
    }
    if (d.name === '无状态负载 Deployment') {
      if (d.value !== null && d.value !== 0) {
        newHard['count/deployments.apps'] = d.value;
        data.quotaData.needDelete = false;
      }
    }

    if (d.name === '有状态负载 StatefulSet') {
      if (d.value !== null && d.value !== 0) {
        newHard['count/statefulsets.apps'] = d.value;
        data.quotaData.needDelete = false;
      }
    }
    if (d.name === '普通任务 Job') {
      if (d.value !== null && d.value !== 0) {
        newHard['count/jobs.batch'] = d.value;
        data.quotaData.needDelete = false;
      }
    }
    if (d.name === '定时任务 CronJob') {
      if (d.value !== null && d.value !== 0) {
        newHard['count/cronjobs.batch'] = d.value;
        data.quotaData.needDelete = false;
      }
    }
    if (d.name === '容器组 Pod') {
      if (d.value !== null && d.value !== 0) {
        newHard['count/pods'] = d.value;
        data.quotaData.needDelete = false;
      }
    }
  }
  data.quotaForm.spec.hard = newHard;
};

const cleanQuota = () => {
  data.quotaData = {
    close: false,
    exists: false,
    needDelete: true,
    oldSpec: {},
    namespaceName: '',
    data: [
      {
        name: 'CPU(核)',
        value: null,
      },
      {
        name: '内存(GiB)',
        value: null,
      },
      {
        name: '无状态负载 Deployment',
        value: null,
      },
      {
        name: '有状态负载 StatefulSet',
        value: null,
      },
      {
        name: '普通任务 Job',
        value: null,
      },
      {
        name: '定时任务 CronJob',
        value: null,
      },
      {
        name: '容器组 Pod',
        value: null,
      },
    ],
  };
  data.quotaForm = {
    metadata: {
      name: '',
      namespace: '',
    },
    spec: {
      hard: {},
    },
  };
};

const confirmQuota = async () => {
  initQuotaForm(data.quotaData);

  // 更新
  if (data.quotaData.exists) {
    if (data.quotaData.needDelete) {
      // 删除
      const [result, err] = await deleteQuota(
        data.cluster,
        data.quotaForm.metadata.namespace,
        data.quotaForm.metadata.name,
      );
      if (err) {
        proxy.$notify.error(err.response.data.message);
        return;
      }
    } else {
      // 更新
      for (let key in data.quotaData.oldSpec.hard) {
        if (!data.quotaForm.spec.hard[key]) {
          data.quotaForm.spec.hard[key] = null;
        }
      }

      // 更新最新属性，移除老的值
      const patchData = {
        spec: {
          hard: data.quotaForm.spec.hard,
        },
      };
      const [result, err] = await patchQuota(
        data.cluster,
        data.quotaForm.metadata.namespace,
        data.quotaForm.metadata.name,
        patchData,
      );
      if (err) {
        proxy.$notify.error(err.response.data.message);
        return;
      }
    }
  } else {
    if (data.quotaData.needDelete) {
      cleanQuota();
      return;
    } else {
      const [result, err] = await createQuota(
        data.cluster,
        data.quotaForm.metadata.namespace,
        data.quotaForm,
      );
      if (err) {
        proxy.$notify.error(err.response.data.message);
        return;
      }
    }
  }

  proxy.$notify.success(`命名空间配额设置成功`);
  cleanQuota();
};

const handleDeleteDialog = (row) => {
  data.deleteDialog.close = true;
  data.deleteDialog.deleteName = row.metadata.name;
};

const confirm = async () => {
  const [result, err] = await deleteNamespace(data.cluster, data.deleteDialog.deleteName);
  if (err) {
    proxy.$message.error(err.response.data.message);
    return;
  }
  proxy.$message.success(`Namespace(${data.deleteDialog.deleteName}) 删除成功`);

  clean();
  getNamespaces();
};

const cancel = () => {
  clean();
};

const clean = () => {
  data.deleteDialog.close = false;
  setTimeout(() => {
    data.deleteDialog.deleteName = '';
  }, 100);
};

const onChange = (v) => {
  data.pageInfo.limit = v.limit;
  data.pageInfo.page = v.page;

  data.tableData = getTableData(data.pageInfo, data.namespaceList);

  if (data.pageInfo.search.searchInfo !== '') {
    searchNamespace();
  }
};

const createNamespace = () => {
  const url = `/namespaces/createNamespace?cluster=${data.cluster}`;
  router.push(url);
};

const getNamespaces = async () => {
  data.loading = true;
  const [result, err] = await getNamespaceList(data.cluster);
  data.loading = false;
  if (err) {
    proxy.$message.error(err.response.data.message);
    return;
  }

  data.namespaceList = result.items;
  data.pageInfo.total = data.namespaceList.length;
  data.tableData = getTableData(data.pageInfo, data.namespaceList);
};

const searchNamespace = async () => {
  data.tableData = searchData(data.pageInfo, data.namespaceList);
};

const jumpNamespaceRoute = (row) => {
  proxy.$notify.warning('暂不支持，请多关注社区变动');
  return;
  router.push({
    name: 'NamespaceDetail',
    query: {
      cluster: data.cluster,
      name: row.metadata.name,
    },
  });
};

const formatStatus = (row, column, cellValue) => {
  if (cellValue.phase === 'Active') {
    return formatterIcon('#28C65A', '运行中');
  }
  return formatterIcon('#FFFF00', cellValue.phase);
};

const editYaml = async (row) => {
  const [result, err] = await getNamespace(data.cluster, row.metadata.name);
  if (err) {
    proxy.$notify.error(err.response.data.message);
    return;
  }
  data.editYamlDialog = true;
  data.yaml = result;
};

const confirmEditYaml = async (yamlData) => {
  //todo
};
</script>

<style scoped="scoped">
.icon-box {
  padding: 3px;
  margin-top: -1px;
}
</style>
