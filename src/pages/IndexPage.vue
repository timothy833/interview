<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />

        <q-btn v-if="isEditing" color="primary" class="q-mt-md" @click="updateData">更新</q-btn>
        <q-btn v-else color="primary" class="q-mt-md" @click="addData">新增</q-btn>
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
              <div>{{ props.row[col.name] }}</div>
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
import { ref, onMounted } from 'vue';
interface dataType {
  id?: string;
  name: string;
  age: number;
}



interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref<dataType[]>([]);
const tempData = ref<dataType>({ name: '', age: 0 });

// 用于标记是否正在编辑
const isEditing = ref(false);


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
const tableButtons = ref<btnType[]>([
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

onMounted(async () => {
  await fetchData();
});

async function fetchData() {
  try {
    const response = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = response.data;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

async function addData() {
  try {
    await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
      name: tempData.value.name,
      age: tempData.value.age,
    });
    tempData.value = { name: '', age: 0 }; // 清空表单
    await fetchData();
  } catch (error) {
    console.error('Error adding data:', error);
  }
}


async function updateData() {
  try {
    if (tempData.value.id) {
      await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
        id: tempData.value.id,
        name: tempData.value.name,
        age: tempData.value.age,
      });
      tempData.value = { name: '', age: 0 }; // 清空表单
      isEditing.value = false; // 结束编辑状态
      await fetchData();
    }
  } catch (error) {
    console.error('Error updating data:', error);
  }
}


async function deleteData(id:string) {
  try {
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    await fetchData();
  } catch (error) {
    console.error('Error deleting data:', error);
  }
}



function handleClickOption(btn: btnType, data: dataType) {
  if (btn.status === 'edit') {
    tempData.value = { ...data }; // 加载数据以供编辑
    isEditing.value = true;
  }  else if (btn.status === 'delete') {
    if (data.id) {
      deleteData(data.id);
    } else {
      console.error('Cannot delete data: ID is missing.');
    }
  }
}
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
