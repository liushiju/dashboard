<template>
  <div style="display: flex; flex-direction: column; width: 100%; height: 100%">
    <div style="width: 100%; height: 60px; background: #ffffff; display: flex; align-items: center">
      <pixiu-icon
        name="icon-back"
        style="cursor: pointer; margin-left: 25px"
        size="16px"
        type="iconfont"
        color="#006eff"
        @click="backToNamespace"
      />

      <el-breadcrumb separator="/" style="margin-left: 10px; margin-top: 1px">
        <el-breadcrumb-item
          ><span class="breadcrumb-create-style"> {{ data.clusterName }} </span>
        </el-breadcrumb-item>
        <el-breadcrumb-item
          ><span class="breadcrumb-create-style"> 命名空间 </span>
        </el-breadcrumb-item>
        <el-breadcrumb-item
          ><span class="breadcrumb-create-style"> 创建命名空间 </span>
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>

    <el-main>
      <div class="app-pixiu-content-card">
        <el-card class="create-card-style" style="width: 75%">
          <el-form
            ref="ruleFormRef"
            label-position="left"
            require-asterisk-position="right"
            label-width="100px"
            :rules="rules"
            status-icon
            :model="data.namespaceForm"
            class="create-card-form"
          >
            <div style="margin-top: 20px" />

            <el-form-item label="名称" prop="metadata.name">
              <template #label>
                <span class="form-item-key-style">名称 </span>
              </template>
              <el-input
                v-model="data.namespaceForm.metadata.name"
                style="width: 40%"
                class="form-item-key-style"
                placeholder="请输入命名空间名称"
              />
            </el-form-item>
            <el-form-item>
              <div class="dialog-describe-style" style="margin-left: 10px; margin-top: -18px">
                最长63个字符，只能包含小写字母、数字及分隔符("-"),且必须以小写字母开头，数字或小写字母结尾
              </div>
            </el-form-item>

            <div style="margin-top: 20px" />
            <el-form-item prop="description" label="描述">
              <template #label>
                <span class="form-item-key-style">描述 </span>
              </template>
              <el-input
                v-model="data.namespaceForm.description"
                placeholder="请输入命名空间的描述信息"
                class="form-item-key-style"
                style="width: 85%"
                type="textarea"
                :autosize="data.autosize"
              />
            </el-form-item>

            <div style="margin-top: 30px" />
            <el-form-item style="margin-left: 30%">
              <el-button class="pixiu-cancel-button" @click="cancel()">取消</el-button>
              <el-button class="pixiu-confirm-button" type="primary" @click="confirm()"
                >确定</el-button
              >
            </el-form-item>
          </el-form>
        </el-card>
      </div>
    </el-main>
  </div>
</template>

<script setup>
import { reactive, getCurrentInstance, onMounted, watch, ref } from 'vue';
import { createNamespace } from '@/services/kubernetes/namespaceService';
import PixiuCard from '@/components/card/index.vue';

const { proxy } = getCurrentInstance();
const ruleFormRef = ref();

const data = reactive({
  loading: false,
  cluster: '',
  clusterName: '',

  autosize: {
    minRows: 6,
  },

  namespaceForm: {
    metadata: {
      name: '',
    },
  },
});

const confirm = () => {
  ruleFormRef.value.validate(async (valid) => {
    if (valid) {
      const [result, err] = await createNamespace(data.cluster, data.namespaceForm);
      if (err) {
        proxy.$message.error(err.response.data.message);
        return;
      }

      proxy.$message.success(`Namespace ${data.namespaceForm.metadata.name} 创建成功`);
      backToNamespace();
    }
  });
};

const cancel = () => {
  backToNamespace();
};

onMounted(() => {
  data.query = proxy.$route.query;
  data.cluster = data.query.cluster;
  data.clusterName = localStorage.getItem(data.cluster);
});

const rules = {
  'metadata.name': [{ required: true, message: '请输入命名空间名称', trigger: 'blur' }],
};

const backToNamespace = () => {
  proxy.$router.push({
    name: 'Namespace',
    query: data.query,
  });
};
</script>

<style></style>
