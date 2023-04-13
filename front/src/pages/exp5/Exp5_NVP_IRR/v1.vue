<template>
    <div>
        <h2>--这里是净现值(NPV)与内部收益率(IRR)实验--</h2>
        <div>
            请输入期数： <a-input-number id="inputNumber" v-model:value="periodvalue" :min="1" :max="50" />
            <!-- 当前值：{{ value }}  -->
            <a-button type="primary" @click="refreshcomp()">确认</a-button>
            <a-button type="primary" danger style="margin-left:20px;" @click="cal()">计算结果</a-button>
        </div>
        <div>
            请输入折现率：<a-input-number v-model:value="discount_rate" :min="0" :max="100" :formatter="value => `${value}%`"
                :parser="value => value.replace('%', '')" />
        </div>
    </div>
    <div>
        <a-table :columns="columns" :data-source="dataSource" bordered>
            <template #bodyCell="{ column, text, record }">
                <template v-if="['cashflow'].includes(column.dataIndex)">
                    <div>
                        <a-input v-if="editableData[record.key]" v-model:value="editableData[record.key][column.dataIndex]"
                            style="margin: -5px 0" />
                        <template v-else>
                            {{ text }}
                        </template>
                    </div>
                </template>
                <template v-else-if="column.dataIndex === 'operation'">
                    <div class="editable-row-operations">
                        <span v-if="editableData[record.key]">
                            <a-typography-link @click="save(record.key)">Save</a-typography-link>
                            <a-popconfirm title="Sure to cancel?" @confirm="cancel(record.key)">
                                <a>Cancel</a>
                            </a-popconfirm>
                        </span>
                        <span v-else>
                            <a @click="edit(record.key)">Edit</a>
                        </span>
                    </div>
                </template>
            </template>
        </a-table>
    </div>

</template>

<script lang="ts">
import { cloneDeep } from 'lodash-es';
import { defineComponent, reactive, ref } from 'vue';
import type { UnwrapRef } from 'vue';

interface DataItem {
    key: string;
    period: string;
    cashflow: number;
    cashflow_discounted:number;
}

export default {
    setup() {
        //期数
        const periodvalue = ref<number>(3);
        const discount_rate = ref<number>(10);

        const columns = [
            {
                title: 'period',
                dataIndex: 'period',
                width: '15%',
            },
            {
                title: 'cashflow',
                dataIndex: 'cashflow',
                width: '15%',
            },
            {
                title: 'cashflow_discounted',
                dataIndex: 'cashflow_discounted',
                width: '15%',
            },
            {
                title: 'operation',
                dataIndex: 'operation',
            },
        ];
        const data: DataItem[] = [];
        for (let i = 0; i <= periodvalue.value; i++) {
            //console.log("111" + periodvalue.value);
            data.push({
                key: i.toString(),
                period: `${i}`,
                cashflow: 0,
                cashflow_discounted:0,
            });
        }

        //下方table
        const dataSource = ref(data);
        const editableData: UnwrapRef<Record<string, DataItem>> = reactive({});

        const edit = (key: string) => {
            editableData[key] = cloneDeep(dataSource.value.filter(item => key === item.key)[0]);
        };
        const save = (key: string) => {
            Object.assign(dataSource.value.filter(item => key === item.key)[0], editableData[key]);
            delete editableData[key];
        };
        const cancel = (key: string) => {
            delete editableData[key];
        };

        const refreshcomp = () => {
            const newData: DataItem[] = [];
            for (let i = 0; i <= periodvalue.value; i++) {
                newData.push({
                    key: i.toString(),
                    period: `${i}`,
                    cashflow: 0,
                    cashflow_discounted:0,
                });
            }
            dataSource.value = newData;
        };

        const cal=()=>{

        }

        return {
            discount_rate,
            periodvalue,
            dataSource,
            columns,
            editingKey: '',
            editableData,
            //itemkey,
            edit,
            save,
            cancel,
            refreshcomp,
            cal
        };
    },
}
</script>

<style scoped>
.editable-row-operations a {
    margin-right: 8px;
}
</style>
