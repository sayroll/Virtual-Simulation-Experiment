<template>
    <div class="container">
        <h2>--这里是净现值(NPV)与内部收益率(IRR)实验--</h2>
        <div class="input-section">
            <div class="input-group">
                <label for="inputNumber">请输入期数：</label>
                <a-input-number id="inputNumber" v-model:value="periodvalue" :min="1" :max="50" />
            </div>
            <a-button type="primary" @click="refreshcomp()">确认</a-button>
        </div>
        <div class="input-group">
            <label for="discountRate">请输入折现率：</label>
            <a-input-number id="discountRate" v-model:value="discount_rate" :min="0" :max="100" :formatter="value => `${value}%`"
                :parser="value => value.replace('%', '')" />
        </div>
        <div class="calculate-button">
            <a-button type="primary" @click="cal()" class="calculate-result">计算结果</a-button>
        </div>
    </div>
    <div class="table-container">
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
                            <a-button type="primary" size="small" @click="save(record.key)" class="operation-btn">Save</a-button>
                            <a-popconfirm title="Sure to cancel?" @confirm="cancel(record.key)">
                                <a-button type="default" size="small" class="operation-btn">Cancel</a-button>
                            </a-popconfirm>
                        </span>
                        <span v-else>
                            <a-button type="primary" size="small" @click="edit(record.key)" class="operation-btn">Edit</a-button>
                        </span>
                    </div>
                </template>
            </template>
        </a-table>
    </div>
    <div class="result-section">
        <div class="result-item">
            <span class="result-label">净现值（NPV）：</span>
            <span class="result-value">{{ npv }}</span>
        </div>
        <div class="result-item">
            <span class="result-label">内部收益率（IRR）：</span>
            <span class="result-value">{{ irr.toFixed(2) }}%</span>
        </div>
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
        const npv = ref<number>(0);
        const irr = ref<number>(0);

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
        const validateCashFlow = (value: any): boolean => {
            return !isNaN(parseFloat(value)) && isFinite(value);
        };

        const save = (key: string) => {
            if (!validateCashFlow(editableData[key].cashflow)) {
                alert('请输入有效的现金流数据！');
                return;
            }

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

        const cal = () => {
        // Calculate NPV
            let npvSum = 0;
            dataSource.value.forEach((item, index) => {
            const discountedCashFlow = item.cashflow / Math.pow(1 + discount_rate.value / 100, index);
            item.cashflow_discounted = discountedCashFlow;
        npvSum += discountedCashFlow;
    });
    npv.value = npvSum;

    // Calculate IRR using bisection method
    const cashflows = dataSource.value.map(item => item.cashflow);
    const maxIter = 1000;
    const tol = 0.0001;
    let lower = -0.9999; // Lower bound, can't be -1 because it will cause a division by zero
    let upper = 1;
    let mid = 0;
    let npvMid = 0;
    let iter = 0;

    while (iter < maxIter) {
        mid = (lower + upper) / 2;
        npvMid = cashflows.reduce((acc, cf, i) => acc + cf / Math.pow(1 + mid, i), 0);

        if (Math.abs(npvMid) < tol) {
            break;
        } else if (npvMid > 0) {
            lower = mid;
        } else {
            upper = mid;
        }
        iter++;
    }

    if (isFinite(mid)) {
        irr.value = mid * 100;
    } else {
        irr.value = NaN;
        alert('IRR 计算出现问题，请尝试使用其他投资评估指标，如 MIRR 或 NPV。');
    }
};
    

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
            cal,
            npv,
            irr
        };
    },
}
</script>
<style scoped>
.container {
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
    font-family: 'Arial', sans-serif;
}

h2 {
    font-size: 28px;
    font-weight: bold;
    text-align: center;
    margin-bottom: 40px;
}

.input-section {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.input-group {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
}

.input-group label {
    font-size: 16px;
    font-weight: bold;
    margin-right: 10px;
}

.confirm-button {
    margin-left: 20px;
}

.calculate-button {
    text-align: center;
    margin-bottom: 20px;
}

.table-container {
    margin-bottom: 40px;
}

.result-section {
    display: flex;
    justify-content: center;
}

.result-card {
    background-color: #f0f2f5;
    padding: 20px;
    border-radius: 4px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
    width: 50%;
    text-align: center;
}

.result-card p {
    font-size: 18px;
    font-weight: bold;
}
.editable-row-operations {
    display: flex;
    justify-content: space-between;
    width: 150px;
}

.operation-btn {
    margin-right: 10px;
}

.a-table {
    width: 100%;
    background-color: #fff;
    border-collapse: collapse;
}

.a-table thead th {
    background-color: #f0f2f5;
    padding: 10px;
    text-align: left;
    font-weight: bold;
}

.a-table tbody tr:nth-child(even) {
    background-color: #f0f2f5;
}

.a-table tbody tr:hover {
    background-color: #e6f7ff;
}

.a-table tbody td {
    padding: 10px;
}

.result-section {
        display: flex;
        flex-direction: column;
        align-items: center;
        font-size: 18px;
        margin-top: 30px;
    }
    
    .result-item {
        display: flex;
        align-items: center;
        margin-bottom: 10px;
    }
    
    .result-label {
        font-weight: bold;
        margin-right: 5px;
    }
    
    .result-value {
        color: #1890ff;
    }
</style>