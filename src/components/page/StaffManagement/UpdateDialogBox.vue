<template lang="pug">
    section
        el-dialog(title="员工信息修改" :visible.sync="showDialog" width="70%" :show-close="false")
            el-form(:model="tmpStaffInfo" size="small" label-position="right"  label-width="80px")
                el-row(:gutter="20")
                    el-col(:span="8")
                        el-form-item(label="姓名")
                            el-input(v-model="tmpStaffInfo.employee_name")
                    el-col(:span="8")
                        el-form-item(label="职位")
                            el-cascader(
                            expand-trigger="hover"
                            :options="postCascader"
                            v-model="defaultPostMapDepartment"
                            @change="handleChange"
                            separator="-"
                            clearable
                            filterable
                            :props="valueName")
                    el-col(:span="8")
                        el-form-item(label="联系方式")
                            el-input(v-model="tmpStaffInfo.employee_phone")
                el-row(:gutter="20")
                    el-col(:span="8")
                        el-form-item(label="月薪")
                            el-input(v-model="tmpStaffInfo.employee_salary")
                    el-col(:span="8")
                        el-form-item(label="学历")
                            el-select(v-model="tmpStaffInfo.employee_edu" placeholder="请选择学历")
                                el-option(v-for="(value, key, index) in edu" :key="key" :label="value" :value="value")
                    el-col(:span="8")
                        el-form-item(label="专业")
                            el-input(v-model="tmpStaffInfo.employee_professional")
                el-row(:gutter="20")
                    el-col(:span="8")
                        el-form-item(label="入职时间")
                            el-date-picker(style="width:100%"
                            v-model="tmpStaffInfo.employee_entry_time"
                            type="date" placeholder="请选择时间" ref="birthdayDom")
                    el-col(:span="8")
                        el-form-item(label="工作年限")
                            el-input(v-model="tmpStaffInfo.employee_work_seniority")
                    el-col(:span="8")
                        el-form-item(label="身份证")
                            el-input(v-model="tmpStaffInfo.employee_identity_card_number" clearable)
                el-row(:gutter="20")
                    el-col(:span="8")
                        el-form-item(label="权限")
                            el-select(v-model="tmpStaffInfo.permissions_id")
                                el-option(v-for="item in permissionsSelect" :key="item.value" :label="item.label" :value="item.value")
                    el-col(:span="16")
                        el-form-item(label="住址")
                            el-input(v-model="tmpStaffInfo.employee_address" clearable)
            div(slot="footer" class="dialog-footer")
                el-button(size="small" @click="showDialog=false") 取 消
                el-button(type="primary" size="small" @click="submitChange") 确 定
</template>
<script>
    import { mapState } from 'vuex'
    export default {
        computed: {
            ...mapState([
                'allStaffInfo',
                'postInfo',
                'departmentInfo',
                'updateStaffRes',
                'postCascader',
                'postMapDepartmentInfo',
                'edu',
                'permissionsSelect',
                'valueName'
            ]),
//            getDepartmentIDByPost(){
//                for (let departmentValue of this.departmentInfo){
//                    if(departmentValue.department_id===this.tmpStaffInfo.department_id){
//                        return departmentValue.department_name;
//                    }
//                }
//            }
        },
        data() {
            return {
                defaultPostMapDepartment:[],
                showDialog: false,
                departmentID:'',
                confirmedStaffInfo:{},
                tmpStaffInfo:{},
            };
        },
        methods:{
            editStaffInfo(row){
                this.showDialog=true;
                for(let key in row){
                    //直接赋值是，vue不能监听到对象的增删，用this.$set(obj,key,val)方法；
                    this.$set(this.confirmedStaffInfo,key,row[key]);
                    this.$set(this.tmpStaffInfo,key,row[key]);
                }
                this.defaultPostMapDepartment = (this.$utils.defaultCascaderArr(this.tmpStaffInfo.association_id,this.postMapDepartmentInfo)).reverse();
            },
            handleChange(val){
                //console.log(val);
            },
            async submitChange(){
                this.showDialog=false;
                this.tmpStaffInfo["association_id"] = this.defaultPostMapDepartment[this.defaultPostMapDepartment.length-1];
                let checkoutPostBool = '';
                for(let val of this.postMapDepartmentInfo){
                    if(this.tmpStaffInfo["association_id"]===val.association_id){
                        checkoutPostBool = val.association_is_department;
                    }
                }
                //console.log(this.tmpStaffInfo);
                if(checkoutPostBool===1){
                    this.$notify.error({
                        title: '职位选择错误！',
                        message: '您选择的是部门，请重新选择，或至-部门管理-修改属性为职位！',
                        duration:6000,
                    });
                }else{
                    //console.log(this.tmpStaffInfo);
                    //defaultPostMapDepartment,职位绑定的值赋给tmpStaffInfo完成格式化
                    await this.$store.dispatch(this.$types.UPDATE_STAFF,this.tmpStaffInfo);
                    await this.$store.dispatch(this.$types.GET_ALL_STAFF_INFO);
                    if(this.updateStaffRes.statusCode==='200220'){
                        this.$notify.success({
                            title: '更新成功！',
                            duration:2000,
                        });
                    }else if(this.updateStaffRes.statusCode==='400221'){
                        this.$notify.error({
                            title: '没有要更新的值！',
                            duration:2000,
                        });
                    }else{
                        this.$notify.error({
                            title: '其他错误！',
                            message: this.updateStaffRes,
                            duration:6000,
                        });
                    }
                }
            }
        },
        watch:{
        },
        //异步请求
        mounted(){
        }
    };
</script>
<style scoped>
    .el-select,.el-cascader{
        width:100%;
    }
</style>
