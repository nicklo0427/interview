<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-form @reset="initData">
          <q-input
            ref="nameRef"
            v-model="tempData.name"
            label="姓名"
            lazy-rules
            :rules="rules.nameRules"
          />
          <q-input
            ref="nameRef"
            v-model="tempData.age"
            label="年齡"
            lazy-rules
            :rules="rules.ageRules"
          />
          <q-btn
            v-if="!tempData.id"
            color="primary"
            class="q-mt-md"
            @click="addNewData"
            >新增</q-btn
          >
          <q-btn v-else color="secondary" class="q-mt-md" @click="patchData"
            >保存</q-btn
          >
        </q-form>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { useQuasar } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const $q = useQuasar();
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const api = {
  GET: {
    url: 'https://dahua.metcfire.com.tw/api/CRUDTest/a',
  },
  POST: {
    url: 'https://dahua.metcfire.com.tw/api/CRUDTest',
  },
  PATCH: {
    url: 'https://dahua.metcfire.com.tw/api/CRUDTest',
  },
  DELETE: {
    url: 'https://dahua.metcfire.com.tw/api/CRUDTest',
  },
};

const nameRef = ref(null);
const ageRef = ref(null);

const tempData = ref({
  name: '',
  age: '',
});

const initData = () => {
  tempData.value = {
    name: '',
    age: '',
  };
};

const rules = ref({
  nameRules: [(val) => (val && val.length > 0) || '請輸入姓名'],
  ageRules: [(val) => /^\d+$/.test(val) || '請輸入正確年齡'],
});

function handleClickOption(btn, data) {
  // ...
  btn.status === 'edit' ? editData(data) : confirmDelete(data);
}

const editData = async (data) => {
  tempData.value = JSON.parse(JSON.stringify(data));
};

const confirmDelete = (data) => {
  $q.dialog({
    title: '刪除確認',
    message: '確定要刪除此筆資料嗎？',
    ok: {
      label: '確定',
      color: 'negative',
    },
    cancel: {
      label: '取消',
      color: 'grey-8',
    },
  }).onOk(() => {
    deleteData(data);
  });
};

const deleteData = async (data) => {
  try {
    const res = await axios.delete(`${api.DELETE.url}/${data.id}`);
    if (res) {
      getTableData();
      initData();
    }
  } catch (error) {
    console.error(error);
  }
};

const patchData = async () => {
  try {
    const res = await axios.patch(api.PATCH.url, tempData.value);
    if (res) {
      getTableData();
      initData();
    }
  } catch (error) {
    console.error(error);
  }
};

const addNewData = async () => {
  try {
    const res = await axios.post(api.POST.url, tempData.value);
    if (res) {
      getTableData();
      initData();
    }
  } catch (error) {
    console.error(error);
  }
};

const getTableData = async () => {
  try {
    const res = await axios.get(api.GET.url);
    blockData.value = res.data;
  } catch (error) {
    console.error(error);
  }
};

getTableData();
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
