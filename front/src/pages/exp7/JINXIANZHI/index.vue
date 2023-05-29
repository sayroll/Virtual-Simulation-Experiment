

<template>
    <div class="container">
        <h2>实验报告</h2>
        <div class="report-section">
            <div class="report-item">
                <h3>一.实验目的</h3>
                <p>这个实验的目的是通过实践理解和应用两种重要的金融分析工具 - 净现值（NPV）和内部回报率（IRR）。学习者将学会如何通过图表更直观地解析和解释这两个指标，从而更好地评估投资项目的盈利能力。</p>
            </div>
            <div class="report-item">
                <h3>二.实验原理</h3>
                <p>
        1. NPV是一种投资决策工具，用于评估投资项目的价值。它将项目产生的所有现金流量折现到现在，然后减去初始投资。NPV公式如下：   </p>
        <p><strong>NPV = ∑ Ct / (1+r)^t - C0</strong></p>
 
    <p>
        2. IRR是使投资项目的NPV为零的折现率。IRR公式如下：</p>
        <p style="font-weight: bold; font-size: larger;">NPV = ∑ Ct / (1+r)^t - C0</p>
    
            </div>
            <div class="report-item">
                <h3>三.实验步骤</h3>
                <p>步骤 1：设定折现率。在我们的实验中，你需要设定五个折现率，这五个折现率将会被用来计算 NPV。</p>
                <p>步骤 2：输入现金流。首先，你需要输入初始投资额（通常是负值，因为这是你支出的现金）。然后，你需要预期的未来现金流。这些现金流是你预期的项目收益。</p>
                <p>步骤 3：计算 NPV。根据你设定的折现率和输入的现金流，应用将自动计算出五个 NPV 值。</p>
                <p>步骤 4：绘制 NPV 图表。根据计算出的 NPV 值和对应的折现率，绘制出 NPV 图表。</p>
                <p>步骤 5：计算 IRR。应用将使用输入的现金流来计算 IRR。</p>
                <p>步骤 6：绘制 IRR 图表。在图表中标出 IRR，方便我们直观地看出在哪个折现率下，项目的 NPV 为零。</p>
            </div>
        </div>
        <div class="input-section">
            <div class="input-group">
                <label for="inputNumber">请输入期数：</label>
                <a-input-number id="inputNumber" v-model:value="periodvalue" :min="1" :max="50" />
            </div>
            <a-button type="primary" @click="refreshcomp()">确认</a-button>
        </div>
        <div class="input-group">
            <label for="discountRate">请输入五组折现率：</label>
            <a-input-number id="discountRate" v-model:value="discount_rate1" :min="0" :max="100"
                :formatter="value => `${value}%`" :parser="value => value.replace('%', '')" />
            <a-input-number id="discountRate" v-model:value="discount_rate2" :min="0" :max="100"
                :formatter="value => `${value}%`" :parser="value => value.replace('%', '')" />
            <a-input-number id="discountRate" v-model:value="discount_rate3" :min="0" :max="100"
                :formatter="value => `${value}%`" :parser="value => value.replace('%', '')" />
            <a-input-number id="discountRate" v-model:value="discount_rate4" :min="0" :max="100"
                :formatter="value => `${value}%`" :parser="value => value.replace('%', '')" />
            <a-input-number id="discountRate" v-model:value="discount_rate5" :min="0" :max="100"
                :formatter="value => `${value}%`" :parser="value => value.replace('%', '')" />
        </div>
        <div class="calculate-button">
            <a-button type="primary" @click="cal(); drawchart();" class="calculate-result">计算结果</a-button>
        </div>
    </div>
    <div class="table-container">
        <a-table :columns=" columns " :data-source=" dataSource " bordered>
            <template #bodyCell=" { column, text, record } ">
                <template v-if=" ['cashflow'].includes(column.dataIndex) ">
                    <div>
                        <a-input v-if=" editableData[record.key] "
                            v-model:value=" editableData[record.key][column.dataIndex] " style="margin: -5px 0" />
                        <template v-else>
                            {{ text }}
                        </template>
                    </div>
                </template>
                <template v-else-if=" column.dataIndex === 'operation' ">
                    <div class="editable-row-operations">
                        <span v-if=" editableData[record.key] ">
                            <a-button type="primary" size="small" @click=" save(record.key) "
                                class="operation-btn">Save</a-button>
                            <a-popconfirm title="Sure to cancel?" @confirm=" cancel(record.key) ">
                                <a-button type="default" size="small" class="operation-btn">Cancel</a-button>
                            </a-popconfirm>
                        </span>
                        <span v-else>
                            <a-button type="primary" size="small" @click=" edit(record.key) "
                                class="operation-btn">Edit</a-button>
                        </span>
                    </div>
                </template>
            </template>
        </a-table>
    </div>
    <div class="result-section">
        <div class="result-item">
            <span class="result-label">净现值1（NPV1）：</span>
            <span class="result-value"> {{ npv_result[0] }} </span>
        </div>
        <div class="result-item">
            <span class="result-label">净现值2（NPV2）：</span>
            <span class="result-value"> {{ npv_result[1] }} </span>
        </div>
        <div class="result-item">
            <span class="result-label">净现值3（NPV3）：</span>
            <span class="result-value"> {{ npv_result[2] }} </span>
        </div>
        <div class="result-item">
            <span class="result-label">净现值4（NPV4）：</span>
            <span class="result-value"> {{ npv_result[3] }} </span>
        </div>
        <div class="result-item">
            <span class="result-label">净现值5（NPV5）：</span>
            <span class="result-value"> {{ npv_result[4] }} </span>
        </div>
        <div class="result-item">
            <span class="result-label">内部收益率（IRR）：</span>
            <span class="result-value">{{ irr.toFixed(2) }}%</span>
        </div>
    </div>

    <div id="chart1" style="width: 600px;height:400px;"></div>
    <div id="chart2" style="width: 600px;height:400px;"></div>
    <!-- <div ref="chart" style="width: 100%; height: 400px;"></div> -->
    <div>hello</div>
</template>



<script lang="ts">
import { cloneDeep } from 'lodash-es';
import { defineComponent, reactive, ref } from 'vue';
import type { UnwrapRef } from 'vue';
import * as echarts from 'echarts';
import { Document } from '@element-plus/icons-vue';
import { onMounted } from 'vue';
import { VueMathjax } from 'vue-mathjax'


interface DataItem {
    key: string;
    period: string;
    cashflow: number;
    cashflow_discounted: number;
}

export default {
    name: 'JINXIANZHI',
    setup() {
        //期数  
        const periodvalue = ref<number>(3);
        const discount_rate1 = ref<number>(10);
        const discount_rate2 = ref<number>(10);
        const discount_rate3 = ref<number>(10);
        const discount_rate4 = ref<number>(10);
        const discount_rate5 = ref<number>(10);
        const npv = ref<number>(0);
        var npv_result: number[] = [0, 0, 0, 0, 0];
        const discount_0 = ref<number>(10);
        const discount_100 = ref<number>(10);
        var npv_show:number[] = [0,0];
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
                cashflow_discounted: 0,
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
                    cashflow_discounted: 0,
                });
            }
            npv_result[0] = 0;
            npv_result[1] = 0;
            npv_result[2] = 0;
            npv_result[3] = 0;
            npv_result[4] = 0;
            irr.value = 0;
            dataSource.value = newData;
        };

        const cal = () => {
            // Calculate NPV
            for (let i = 0; i < 5; i++) {
                let npvSum = 0;
                let discount_rate = 0;
                switch (i) {
                    case 0:
                        discount_rate = discount_rate1.value;
                        break;
                    case 1:
                        discount_rate = discount_rate2.value;
                        break;
                    case 2:
                        discount_rate = discount_rate3.value;
                        break;
                    case 3:
                        discount_rate = discount_rate4.value;
                        break;
                    case 4:
                        discount_rate = discount_rate5.value;
                        break;
                }
                dataSource.value.forEach((item, index) => {
                    const discountedCashFlow = item.cashflow / Math.pow(1 + discount_rate / 100, index);
                    item.cashflow_discounted = discountedCashFlow;
                    npvSum += discountedCashFlow;
                });
                npv_result[i] = npvSum;
            };
            //npv.value = npvSum;
            for(let i = 0 ; i < 2;i++)
            {
                let npvSum = 0;
                let discount_rate = 0;
                discount_0.value = 0;
                discount_100.value = 100;
                switch (i) {
                    case 0:
                        discount_rate = discount_0.value;
                        break;
                    case 1:
                        discount_rate = discount_100.value;
                        break;
                }
                dataSource.value.forEach((item, index) => {
                    const discountedCashFlow = item.cashflow / Math.pow(1 + discount_rate / 100, index);
                    npvSum += discountedCashFlow;
                });
                npv_show[i] = npvSum;
            }
            // Calculate IRR using bisection method
            const cashflows = dataSource.value.map(item => item.cashflow);
            const maxIter = 1000;
            const tol = 0.0001;
            let lower = -0.9999; // Lower bound, can't be -1 because it will cause a division by zero
            let upper = 2;
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

        //下方图表
        // const chartRef = ref(null);
        onMounted(() => {
            var mychart = echarts.init(document.getElementById('chart1'));

            // 模拟数据
            var options = {
                title: {
                    text: 'NPV图示'
                },
            };

            mychart.setOption(options);

            //IRR表
            var mychart2 = echarts.init(document.getElementById('chart2'));

            // 模拟数据
            var options2 = {
                title: {
                    text: 'IRR 图示'
                },
            };
            mychart2.setOption(options2);
        });

        const drawchart = () => {
            var mychart = echarts.getInstanceByDom(document.getElementById('chart1'));
            var mychart2 = echarts.getInstanceByDom(document.getElementById('chart2'));
            console.log(npv_result);
            console.log(discount_rate5.value);
            console.log(typeof(discount_rate5.value));
            console.log(discount_rate4.value);
            console.log(discount_rate3.value);
            console.log(discount_rate2.value);
            console.log(discount_rate1.value);
            var options = {
                title: {
                    text: 'NPV图示',
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'cross'
                    }
                },
                xAxis: {
                    //data:[-2,-1,0,1,2],
                    data: [discount_rate1.value/100, discount_rate2.value/100, discount_rate3.value/100, discount_rate4.value/100, discount_rate5.value/100],
                    name:'折现率',
                },
                yAxis: {
                    type: 'value',
                    name:'净现值',
                },
                series: [
                    {
                        data:[npv_result[0],npv_result[1],npv_result[2],npv_result[3],npv_result[4]],
                        type: 'line'
                    }
                ]
            };
            mychart.setOption(options);
            var x;
            var y;
            if(irr.value/100 <0)
            {
                x = [(irr.value/100).toFixed(4),0/100,100/100];
                y = [
                    {value:0,itemStyle:{color:'red'}},{value:npv_show[0]},{value:npv_show[1]}];
            }
            else
            {
                if(irr.value/100 < 1)
                {
                    x = [0/100,(irr.value/100).toFixed(4),100/100];
                    y = [{value:npv_show[0]},{value:0,itemStyle:{color:'red'}},{value:npv_show[1]}];
                }
                else
                {
                    x = [0/100,100/100,(irr.value/100).toFixed(4)];
                    y = [{value:npv_show[0]},{value:npv_show[1]},{value:0,itemStyle:{color:'red'}}];
                }
            }
            var options2 = {
                title: {
                    text: 'IRR图示',
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'cross'
                    }
                },
                xAxis: {
                    //data:[-2,-1,0,1,2],
                    data: x,
                    name:"折现率",
                    boundaryGap:true,
                },
                yAxis: {
                    type: 'value',
                    name:'净现值',
                },
                series: [
                    {
                        data:y,
                        type: 'line'
                    }
                ]
            };
            mychart2.setOption(options2);
        };

        return {
            discount_rate1,
            discount_rate2,
            discount_rate3,
            discount_rate4,
            discount_rate5,
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
            drawchart,
            npv,
            npv_result,
            irr,
            npv_show,
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
.report-section {
    margin-bottom: 40px;
}
.report-item {
  text-align: left;
  text-indent: 2em;
}
.report-item h3 {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 10px;
}

.report-item p {
    font-size: 18px;
    margin-bottom: 20px;
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