<template>
  <div class="w-full h-full p-[12px] bg-[#F4F5F7]">
    <div class="w-full p-[9px] bg-[#fff] rounded-[5px]">

      <div class="mt-[12px] w-full">
        <div class="flex items-center gap-x-10 mb-5">
          <div class="text-[#3B3D47] text-[10px]">批量空投</div>
         <a-form-item label="选择节点">
           <a-select
               v-model:value="selectedValue"
               style="width: 390px"
               :options="nodeList"
           ></a-select>
         </a-form-item>
          <a-button type="primary" @click="show">批量空投</a-button>
        </div>
        <a-table :columns="columns" :data-source="tableData" :scroll="{ x: 1300, y: 1000 }" :row-selection="rowSelection">
          <template #bodyCell="{ column, text}">
            <template  v-if="column?.render">
              {{column?.render(text?.[column.key])}}
            </template>
            <template v-else>
              {{text?.[column.key]}}
            </template>
          </template>
        </a-table>
      </div>
    </div>
  </div>
  <a-modal
      v-model:open="open"
      :closable="false"
      placement="right"
      title="批量空投"
      destroy-on-close
      @close="batchReset"
  >
    <template #extra>
      <a-button type="text">X</a-button>
    </template>
    <a-form ref="formRef" :model="batch" :rules="rules">
      <div class="space-y-[25px]">
        <a-form-item name="token" label="空投地址">
          <a-input v-model:value="batch['token']"></a-input>
        </a-form-item>
        <a-form-item name="baseamount" label="空投数量">
          <a-input-number class="w-full" v-model:value="batch['baseamount']"></a-input-number>
        </a-form-item>
      </div>
    </a-form>

    <template #footer>
      <div class="flex items-center justify-end gap-x-[10px]">
        <a-button style="margin-right: 8px" @click="open = false">取消</a-button>
        <a-button type="primary" @click="onSubmit">确认</a-button>
      </div>

    </template>
  </a-modal>
</template>

<script lang="ts" setup>
import {useRead} from "@/hooks/useRead.ts";
import {formatAddress} from "utils/base.ts";
import {message} from "ant-design-vue";
import {Rule} from "postcss";
import {useWrite} from "@/hooks/useWrite.ts";
import {parseEther} from "viem";
const open = ref(false);
const dateFormat = 'YYYY-MM-DD HH:mm';


const columns = [
  {
    title: 'vip地址',
    key: 'vip_address',
    render(value:string){
      return formatAddress(value)
    }
  },
]
const selectedValue = ref();
const tableData = ref([]);
const nodeList = ref([])
const nodeVipParams = ref(["",0,10])

const {refetch} = useRead('get_nodevipinfo',nodeVipParams,{
  type:'ERC1229',
  onSuccess(value: any) {
    tableData.value = value.vips.map((item: any, index: number) => {
      return {
        key: index,
        index: index,
        text:item,
        vip_address: item,
      }
    })
  },
  onError(error){
    message.error(error)
  }
})

const nodePage = ref([0,10])
useRead('get_node_list',nodePage,{
  type:'ERC1229',
 async onSuccess(res){
    nodeList.value = res.map(item=>{
      return {
        label:item,
        value:item
      }
    })
   console.log(res)
   nodeVipParams.value[0] = nodeList.value[0].value
   selectedValue.value = nodeList.value[0].value
   refetch()
  },
  onError(error){
    message.error(error)
  }
})

watch(()=>selectedValue.value,(newVal)=>{
  nodeVipParams.value[0] = newVal
  refetch()
})

const selectedList = ref([]);
const rowSelection = ref({
  checkStrictly: true,
  selectedRowKeys:[],
  onChange: (selectedRowKeys: [], selectedRows: []) => {
    rowSelection.value.selectedRowKeys = selectedRowKeys
    selectedList.value = selectedRows;
  },
});


const batch = ref({})
const formRef = ref();
const rules: Record<string, Rule[]> = {
  token: [
    { required: true, message: '请输入空投地址', trigger: 'change' },
  ],
  baseamount: [{ required: true, message: '请输入空投数量', trigger: 'change' }],
};

const batchReset = ()=>{
  formRef.value?.resetFields()
}


const {write} = useWrite('platform_airdrop',{
  type:'ERC1229',
  onSuccess(value: any) {
    message.success('空投成功')
    open.value = false
    selectedList.value = [];
    batchReset()
    refetch()
  },
  onError(error){
    message.error(error)
  }
})

const onSubmit = ()=>{
  formRef.value
      .validate()
      .then(() => {
        write([
          selectedList.value.map(item=>item.vip_address),
          batch.value.token,
          parseEther(String(batch.value.baseamount))
        ])
      })
}


const show = ()=>{
  if(selectedList.value.length < 1){
    message.error('请先选择节点')
    return
  }
  rowSelection.value.selectedRowKeys = []
  open.value = true
}

</script>

<style lang="scss" scoped>
:deep(.ant-form-item){
  margin-bottom: 0;
}
</style>
