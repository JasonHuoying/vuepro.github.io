<style lang="less">
    .pagination {
        margin-top: 10px;
        text-align: right;
        position: relative;
    }
    .ivu-upload {
        display: inline-block !important;
    }
    .table {
        margin-top: 10px;
    }
    .ivu-row {
        font-size: 14px;
        margin: 5px 0;
    }
    .search-input {
        float: right;
        width: 350px;
    }
    .back-project-list {
        position: absolute;
        left: 0;
        top: 0;
    } // .ivu-upload-list-file{
    //     float:left;
    // }
</style>
<template>
    <div>
        <!-- <head-top></head-top> -->
        <div style="width:100%">
            <!-- <h2 class="project-title" style="display:inline-block">文档库</h2> -->
            <!--顶部新增和搜索-->
            <div class="top-action" style="margin-bottom:0">
                <span class="project-title" style="margin:10px 50px 0 20px;font-size:26px">文档库</span>
                <Button @click="addShow()" icon="plus-round">新增文件夹</Button>
                <Upload multiple :show-upload-list="false" :headers="{'token':user}" 
                :data="modelType" 
                :before-upload="handleBeforeUpload" 
                :on-success="handleSuccess" 
                action="http://122.144.199.68:9002/docfiles/singlefile/">
                    <Button type="ghost" icon="ios-cloud-upload-outline">文件上传</Button>
                </Upload>
                <!--<div class="search-input">
                            <Input v-model="searchValue" placeholder="请输入中心名称搜索">
                            <Button slot="append" icon="ios-search" @click="initList">搜索</Button>
                            </Input>
                        </div>-->
            </div>
        </div>
        <!--table-->
        <div class="table">
            <Table ref="currentRowTable" stripe :columns="columns" :loading="loading" :height="height" :data="data"></Table>
        </div>
        <!--pagination-->
        <div class="pagination">
            <Page :total="total" show-sizer :page-size="pageSize" @on-page-size-change="pageSizeChange" placement="top" show-total @on-change="pageChange"></Page>
            <Button class="back-project-list" style="margin-top:0" @click="backProjectList">返回列表</Button>
        </div>
        <!-- 新增文件夹 -->
        <div class="add-project">
            <Modal v-model="addVisible" :mask-closable="false" :closable="false" id="isModify" :title="titleName" :loading="addLoading">
                <div slot="footer">
                    <Button type="text" size="large" @click="cancel">取消</Button>
                    <Button type="primary" size="large" @click="ok('formValidate')">确定</Button>
                </div>
                <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="100">
                    <FormItem label="文件夹名称" prop="name">
                        <Input v-model="formValidate.name" type="text" placeholder="请输入文件夹名称"></Input>
                    </FormItem>
                </Form>
            </Modal>
        </div>
    </div>
</template>
<script>
    import {
        getDocuments,
        getDocumentsFolder,
        delDocuments,
        delFile,
        setDocumentsFolder,
        downloadFileId
    } from '../../libs/getData';
    // import headTop from "../common/headTop.vue";
    export default {
        data() {
            return {
                data: [], //table数据
                loading: true, //table的loading
                addLoading: true, //添加中心的loading
                total: 0, //总数
                modelType:{
                     folderId:'',
                     fileName:''
                },
                pageNum: 1, //页码
                pageSize: 10, //每页数量
                height: document.documentElement.clientHeight - 190, //table高度
                searchValue: '', //搜索
                addVisible: false, //添加的显示隐藏
                columns: [{
                        type: 'index',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '文件/文件夹名称',
                        key: 'name'
                    },
                    {
                        title: '操作',
                        key: 'action',
                        width: 230,
                        render: (h, params) => {
                            return h('div', [
                                params.row.type == '1' ?
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '20px',
                                        width: '76px'
                                    },
                                    on: {
                                        click: () => {
                                            this.$router.push(`/index/folder/${this.$route.params.projectId}/${params.row.vid}`);
                                        }
                                    }
                                }, '进入文件夹') :
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '20px',
                                        width: '76px'
                                    },
                                    on: {
                                        click: () => {
                                            //
                                        }
                                    }
                                }, '查看文件'),
                                params.row.type == '1' ? null :
                                h('a', {
                                        // props: {
                                        //     type: 'default',
                                        //     size: 'small'
                                        // },
                                        domProps:{
                                            href:`http://122.144.199.68:9002/docfiles/download?fileId=${params.row.vid}`,
                                            download:"",
                                            class:"ivu-btn ivu-btn-default ivu-btn-small"
                                        },
                                        style: {
                                            marginRight: '5px',
                                            display:'inline-block',
                                            padding: '2px 7px',
                                            fontSize: '12px',
                                            borderRadius: '3px',
                                            color: '#fff',
                                            backgroundColor: '#2d8cf0',
                                            borderColor:'#2d8cf0',
                                            marginBottom: '0',
                                            fontWeight: '400',
                                             textAlign: 'center',
                                            verticalAlign: 'middle',
                                           
                                            touchAction: 'manipulation',
                                            backgroundImage: 'none',
                                            border: '1px solid transparent',
                                            whiteSpace: 'nowrap',
                                            lineHeight: '1.5',
                                        },
                                        on: {
                                            // click: () => {
                                            //      //this.downloadFile(params.row);
                                            //     //  window.open(`http://122.144.199.68:9002/docfiles/download?fileId=${params.row.vid}`);
                                            // }
                                        }
                                    },
                                    '下载'
                                ),
                                h('Button', {
                                        props: {
                                            type: 'default',
                                            size: 'small'
                                        },
                                        style: {
                                            marginRight: '5px',
                                            color: '#fff',
                                            backgroundColor: 'red'
                                        },
                                        on: {
                                            click: () => {
                                                if (params.row.type == '1') {
                                                    this.deldocument(params.row);
                                                }else{
                                                    this.delFileId(params.row);
                                                }
                                            }
                                        }
                                    },
                                    '删除'
                                ),
                            ]);
                        }
                    }
                ],
                formValidate: { //新增，修改的值
                    name: '',
                    vid: '',
                    type: ''
                },
                ruleValidate: { //验证规则
                    name: [{
                        required: true,
                        message: '文件名称不能为空',
                        trigger: 'blur'
                    }],
                },
                viewVisible: false, //查看的显示隐藏
                viewLoading: false,
                titleName: "",
                user: ""
            }
        },
        methods: {
            initList() { //初始化表格
                this.loading = true;
                getDocuments({
                    pageSize: this.pageSize,
                    pageNum: this.pageNum,
                    projectId: this.$route.params.projectId,
                }, (res) => {
                    this.data = res.data.docBaseInfoBOList;
                    console.log(res.data)
                    console.log(this.$store)
                    this.user = this.$store.state.token;
                    this.formValidate.vid = res.data.id;
                    this.modelType.folderId=res.data.id;
                    this.total = res.data.docBaseInfoBOList.length;
                    this.loading = false;
                })
            },
            addShow() {
                this.isModify = true;
                this.titleName = "新增文件夹";
                this.addVisible = true;
            },
            ok(name) { //新增提交
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        if (this.isModify) {
                            console.log(this.formValidate);
                            setDocumentsFolder({
                                projectId: this.$route.params.projectId,
                                folderName: this.formValidate.name,
                                vid: this.formValidate.vid,
                            }, (res) => {
                                console.log(this.formValidate.vid)
                                this.$Message.success('新增成功!');
                                this.addVisible = false;
                                this.initList();
                            });
                        }
                    }
                });
            },
            cancel() { //新增取消
                this.addVisible = false;
                this.initList();
            },
            deldocument(params) {
                this.$Modal.confirm({
                    title: "确认删除？",
                    content: "确定要删除该文件夹？",
                    onOk: () => {
                        delDocuments({
                            projectId: this.$route.params.projectId,
                            vid: params.vid,
                        }, (res) => {
                            this.$Message.success('删除成功!');
                            this.initList();
                        });  
                    }
                });
            },
            delFileId(params) {
                this.$Modal.confirm({
                    title: "确认删除？",
                    content: "确定要删除该文件？",
                    onOk: () => {
                        delFile({
                            vid: params.vid,
                        }, (res) => {
                            this.$Message.success('删除成功!');
                            this.initList();
                        });
                    }
                });
            },
            
            downloadFile(params) {
                this.$Modal.confirm({
                    title: "确认下载？",
                    content: "确定下载该文件？",
                    onOk: () => {
                        downloadFileId({
                            fileId: params.vid,
                        }, (res) => {
                            window.onload(res.data);
                             this.$Message.success('下载成功!');
                            // this.initList();
                        });
                    }
                });
            },
            pageChange(page) { //页码变化
                this.pageNum = page;
                this.initList();
            },
            pageSizeChange(value) { //每页条数变化
                this.pageSize = value;
                this.pageNum = 1;
                this.initList();
            },
            backProjectList() {
                this.$router.push("/project");
            },
            handleSuccess(res, file) {
                
                if(res.success==false){
                    this.$Message.error(res.data);
                }else{
                   this.$Message.success('上传成功!');
                    this.initList();
                }
            },
            handleBeforeUpload(file) {               
                this.modelType.fileName = file.name;
            }
        },
        mounted() {
            this.initList();
        },
        components: {
            //  'head-top': headTop,
        },
    }
</script>
