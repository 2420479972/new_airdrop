<template>
  <div class="w-full h-full p-[12px] bg-[#F4F5F7]">
    <div class="w-full p-[9px] bg-[#fff] rounded-[5px]">
      <div class="text-[#3B3D47] text-[10px]">聚合联盟</div>
      <div class="mt-[12px] w-full">
        <a-table :columns="columns" :data-source="data" :scroll="{ x: 1300, y: 1000 }">
          <template #bodyCell="{ column, text}">
            <template v-if="column.key == 'url'">
              <a :href="text?.['baseinfo'][column.key]">{{text?.['baseinfo'][column.key]}}</a>
            </template>
            <template v-if="column.key == 'action'">
              <a-button type="link" @click="handleEdit(text)">编辑</a-button>
            </template>
            <template  v-else-if="column?.render">
              {{column?.render(text?.['baseinfo'][column.key])}}
            </template>
            <template v-else>
              {{text?.['baseinfo'][column.key]}}
            </template>
          </template>
        </a-table>
      </div>
    </div>
  </div>
  <a-drawer
      v-model:open="open"
      :closable="false"
      :size="524"
      placement="right"
      title="编辑"
      destroy-on-close
  >
    <template #extra>
      <a-button type="text">X</a-button>
    </template>
    <a-form ref="formRef" :model="editData" :rules="rules">
      <div class="space-y-[25px]">
        <a-form-item name="token" label="会议发起地址">
          <a-input v-model:value="editData['info']"></a-input>
        </a-form-item>
        <a-form-item name="wallet" label="会议链接">
          <a-input v-model:value="editData['url']"></a-input>
        </a-form-item>
        <a-form-item name="time_start" label="开放时间">
          <a-date-picker  format="YYYY-MM-DD HH:mm"  :show-time="{ format: 'HH:mm' }" class="w-full" v-model:value="editData['time_start']"/>
        </a-form-item>
        <a-form-item label="结束时间" name="time_end">
          <a-date-picker format="YYYY-MM-DD HH:mm" :show-time="{ format: 'HH:mm' }" class="w-full" v-model:value="editData['time_end']"/>
        </a-form-item>
      </div>
    </a-form>

    <template #footer>
      <div class="flex items-center justify-end gap-x-[10px]">
        <a-button style="margin-right: 8px">取消</a-button>
        <a-button type="primary" @click="onSubmit">确认</a-button>
      </div>

    </template>
  </a-drawer>
</template>

<script lang="ts" setup>
import {useRead} from "@/hooks/useRead.ts";
import {formatTime, getNumber} from "utils/base.ts";
import {message} from "ant-design-vue";
import {useWrite} from "@/hooks/useWrite.ts";
import {parseEther} from "viem";
import {Rule} from "postcss";
import dayjs from "dayjs";

const columns = [

  {
    title: '开始时间',
    key: 'time_start',
    render(value:string){
      return formatTime(value)
    }
  },
  {
    title: '结束时间',
    key: 'time_end',
    render(value:string){
      return formatTime(value)
    }
  },
  {
    title: '会议发起地址',
    key: 'info',
  },
  {
    title: '会议链接',
    key: 'url',
  },
  {
    title: '操作',
    key: 'action',
  },
].map(item => {
  return {
    ...item,
    align: 'center'
  }
});

const data = ref([]);

const page = ref([0,10]);
useRead('get_aggregate_alliances',page,{
  type:'ERC1229',
  onSuccess(res){
    console.log(res)
    data.value = res
  },
  onError(error){
    message.error(error)
  }
})


const editData = ref({});
const rules: Record<string, Rule[]> = {
  info: [
    { required: true, message: '请输入会议发起地址', trigger: 'change' },
  ],
  url: [{ required: true, message: '会议链接', trigger: 'change' }],
  time_start: [
    {
      required: true,
      message: '请选择开放时间',
      trigger: 'change',
    },
  ],
  time_end: [{
    required: true,
    message: '请选择结束时间',
    trigger: 'change',
  },],
};
const open = ref(false);
const {write} = useWrite('set_aggregate_alliance',{
  type:'ERC1229',
  onSuccess(res){
    message.success('操作成功')
  },
  onError(error){
    message.error(error)
  }
})
const handleEdit = (item)=>{
  open.value = true;
  editData.value = {
    ...item['baseinfo'],
    time_start: dayjs(formatTime(item['baseinfo'].time_start).toString(), 'YYYY-MM-DD HH:mm'),
    time_end: dayjs(formatTime(item['baseinfo'].time_end).toString(), 'YYYY-MM-DD HH:mm'),
  }
}


const formRef = ref();
const  onSubmit = () => {
  formRef.value
      .validate()
      .then(() => {
        console.log({
          ...editData.value,
          time_end:BigInt(editData.value.time_end.unix()),
          time_start:BigInt(editData.value.time_start.unix()),
        })
        write([
            [
              {
                ...editData.value,
                time_end:BigInt(editData.value.time_end.unix()),
                time_start:BigInt(editData.value.time_start.unix()),
              }
            ]
        ])
      })
};

</script>

<style lang="scss" scoped>

</style>
