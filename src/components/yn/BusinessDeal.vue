<template>
    <div class="table">
          <verifyPermission></verifyPermission>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-menu"></i>业务</el-breadcrumb-item>
                <el-breadcrumb-item>业务处理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
         <el-table :data="data" border style="width: 100%"  :row-class-name="tableRowClassName"  v-loading="loading"
    element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.8)">
             <el-table-column type="index" :index="indexMethod" prop="index"  label="序号" width="100px">  </el-table-column>
             <el-table-column prop="id" label="标的id" ></el-table-column>
            <el-table-column prop="borrowerId" label="借款人id" ></el-table-column>
            <el-table-column prop="amount" label=" 借款金额"  ></el-table-column>
            <el-table-column prop="periodNo" label="分期"  ></el-table-column>
            <el-table-column prop="termQuantity" label="每期的数量"  ></el-table-column>
            <el-table-column prop="termUnit" label=" 期的单位"  ></el-table-column>

             <el-table-column  label="标的状态"  >
             <template slot-scope="scope">
               <span v-if="scope.row.status==70 || scope.row.status==72 " type="text">提现失败</span>
               <span v-else-if="scope.row.status==54 || scope.row.status==41 " type="text">转账失败</span>
               <span v-else-if="scope.row.status==53" type="text">生成债务债权失败</span>
               <span v-else-if="scope.row.status==42 || scope.row.status==33" type="text">满标失败</span>
             </template>
            </el-table-column>
            <el-table-column prop="fullBidTime"   :formatter="timetrans"   label="满标时间"  ></el-table-column>
            <el-table-column  label="资料审核">
             <template slot-scope="scope">
                <el-button size="small" type="primary"  @click="Retry(scope.$index, scope.row)">重试</el-button>   
             </template>       
            </el-table-column>
        </el-table>
        <div class="pagination">
            <el-pagination
                 @size-change="handleSizeChange"
                 @current-change="handleCurrentChange"
                 :current-page.sync="currentPage1"
                 :page-size="pageSize"
                 layout="total, prev, pager, next"
                 :total="totalNum">
           </el-pagination>
        </div>
           <dialogEE v-bind:dialogVisible="dialogVisible" ></dialogEE>
    


    </div>
</template>

<script>
  import dialogEE from './Dialog.vue';
  import verifyPermission from './Permission.vue';
    export default {
        data() {
            return {
            //    url:"http://172.20.15.48:8080/backend/IListingService/getLoanFailedRetryListingList",   
            //    url1:"http://172.20.15.48:8080/backend/IListingService/loanFailedRetry",   
                url:"/backend/IListingService/getLoanFailedRetryListingList",   
                url1:"/backend/IListingService/loanFailedRetry",              
                tableData: [], 
                dialogVisible: false, 
                 cur_page: 1, 
                currentPage1:1,
                totalNum:0,
                pageSize:20, 
                loading:false     
            }
        },
        components: {
			'dialogEE':dialogEE,  
            'verifyPermission':verifyPermission,
        },
        created(){

              this.GetqueryData(this.cur_page,this.pageSize);
        },
        computed: {
            data(){
                const self = this;
                if(self.tableData===null){
                    return self.tableData;
                }else{
                 return self.tableData.filter(function(d){                             
                     return d;                                         
                 })
                }
               
            },
           
          
        },
        methods: {
          indexMethod(index) {
              return (index+1)+(this.cur_page-1)*this.pageSize;
           },
            tableRowClassName(row, index) {
               //把每一行的索引放进row
               row.index = (index+1)+(this.cur_page-1)*this.pageSize;
           }, 
           handleSizeChange(val) {

            console.log(`每页 ${val} 条`);
          },
            handleCurrentChange(val){
                this.cur_page = val;
                this.GetqueryData(this.cur_page,this.pageSize);
            },
          
           GetqueryData(cur_page,pageSize){
                let self = this;
                  let axiosDate = new Date();
                self.loading=true;
                 self.$axios.post(self.url, {"startPage":cur_page,"pageSize":pageSize}).then((res) => {
                   if(res.data.result=="0"){
                          let oDate = new Date()
                        let time = oDate.getTime() - axiosDate.getTime()
                        if (time < 500) time = 500
                        setTimeout(() => {
                          self.loading=false;
                        }, time)
                         self.tableData =res.data.content.list;
                         self.totalNum=res.data.content.totalCount;
                                                                                                     
                   }else if( res.data.result=='-99' || res.data.result=='-999'){
                            self.$message.error(res.data.resultMessage);
                    }else{
                        self.$message.error(res.data.resultMessage);
                    }
                      
                })
           },
           Retry(index,row){
   
                let self = this;
                let listingId=row.id;
                 self.$axios.post(self.url1,{"listingId":listingId}).then((res) => {
                   if(res.data.result=="0"){
                          this.$message.success('重试成功！');                                                                                             
                   }else if( res.data.result=='-99' || res.data.result=='-999'){
                            self.$message.error(res.data.resultMessage);
                    }else{
                        self.$message.error(res.data.resultMessage);
                    }
                      
                })

           } ,
           CheckCode(type){
             var statusName;
            if(type==70 || type==72){
                statusName="提现失败";
            }else if(type==54 || type==41){
                statusName="转账失败";
            }else if(type==53){
                statusName="生成债务债权失败";
            }else if(type==42 || type==33){
                statusName="满标失败";
            }
             return statusName;        
            },
          timetrans:function(row, column) {  
               var date = row[column.property];  
                  var date = new Date(date);//如果date为13位不需要乘1000
                  if (date == undefined) {  
                      return "";  
                  }    
            var Y = date.getFullYear() + '-';
            var M = (date.getMonth()+1 < 10 ? '0'+(date.getMonth()+1) : date.getMonth()+1) + '-';
            var D = (date.getDate() < 10 ? '0' + (date.getDate()) : date.getDate()) + ' ';
            var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
            var m = (date.getMinutes() <10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
            var s = (date.getSeconds() <10 ? '0' + date.getSeconds() : date.getSeconds());
            return Y+M+D+h+m+s;       
          }
        }
    }
</script>

<style scoped>
.handle-box{
    margin-bottom: 20px;
}
.handle-select{
    width: 120px;
}
.handle-input{
    width: 150px;
    display: inline-block;
}

.radio{
display: inline-block;
  margin-left: 50px;
  font-style: normal;
}
  .demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
  .title{
      padding: 1% 0;
      font-size: 14px;
      font-weight: bold;
  }
  .ellipsis{
      overflow: hidden; 
      text-overflow:ellipsis;
       white-space: nowrap;
  }
</style>
<style>
.el-dialog{
    width: 90% !important;
}
</style>
