<template>
    <section  style="overflow-y:auto;">
        <div class="L-center-div">
            <!-- table表格 -->
            <el-table :data="tableData" style="width: 100%" min-width="1000px" :show-header='false' class='L-table' highlight-current-row>
                <el-table-column width="60">
                    <template scope="scope">
                        <img :src="scope.row.title | filter" alt="" style="width:20px; height:20px;margin-top:10px;">
                    </template>
                </el-table-column>
                <el-table-column prop="title" width="150">
                </el-table-column>
                <el-table-column prop="content" width="500" class-name="L-column">
                </el-table-column>
                <el-table-column style="width:40%">
                </el-table-column>
                <el-table-column width="100">
                    <template scope="scope">
                        <el-button class="L-button" size="small" @click="toUrl(scope.row)">
                            <img src="../imgs/url.png" alt="" style="width:1.5rem; height:1.5rem;">
                        </el-button>
                    </template>
                </el-table-column>
                <el-table-column prop="sendTime" width="150" :span="6" :offset="6" :formatter="formatTime">
                </el-table-column>
            </el-table>
            <!-- 分页 -->
           <el-pagination class="L-pagination" layout="total, prev, pager, next" @current-change="currentChange"  :current-page="this.page.currentPage" :total="this.page.total">
           </el-pagination>
        </div>
    </section>
</template>

<script>
    import { api } from "../js/api/api";
    import { getCookie } from '../js/api/cookie';
    import { getDate } from '../js/api/date';

    export default {
        data() {
            return {
                tableData: [],
                page:{
                    currentPage:1,
                    total:0,
                    rows:10
                }
            }
        },
        mounted() {
            this.getUsers();
        },
        methods:{
            currentChange:function(val){
                this.page.currentPage = val;
                this.getUsers();
            },
            getUsers() {
                if(getCookie("token")){
        			let para = {
        				rows: this.page.rows,
        				page: this.page.currentPage,
                        sort:"send_time",
        				order: "desc",
                        readStatus: 0
        			};
                    this.$http.post(api.NewMessage , para)
        			.then((res) => {
    					if(res.ok && res.body.rows.length != 0){
                            this.$root.eventHub.$emit("EVENT_MESSAGE_NEW", "ture")
    						this.tableData = res.body.rows;
    						this.page.total = res.body.total;
    					}else{
                            this.$root.eventHub.$emit("EVENT_MESSAGE_NEW", "")
    					}
        			},(err) => {
    					console.log(err)
    				   }
                    )
                    setTimeout(() => {
                        this.getUsers()
                    }, 5 * 60 * 1000)
                }
    		},
            formatTime:function(row){
                return getDate(row.sendTime)
            },
            toUrl:function(data){
                this.$http.post( api.updateReadStatus , {sid:data.sid}).then((res) => {
                    if(res.ok){
                        this.$router.push({
                            path:data.url
                        })
                    }
                })
            }
        },
        filters:{
            filter:function(data){
                if(data == "产品信息"){
                    return  "./imgs/pro.png"
                }else if(data == "绑定信息"){
                    return  "./imgs/addr.png"
                }else if(data == "服务信息"){
                    return  "./imgs/service.png"
                }else if(data == "订单信息"){
                    return  "./imgs/order.png"
                }
            }
        }
    }
</script>

<style lang="css">
    .L-table{
        position: relative;
        margin-top: 50px;
    }
    .L-table:before{
        content: '';
        position:absolute;
        top:0;
        left:0;
        width: 100%;
        height: 0 !important;
        border: 1px solid #A5A5A5;
    }
    .L-table tr{
        height:5rem;
        border-bottom: 1px solid #A5A5A5
    }
    .L-column .cell{
        text-align: left !important;
    }
    .L-pagination{
        text-align: right;
        padding: 0 !important;
    }
</style>
