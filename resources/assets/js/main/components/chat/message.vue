<template>
    <div :data-id="info.id">

        <!--文本、任务、报告、视频通话-->
        <div v-if="info.type==='text' || info.type==='taskB' || info.type==='report' || info.type==='video' || info.type==='voice'">
            <div v-if="info.self===true" class="list-right">
                <div v-if="info.error" class="item-error" @click="clickError(info.error)">
                    <Icon type="md-alert" />
                </div>
                <div class="item-right">
                    <div class="item-username" @click="clickUser">
                        <em class="item-name"><user-view :username="userName" :info="info" placement="left"/></em>
                        <em v-if="info.indate" class="item-date">{{formatCDate(info.indate)}}</em>
                    </div>
                    <div class="item-text" :class="{'text-emoji':textIsEmoji(info.text), 'text-error':info.error}">
                        <div class="item-text-view">{{textMsg(info.text)}}</div>
                    </div>
                    <template v-if="info.type==='taskB'">
                        <div v-if="info.other.type==='task'" class="item-link" @click="taskDetail(info.other.id)"><span>{{$L('来自关注任务')}}:</span><a href="javascript:void(0)">{{info.other.title}}</a></div>
                        <div v-if="info.other.type==='file'" class="item-link"><span>{{$L('来自关注任务')}}:</span><a target="_blank" :href="fileDownUrl(info.other.id)">{{info.other.name}}</a></div>
                    </template>
                    <div v-else-if="info.type==='report'" class="item-link" @click="reportDetail(info.other.id, info.other.title)"><span>{{$L('来自工作报告')}}:</span><a href="javascript:void(0)">{{info.other.title}}</a></div>
                    <div v-else-if="info.type==='video' || info.type==='voice'" class="item-link">
                        <Icon v-if="info.type==='voice'" type="ios-call-outline"/>
                        <Icon v-else type="ios-videocam-outline"/>
                        <span>{{$L('通话时长：%', formatSecond(info.other.second))}}</span>
                    </div>
                </div>
                <UserImg :info="info" @click="clickUser" class="item-userimg"/>
            </div>
            <div v-else-if="info.self===false" class="list-item">
                <UserImg :info="info" @click="clickUser" class="item-userimg"/>
                <div class="item-left">
                    <div class="item-username" @click="clickUser">
                        <em class="item-name"><user-view :username="userName" :info="info" placement="right"/></em>
                        <em v-if="info.__usertag" class="item-tag">{{info.__usertag}}</em>
                        <em v-if="info.indate" class="item-date">{{formatCDate(info.indate)}}</em>
                    </div>
                    <div class="item-text" :class="{'text-emoji':textIsEmoji(info.text), 'text-error':info.error}">
                        <div class="item-text-view">{{textMsg(info.text)}}</div>
                    </div>
                    <template v-if="info.type==='taskB'">
                        <div v-if="info.other.type==='task'" class="item-link" @click="taskDetail(info.other.id)"><span>{{$L('来自关注任务')}}:</span><a href="javascript:void(0)">{{info.other.title}}</a></div>
                        <div v-if="info.other.type==='file'" class="item-link"><span>{{$L('来自关注任务')}}:</span><a target="_blank" :href="fileDownUrl(info.other.id)">{{info.other.name}}</a></div>
                    </template>
                    <div v-else-if="info.type==='report'" class="item-link" @click="reportDetail(info.other.id, info.other.title)"><span>{{$L('来自工作报告')}}:</span><a href="javascript:void(0)">{{info.other.title}}</a></div>
                    <div v-else-if="info.type==='video' || info.type==='voice'" class="item-link">
                        <Icon v-if="info.type==='voice'" type="ios-call-outline"/>
                        <Icon v-else type="ios-videocam-outline"/>
                        <span>{{$L('通话时长：%', formatSecond(info.other.second))}}</span>
                    </div>
                </div>
            </div>
        </div>

        <!--图片-->
        <div v-else-if="info.type==='image'">
            <div v-if="info.self===true" class="list-right">
                <div v-if="info.error" class="item-error" @click="clickError(info.error)">
                    <Icon type="md-alert" />
                </div>
                <div class="item-right">
                    <div class="item-username" @click="clickUser">
                        <em class="item-name"><user-view :username="userName" :info="info" placement="left"/></em>
                        <em v-if="info.indate" class="item-date">{{formatCDate(info.indate)}}</em>
                    </div>
                    <a class="item-image" :href="info.url" target="_blank">
                        <img class="item-image-view" :src="info.url"/>
                    </a>
                </div>
                <UserImg :info="info" @click="clickUser" class="item-userimg"/>
            </div>
            <div v-else-if="info.self===false" class="list-item">
                <UserImg :info="info" @click="clickUser" class="item-userimg"/>
                <div class="item-left">
                    <div class="item-username" @click="clickUser">
                        <em class="item-name"><user-view :username="userName" :info="info" placement="right"/></em>
                        <em v-if="info.__usertag" class="item-tag">{{info.__usertag}}</em>
                        <em v-if="info.indate" class="item-date">{{formatCDate(info.indate)}}</em>
                    </div>
                    <a class="item-image" :href="info.url" target="_blank">
                        <img class="item-image-view" :src="info.url"/>
                    </a>
                </div>
            </div>
        </div>

        <!--通知-->
        <div v-else-if="info.type==='notice'">
            <div class="item-notice">{{info.notice}}</div>
        </div>

    </div>
</template>

<style lang="scss" scoped>
    /*通用*/
    .list-item, .list-right {
        display: flex;
        width: 100%;
        padding-top: 7px;
        padding-bottom: 7px;
        background-color: #E8EBF2;
        .item-left, .item-right {
            display: flex;
            flex-direction: column;
            max-width: 80%;
            .item-username {
                font-size: 12px;
                padding-top: 1px;
                padding-bottom: 4px;
                display: flex;
                flex-direction: row;
                align-items: center;
                em {
                    display: inline-block;
                    font-style: normal;
                    &.item-name {
                        color: #888888;
                    }
                    &.item-tag {
                        color: #ffffff;
                        background-color: #ff0000;
                        line-height: 16px;
                        padding: 2px 4px;
                        margin-left: 3px;
                        border-radius: 2px;
                        font-size: 12px;
                        transform: scale(0.8);
                        font-weight: 600;
                    }
                    &.item-date {
                        margin-left: 4px;
                        color: #aaaaaa;
                    }
                }
            }
        }
        .item-left {
            align-items: flex-start;
        }
        .item-right {
            align-items: flex-end;
            .item-username {
                text-align: right;
            }
            .item-link {
                transform-origin: right center;
            }
        }
        .item-userimg {
            width: 38px;
            height: 38px;
            margin-left: 8px;
            margin-right: 8px;
            border-radius: 3px;
            font-size: 20px;
        }
        .item-error {
            cursor: pointer;
            width: 48px;
            position: relative;
            > i {
                color: #ff0000;
                font-size: 18px;
                position: absolute;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
            }
        }
    }

    .list-right {
        justify-content: flex-end;
    }

    /*文本*/
    .item-text {
        display: inline-block;
        border-radius: 6px;
        padding: 8px;
        background-color: #ffffff;
        max-height: 580px;
        overflow: auto;
        &.text-emoji {
            background-color: transparent;
            .item-text-view {
                font-size: 52px;
                line-height: normal;
            }
        }
        &.text-error {
            box-shadow: 0 0 4px 0 #ffa1a1;
        }
        .item-text-view {
            max-width: 520px;
            color: #242424;
            font-size: 14px;
            line-height: 18px;
            word-break: break-all;
        }
    }

    /*信息底标*/
    .item-link {
        display: flex;
        align-items: center;
        max-width: 100%;
        font-size: 12px;
        color: #ffffff;
        background-color: #cacaca;
        margin-top: 6px;
        margin-bottom: -2px;
        height: 20px;
        line-height: 20px;
        padding: 0 5px;
        border-radius: 4px;
        transform: scale(0.96);
        transform-origin: left center;
        > i {
            font-size: 14px;
            padding-right: 2px;
        }
        > span {
            white-space: nowrap;
        }
        > a {
            color: #3D90E2;
            padding-left: 3px;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            max-width: 100%;
        }
    }

    /*图片*/
    .item-image {
        display: inline-block;
        text-decoration: none;
        .item-image-view {
            max-width: 220px;
            max-height: 220px;
            border-radius: 6px;
        }
    }

    /*通知*/
    .item-notice {
        color: #777777;
        font-size: 12px;
        text-align: center;
        padding: 12px 24px;
    }
</style>

<script>

    export default {
        name: 'ChatMessage',
        props: {
            info: {
                type: Object,
                default: {},
            },
        },

        mounted() {

        },

        computed: {
            userName() {
                return this.info.send_username || this.info.username;
            },

            userImg() {
                return this.info.send_userimg || this.info.userimg;
            },
        },

        methods: {
            textMsg(text) {
                return (text + "").replace(/\n/, '<br/>');
            },

            textIsEmoji(text) {
                return text.replace(/[\uD800-\uDBFF][\uDC00-\uDFFF]/g, "_") === "_";
            },

            formatCDate(v) {
                let string = '';
                if ($A.runNum(v) > 0) {
                    if ($A.formatDate('Ymd') === $A.formatDate('Ymd', v)) {
                        string = $A.formatDate('H:i', v)
                    } else if ($A.formatDate('Y') === $A.formatDate('Y', v)) {
                        string = $A.formatDate('m-d', v)
                    } else {
                        string = $A.formatDate('Y-m-d', v)
                    }
                }
                return string ? '(' + string + ')' : '';
            },

            clickError(err) {
                this.$Modal.error({
                    title: this.$L("错误详情"),
                    content: err
                });
            },

            clickUser(e) {
                this.$emit('clickUser', this.info, e);
            },

            fileDownUrl(id) {
                return $A.apiUrl('project/files/download?fileid=' + id);
            },

            formatSecond(d) {
                if (d > 3600) {
                    return Math.ceil(d / 3600) + '小时';
                } else if (d > 60) {
                    return Math.ceil(d / 60) + '分钟';
                } else {
                    return d + '秒';
                }
            }
        }
    }
</script>
