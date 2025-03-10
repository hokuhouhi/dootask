<template>
    <ETooltip
        v-if="user"
        class="common-avatar"
        :open-delay="600"
        :disabled="tooltipDisabled"
        :placement="tooltipPlacement">
        <div slot="content" class="common-avatar-transfer">
            <slot/>
            <p>{{$L('昵称')}}: {{user.nickname}}</p>
            <p>{{$L('职位/职称')}}: {{user.profession || '-'}}</p>
            <div v-if="userId != userid && showIconMenu" class="avatar-icons">
                <Icon type="ios-chatbubbles" @click="openDialog"/>
            </div>
        </div>
        <div class="avatar-wrapper">
            <div v-if="showIcon" :class="['avatar-box', userId === userid || user.online ? 'online' : '']" :style="boxStyle">
                <em :style="spotStyle"></em>
                <EAvatar v-if="showImg" :class="{'avatar-default':isDefault}" :src="user.userimg" :size="avatarSize"/>
                <EAvatar v-else :size="avatarSize" class="avatar-text">
                    <span :style="spotStyle">{{nickname}}</span>
                </EAvatar>
            </div>
            <div v-if="showName" class="avatar-name" :style="nameStyle">{{user.nickname}}</div>
        </div>
    </ETooltip>
</template>

<script>
    import {mapState} from "vuex";
    export default {
        name: 'UserAvatar',
        props: {
            userid: {
                type: [String, Number],
                default: ''
            },
            size: {
                type: [String, Number],
                default: 'default'
            },
            showIcon: {
                type: Boolean,
                default: true
            },
            showName: {
                type: Boolean,
                default: false
            },
            tooltipDisabled: {
                type: Boolean,
                default: false
            },
            showIconMenu: {
                type: Boolean,
                default: false
            },
            tooltipPlacement: {
                type: String,
                default: 'bottom'
            },
            borderWitdh: {
                type: Number,
                default: 0
            },
            borderColor: {
                type: String,
                default: ''
            },
            asynch: {
                type: Boolean,
                default: true
            },
        },
        data() {
            return {
                user: null
            }
        },
        mounted() {
            this.getData()
        },
        computed: {
            ...mapState(["userId", "userInfo", "userOnline"]),

            boxStyle() {
                const style = {};
                const {borderWitdh, borderColor} = this
                if (borderWitdh > 0) {
                    style.border = borderWitdh + "px solid " + (borderColor || "#ffffff");
                }
                return style;
            },

            spotStyle() {
                let {borderWitdh, size} = this
                if (size === 'default') size = 32;
                if (borderWitdh > 0) size-= borderWitdh;
                if (size == 32) {
                    return {}
                }
                return {
                    'transform': 'scale(' + Math.min(1.25, size / 32) + ')',
                }
            },

            nameStyle() {
                const {showIcon} = this;
                if (!showIcon) {
                    return {
                        paddingLeft: 0
                    }
                } else {
                    return {}
                }
            },

            avatarSize() {
                let {borderWitdh, size} = this
                if (size === 'default') size = 32;
                if (borderWitdh > 0) {
                    return size - borderWitdh * 2;
                } else {
                    return size;
                }
            },

            showImg() {
                const {userimg} = this.user
                if (!userimg) {
                    return false;
                }
                return !$A.rightExists(userimg, '/avatar.png');
            },

            isDefault() {
                const {userimg} = this.user
                return $A.strExists(userimg, '/avatar/default_');
            },

            nickname() {
                const {nickname} = this.user;
                if (!nickname) {
                    return "D";
                }
                let value = nickname.substring(0, 2);
                if (/^[\u4e00-\u9fa5]+$/.test(value)) {
                    value = value.substring(0, 1);
                }
                return value || 'D';
            }
        },
        watch: {
            userid() {
                this.getData()
            },

            userInfo(info) {
                if (info.userid == this.userid) {
                    this.user = info;
                }
            },

            userOnline(data) {
                if (this.user && typeof data[this.user.userid] !== "undefined") {
                    this.$set(this.user, 'online', data[this.user.userid]);
                }
            }
        },
        methods: {
            getData() {
                if (!this.userid) {
                    return;
                }
                if (this.userid == this.userInfo.userid) {
                    this.user = this.userInfo;
                    return;
                }
                if (this.asynch) {
                    setTimeout(this.loadData);
                } else {
                    this.loadData();
                }
            },

            loadData() {
                this.$store.dispatch("getUserBasic", {
                    userid: this.userid,
                    success: (user) => {
                        this.user = user;
                    }
                });
            },

            openDialog() {
                this.goForward({path: '/manage/messenger'});
                this.$store.dispatch("openDialogUserid", this.userid);
            }
        }
    };
</script>
