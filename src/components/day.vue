<template>
    <div class="day">
        <vheader :headermsg="header"></vheader>
        <div style="font-size: 1.2rem;width:100%">
            <table  align="center">
                <tr>
                    <td>  <selectlist  :queryList="select_list"></selectlist> </td>
                    <!--<td>每日交易量查询</td>-->
                </tr>
            </table>
        </div>
        开始时间：  <input  v-model="start"  type="date" @change="search(start,end)"/> <br/>
        结束时间：  <input  v-model="end"  type="date" @change="search(start,end)" /> <br/>
        <div class="testChart">
            <chart :options="chartData"></chart>
        </div>
    </div>
</template>
<style>
    .day{
        padding: 8px;
        margin-bottom: 55px;
    }
    .testChart .echarts {
        width: 100%;
        height: 400px;
    }
</style>
<script>
    import chart from 'vue-echarts';
    import vheader from './vheader.vue';
    import selectlist from './selectlist.vue';

    export default{
        components: {
            chart,vheader,selectlist
        },
         name: 'day',
        methods:{
            search: function(startDate,endDate) {
                if(startDate&&endDate) {
                    if (endDate < startDate) {
                        alert("结束时间不能小于开始时间")
                    } else {
                        this.curDataFee=[];
                        this.curData=[];
                        this.$http.get("/statistics/day?startDate=" + startDate + "&endDate=" + endDate).then(function (res) {
                            this.datamap = res.body['map'];
                            this.datalist =this.datamap['dateList'];
                            this.newMap=this.datamap['newMap'];
                            for( var i=0;i<this.datalist.length;i++){
                                this.curDataFee.push(this.newMap[this.datalist[i]].dayCount);
                                this.curData.push(this.newMap[this.datalist[i]].dayServiceCharge);
                            }
                        }, function (e) {
                            console.log(e)
                        });
                    }
                }else{
                    alert("请选择开始时间或结束时间");
                }
            }
        },
        created()
        {
            var now = new Date();
            for(var i = 0; i < 10; i++){
                this.yearList.push(now.getFullYear() - i)
            }
            this.maxYear = this.year = now.getFullYear();
            this.minYear = this.year - 9;
//            获取开始时间
            var ago = new Date(now.getTime() - 7 * 24 * 3600 * 1000);

            var d = ago.getDate();
            var m = ago.getMonth()+1; //January is 0!
            var yy = ago.getFullYear();

            if(d<10) {
                d = '0'+d;
            }

            if(m<10) {
                m = '0'+m
            }
            this.start =yy+"-"+m+"-"+d;
            var dd = now.getDate();
            var mm = now.getMonth()+1; //January is 0!
            var yyyy = now.getFullYear();

            if(dd<10) {
                dd = '0'+dd
            }

            if(mm<10) {
                mm = '0'+mm
            }
            this.end=yyyy+"-"+mm+"-"+dd;
            this.search( this.start,this.end);
        },
        data()
        {
            return {
                header:"每日交易量",
                select_list:"5",
                curData: [],
//                preData: [],
                curDataFee: [],
//                preDataFee: [],
//                year: 0,
                maxYear:0,
                minYear:0,
                yearList:[],
                start:"",
                end:"",
                datalist:[],//时间年月日
                datamap:{},  //相对应时间数据
                newMap:{}
            }
        },
        computed:{
            chartData(){
                return{
//                    title: {
//                        text: '堆叠区域图'
//                    },
                    tooltip : {
                        trigger: 'axis',
                        axisPointer: {
                            type: 'cross',
                            label: {
                                backgroundColor: '#6a7985'
                            }
                        }
                    },
                    legend: {
                        data:['交易量','交易额']
                    },
                    toolbox: {
                        feature: {
//                            saveAsImage: {}
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '15%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
//                            boundaryGap : false,
                            data : this.datalist,
                            xAxisIndex:0
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value',
                            name:"交易量"
                        },
                        {
                            type: 'value',
                            name:"交易额"
                        }
                    ],
                    series : [
                        {
                            name:'交易量',
                            type:'bar',
//                            stack: '交易量',
                            //areaStyle: {normal: {}},
                            data:this.curDataFee,
                            connectNulls:true,
                            yAxisIndex:0
                        },

                        {
                            name:'交易额',
                            type:'line',
//                            stack: '交易量',
//                            areaStyle: {normal: {}},
                            data:this.curData,
                            connectNulls:true,
                            yAxisIndex:1
                        },
                    ]
                }
            }
        }
    }
</script>
