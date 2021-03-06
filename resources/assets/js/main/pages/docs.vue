<template>
    <div class="w-main docs">

        <v-title>{{$L('知识库')}}-{{$L('轻量级的团队在线协作')}}</v-title>

        <div class="w-nav">
            <div class="nav-row">
                <div class="w-nav-left">
                    <div class="page-nav-left">
                        <span class="hover" @click="[addBookId=0,addBookShow=true]"><i class="ft icon">&#xE740;</i> {{$L('新建知识库')}}</span>
                        <div v-if="loadIng > 0" class="page-nav-loading"><w-loading></w-loading></div>
                        <div v-else class="page-nav-refresh"><em @click="getBookLists(true)">{{$L('刷新')}}</em></div>
                    </div>
                </div>
                <div class="w-nav-flex"></div>
            </div>
        </div>

        <w-content>
            <div class="docs-main">
                <div class="docs-body">
                    <div class="docs-menu">
                        <h3>{{$L('我的知识库')}}</h3>
                        <ul>
                            <li v-for="book in bookLists" :class="{active:book.id==selectBookData.id}" @click="[selectBookData=book,getSectionLists(true)]">
                                <div class="docs-title">{{book.title}}</div>
                                <div class="docs-time">{{$A.formatDate("Y-m-d H:i:s", book.indate)}}</div>
                            </li>
                            <li v-if="bookHasMorePages" class="more" @click="getBookLists">{{$L('加载下一页...')}}</li>
                            <li v-else-if="bookLoading" class="load"><WLoading/></li>
                            <li v-else-if="bookLists.length == 0" class="none">{{bookNoDataText}}</li>
                        </ul>
                    </div>
                    <div class="docs-container">
                        <div v-if="selectBookData.id > 0" class="docs-box">
                            <div class="docs-header">
                                <div class="docs-h1">{{selectBookData.title}}</div>
                                <div class="docs-setting">
                                    <Button @click="[addSectionId=0,addSectionShow=true]">{{$L('新增章节')}}</Button>
                                    <Button @click="[addBookId=selectBookData.id,addBookShow=true]">{{$L('修改标题')}}</Button>
                                    <Button @click="showShare">{{$L('分享')}}</Button>
                                    <Button @click="[settingDrawerShow=true,settingDrawerTab='setting']">{{$L('设置')}}</Button>
                                    <Button type="warning" ghost @click="onBookDelete(selectBookData.id)">{{$L('删除')}}</Button>
                                </div>
                            </div>
                            <div class="docs-section">
                                <nested-draggable :lists="sectionLists" :disabled="sortDisabled" @change="handleSection"></nested-draggable>
                                <div v-if="sectionLists.length == 0" class="none">{{sectionNoDataText}}</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </w-content>

        <Modal
            v-model="addBookShow"
            :title="$L(addBookId > 0 ? '修改标题' : '新建知识库')"
            :closable="false"
            :mask-closable="false">
            <Form ref="bookAdd" :model="formBookAdd" :rules="ruleBookAdd" :label-width="110">
                <FormItem prop="title" :label="$L('知识库名称')" style="margin-right:28px">
                    <Input type="text" v-model="formBookAdd.title" :maxlength="32"></Input>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="default" @click="addBookShow=false">{{$L('取消')}}</Button>
                <Button type="primary" :loading="loadIng > 0" @click="onBookAdd">{{$L(addBookId > 0 ? '提交' : '添加')}}</Button>
            </div>
        </Modal>

        <Modal
            v-model="addSectionShow"
            :title="$L(addSectionId > 0 ? '修改文档标题' : '新建文档')"
            :closable="false"
            :mask-closable="false">
            <Form ref="sectionAdd" :model="formSectionAdd" :rules="ruleSectionAdd" :label-width="110">
                <FormItem prop="title" :label="$L('文档标题')" style="margin-right:28px">
                    <Input type="text" v-model="formSectionAdd.title" :maxlength="32"></Input>
                </FormItem>
                <FormItem v-if="addSectionId <= 0" prop="type" :label="$L('文档类型')" style="margin-right:28px">
                    <ButtonGroup>
                        <Button v-for="(it, ik) in sectionTypeLists"
                                :key="ik"
                                :type="`${formSectionAdd.type==it.value?'primary':'default'}`"
                                @click="formSectionAdd.type=it.value">{{it.text}}</Button>
                    </ButtonGroup>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="default" @click="addSectionShow=false">{{$L('取消')}}</Button>
                <Button type="primary" :loading="loadIng > 0" @click="onSectionAdd">{{$L(addSectionId > 0 ? '提交' : '添加')}}</Button>
            </div>
        </Modal>

        <WDrawer v-model="settingDrawerShow" maxWidth="750">
            <Tabs v-if="settingDrawerShow" v-model="settingDrawerTab">
                <TabPane :label="$L('文档设置')" name="setting">
                    <book-setting :canload="settingDrawerShow && settingDrawerTab == 'setting'" :id="selectBookData.id"></book-setting>
                </TabPane>
                <TabPane :label="$L('文档成员')" name="member">
                    <book-users :canload="settingDrawerShow && settingDrawerTab == 'member'" :id="selectBookData.id"></book-users>
                </TabPane>
            </Tabs>
        </WDrawer>
    </div>
</template>

<style lang="scss" scoped>
    .docs {
        .docs-main {
            display: flex;
            flex-direction: column;
            width: 100%;
            min-width: 1024px;
            height: 100%;
            padding: 15px;
            .docs-body {
                display: flex;
                flex-direction: row;
                width: 100%;
                height: 100%;
                min-height: 500px;
                .docs-menu {
                    display: flex;
                    flex-direction: column;
                    width: 230px;
                    border-radius: 3px 0 0 3px;
                    background: rgba(255, 255, 255, 0.8);
                    h3 {
                        font-size: 18px;
                        font-weight: normal;
                        padding: 10px 12px;
                        color: #333333;
                    }
                    ul {
                        flex: 1;
                        overflow: auto;
                        li {
                            padding: 12px;
                            cursor: pointer;
                            &.more {
                                text-align: center;
                                color: #555555;
                                margin-bottom: 6px;
                                &:hover {
                                    color: #333333;
                                }
                            }
                            &.load {
                                text-align: center;
                                height: 42px;
                                margin-bottom: 9px;
                            }
                            &.none {
                                background-color: transparent;
                                text-align: center;
                                color: #666666;
                                padding: 8px 18px;
                            }
                            &.active {
                                background-color: #ffffff;
                            }
                            .docs-title {
                                color: #242424;
                                font-size: 13px;
                            }
                            .docs-time {
                                display: block;
                                color: #999;
                                font-size: 12px;
                                margin-top: 2px;
                                position: relative;
                            }
                        }
                    }
                }
                .docs-container {
                    flex: 1;
                    background-color: #ffffff;
                    border-radius: 0 3px 3px 0;
                    .docs-box {
                        width: 100%;
                        height: 100%;
                        display: flex;
                        flex-direction: column;
                    }
                    .docs-header {
                        display: flex;
                        align-items: center;
                        margin: 6px 24px 0;
                        padding: 12px 0;
                        border-bottom: 1px solid #eeeeee;
                        .docs-h1 {
                            flex: 1;
                            font-size: 16px;
                            white-space: nowrap;
                        }
                        .docs-setting {
                            display: flex;
                            align-items: center;
                            > button {
                                margin: 0 6px;
                                &:last-child {
                                    margin-right: 0;
                                }
                            }
                        }
                    }
                    .docs-section {
                        flex: 1;
                        padding: 12px 26px;
                        overflow: auto;
                        transform: translateZ(0);
                        .none {
                            background-color: transparent;
                            text-align: center;
                            color: #666666;
                            padding: 48px 24px;
                        }
                    }
                }
            }
        }
    }
</style>
<script>
    import WContent from "../components/WContent";
    import NestedDraggable from "../components/docs/NestedDraggable";
    import BookSetting from "../components/docs/setting";
    import BookUsers from "../components/docs/users";
    import WDrawer from "../components/iview/WDrawer";
    export default {
        components: {WDrawer, BookUsers, BookSetting, NestedDraggable, WContent},
        data () {
            return {
                loadIng: 0,

                userInfo: {},

                bookLists: [],
                bookListPage: 1,
                bookListTotal: 0,
                bookLoading: false,
                bookHasMorePages: false,
                bookNoDataText: "",

                addBookId: 0,
                addBookShow: false,
                formBookAdd: {
                    title: '',
                },
                ruleBookAdd: {},
                selectBookData: {},

                sectionLists: [],
                sectionNoDataText: "",

                addSectionId: 0,
                addSectionShow: false,
                formSectionAdd: {
                    title: '',
                    type: 'document',
                },
                ruleSectionAdd: {},
                sectionTypeLists: [],

                sortDisabled: false,

                settingDrawerShow: false,
                settingDrawerTab: 'setting',
            }
        },

        created() {
            this.bookNoDataText = this.$L("数据加载中.....");
            this.sectionNoDataText = this.$L("数据加载中.....");
            this.sectionTypeLists = [
                {value: 'document', text: this.$L("文本")},
                {value: 'mind', text: this.$L("脑图")},
                {value: 'sheet', text: this.$L("表格")},
                {value: 'flow', text: this.$L("流程图")},
                {value: 'folder', text: this.$L("目录")},
            ];
            this.ruleBookAdd = {
                title: [
                    { required: true, message: this.$L('请填写知识库名称！'), trigger: 'change' },
                    { type: 'string', min: 2, message: this.$L('知识库名称长度至少2位！'), trigger: 'change' }
                ],
            };
            this.ruleSectionAdd = {
                title: [
                    { required: true, message: this.$L('请填写文档标题！'), trigger: 'change' },
                    { type: 'string', min: 2, message: this.$L('文档标题长度至少2位！'), trigger: 'change' }
                ],
                type: [
                    { required: true },
                ],
            };
        },

        mounted() {
            this.getBookLists(true);
            this.userInfo = $A.getUserInfo((res, isLogin) => {
                if (this.userInfo.id != res.id) {
                    this.userInfo = res;
                    isLogin && this.getBookLists(true);
                } else {
                    this.userInfo = res;
                }
            }, false);
        },

        deactivated() {
            this.addBookShow = false;
            this.addSectionShow = false;
        },

        computed: {

        },

        watch: {
            addBookShow(val) {
                if (val && this.addBookId > 0) {
                    let tempLists = this.bookLists.filter((res) => { return res.id == this.addBookId });
                    if (tempLists.length === 1) {
                        this.$set(this.formBookAdd, 'title', tempLists[0].title);
                    } else {
                        this.$set(this.formBookAdd, 'title', '');
                    }
                }
            },
            addSectionShow(val) {
                if (val && this.addSectionId > 0) {
                    let tempLists = this.children2lists(this.sectionLists).filter((res) => { return res.id == this.addSectionId });
                    if (tempLists.length === 1) {
                        this.$set(this.formSectionAdd, 'title', tempLists[0].title);
                    } else {
                        this.$set(this.formSectionAdd, 'title', '');
                    }
                }
            }
        },

        methods: {
            children2lists(lists) {
                let array = [];
                lists.forEach((item) => {
                    array.push({
                        id: item.id,
                        title: item.title
                    });
                    array = array.concat(this.children2lists(item.children))
                });
                return array;
            },

            getBookLists(resetLoad) {
                if (resetLoad === true) {
                    this.bookListPage = 1;
                } else {
                    if (this.bookHasMorePages === false) {
                        return;
                    }
                    this.bookListPage++;
                }
                this.bookLoading = true;
                this.bookNoDataText = this.$L("数据加载中.....");
                this.bookHasMorePages = false;
                $A.apiAjax({
                    url: 'docs/book/lists',
                    data: {
                        page: Math.max(this.bookListPage, 1),
                        pagesize: 20,
                    },
                    complete: () => {
                        this.bookLoading = false;
                    },
                    error: () => {
                        this.bookNoDataText = this.$L("数据加载失败！");
                    },
                    success: (res) => {
                        if (res.ret === 1) {
                            res.data.lists.forEach((item) => {
                                let find = this.bookLists.find((t)=>{return t.id==item.id});
                                if (!find) {
                                    this.bookLists.push(item);
                                }
                            });
                            this.bookListTotal = res.data.total;
                            this.bookNoDataText = this.$L("没有相关的数据");
                            this.bookHasMorePages = res.data.hasMorePages;
                            if (typeof this.selectBookData.id === "undefined") {
                                this.selectBookData = this.bookLists[0];
                                this.getSectionLists();
                            }
                        } else {
                            this.bookLists = [];
                            this.bookListTotal = 0;
                            this.bookNoDataText = res.msg;
                        }
                    }
                });
            },

            onBookAdd() {
                this.$refs.bookAdd.validate((valid) => {
                    if (valid) {
                        this.loadIng++;
                        $A.apiAjax({
                            url: 'docs/book/add',
                            data: Object.assign(this.formBookAdd, {id:this.addBookId}),
                            complete: () => {
                                this.loadIng--;
                            },
                            success: (res) => {
                                if (res.ret === 1) {
                                    this.addBookShow = false;
                                    this.$Message.success(res.msg);
                                    this.$refs.bookAdd.resetFields();
                                    //
                                    if (this.addBookId > 0) {
                                        this.bookLists.some((item) => {
                                            if (item.id == this.addBookId) {
                                                this.$set(item, 'title', res.data.title);
                                                return true;
                                            }
                                        });
                                    } else {
                                        this.bookLists.unshift(res.data);
                                        this.selectBookData = this.bookLists[0];
                                        this.getSectionLists();
                                    }
                                }else{
                                    this.$Modal.error({title: this.$L('温馨提示'), content: res.msg });
                                }
                            }
                        });
                    }
                });
            },

            onBookDelete(bookId) {
                this.$Modal.confirm({
                    title: this.$L('删除知识库'),
                    content: this.$L('你确定要删除此知识库吗？'),
                    loading: true,
                    onOk: () => {
                        $A.apiAjax({
                            url: 'docs/book/delete',
                            data: {
                                id: bookId
                            },
                            error: () => {
                                this.$Modal.remove();
                                alert(this.$L('网络繁忙，请稍后再试！'));
                            },
                            success: (res) => {
                                this.$Modal.remove();
                                this.bookLists.some((item, index) => {
                                    if (item.id == bookId) {
                                        this.bookLists.splice(index, 1);
                                        return true;
                                    }
                                })
                                this.selectBookData = this.bookLists[0];
                                this.getSectionLists();
                                //
                                setTimeout(() => {
                                    if (res.ret === 1) {
                                        this.$Message.success(res.msg);
                                    } else {
                                        this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                    }
                                }, 350);
                            }
                        });
                    }
                });
            },

            getSectionLists(isClear) {
                if (isClear === true) {
                    this.sectionLists = [];
                }
                let bookid = this.selectBookData.id;
                this.loadIng++;
                this.sectionNoDataText = this.$L("数据加载中.....");
                $A.apiAjax({
                    url: 'docs/section/lists',
                    data: {
                        act: 'edit',
                        bookid: bookid
                    },
                    complete: () => {
                        this.loadIng--;
                    },
                    error: () => {
                        if (bookid != this.selectBookData.id) {
                            return;
                        }
                        this.sectionNoDataText = this.$L("数据加载失败！");
                    },
                    success: (res) => {
                        if (bookid != this.selectBookData.id) {
                            return;
                        }
                        if (res.ret === 1) {
                            this.sectionLists = res.data.tree;
                            this.sectionNoDataText = this.$L("没有相关的数据");
                        }else{
                            this.sectionLists = [];
                            this.sectionNoDataText = res.msg;
                        }
                    }
                });
            },

            onSectionAdd() {
                this.$refs.sectionAdd.validate((valid) => {
                    if (valid) {
                        this.loadIng++;
                        let bookid = this.selectBookData.id;
                        $A.apiAjax({
                            url: 'docs/section/add',
                            data: Object.assign(this.formSectionAdd, {
                                id: this.addSectionId,
                                bookid: bookid,
                            }),
                            complete: () => {
                                this.loadIng--;
                            },
                            success: (res) => {
                                if (bookid != this.selectBookData.id) {
                                    return;
                                }
                                if (res.ret === 1) {
                                    this.addSectionShow = false;
                                    this.$Message.success(res.msg);
                                    this.$refs.sectionAdd.resetFields();
                                    //
                                    this.getSectionLists();
                                }else{
                                    this.$Modal.error({title: this.$L('温馨提示'), content: res.msg });
                                }
                            }
                        });
                    }
                });
            },

            onSectionDelete(detail) {
                let sectionType = this.sectionTypeLists.find((item) => { return item.value == detail.type });
                this.$Modal.confirm({
                    title: this.$L('删除文档'),
                    content: this.$L('你确定要删除%【%】吗？', sectionType ? sectionType.text : '', detail.title),
                    loading: true,
                    onOk: () => {
                        $A.apiAjax({
                            url: 'docs/section/delete',
                            data: {
                                id: detail.id
                            },
                            error: () => {
                                this.$Modal.remove();
                                alert(this.$L('网络繁忙，请稍后再试！'));
                            },
                            success: (res) => {
                                this.$Modal.remove();
                                this.getSectionLists();
                                //
                                setTimeout(() => {
                                    if (res.ret === 1) {
                                        this.$Message.success(res.msg);
                                    } else {
                                        this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                    }
                                }, 350);
                            }
                        });
                    }
                });
            },

            handleSection(act, detail) {
                switch (act) {
                    case 'open':
                        this.goForward({name: 'docs-edit', params: {sid:detail.id, other:detail||{}}});
                        break;

                    case 'edit':
                        this.addSectionId = detail.id;
                        this.addSectionShow = true
                        break;

                    case 'add':
                        this.addSectionId = detail.id * -1;
                        this.addSectionShow = true
                        break;

                    case 'delete':
                        if (this.sortDisabled) {
                            this.$Modal.warning({
                                title: this.$L('温馨提示'),
                                content: this.$L('正在进行其他操作，请稍后重试...')
                            });
                            return;
                        }
                        this.onSectionDelete(detail);
                        break;

                    case 'sort':
                        this.sortDisabled = true;
                        this.loadIng++;
                        $A.apiAjax({
                            url: 'docs/section/sort',
                            data: {
                                bookid: this.selectBookData.id,
                                newsort: detail,
                            },
                            complete: () => {
                                this.sortDisabled = false;
                                this.loadIng--;
                            },
                            error: () => {
                                this.getSectionLists();
                                alert(this.$L('网络繁忙，请稍后再试！'));
                            },
                            success: (res) => {
                                if (res.ret === 1) {
                                    this.$Message.success(res.msg);
                                } else {
                                    this.getSectionLists();
                                    this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                }
                            }
                        });
                        break;
                }
            },

            showShare() {
                this.$Modal.confirm({
                    render: (h) => {
                        return h('div', [
                            h('div', {
                                style: {
                                    fontSize: '16px',
                                    fontWeight: '500',
                                    marginBottom: '20px',
                                }
                            }, this.$L('文档链接')),
                            h('Input', {
                                props: {
                                    value: $A.webUrl('docs/view/b' + this.selectBookData.id),
                                    readonly: true,
                                },
                            })
                        ])
                    },
                });
            }
        },
    }
</script>
