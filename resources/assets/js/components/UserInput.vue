<template>
    <div v-if="ready" :class="['common-user', maxHiddenClass]">
        <Select
            v-model="values"
            :transfer="transfer"
            :remote-method="searchUser"
            :placeholder="placeholder"
            :loading="loading"
            :loading-text="$L('加载中...')"
            :default-label="value"
            :default-event-object="true"
            :multipleMax="multipleMax"
            :multipleUncancelable="uncancelable"
            multiple
            filterable
            transfer-class-name="common-user-transfer"
            @on-open-change="openChange"
            @on-set-default-options="setDefaultOptions">
            <div v-if="multipleMax" slot="drop-prepend" class="user-drop-prepend">{{$L('最多只能选择' + multipleMax + '个')}}</div>
            <Option
                v-for="(item, key) in list"
                :value="item.userid"
                :key="key"
                :label="item.nickname"
                :avatar="item.userimg"
                :disabled="isDisabled(item.userid)">
                <div class="user-input-option">
                    <div class="user-input-avatar"><EAvatar class="avatar" :src="item.userimg"/></div>
                    <div class="user-input-nickname">{{ item.nickname }}</div>
                    <div class="user-input-userid">ID: {{ item.userid }}</div>
                </div>
            </Option>
        </Select>
        <div v-if="!initialized" class="common-user-loading"><Loading/></div>
    </div>
</template>

<script>
    export default {
        name: 'UserInput',
        props: {
            value: {
                type: [String, Number, Array],
                default: ''
            },
            uncancelable: {
                type: Array,
                default: () => {
                    return [];
                }
            },
            disabledChoice: {
                type: Array,
                default: () => {
                    return [];
                }
            },
            placeholder: {
                default: ''
            },
            transfer: {
                type: Boolean,
                default: true
            },
            multipleMax: {
                type: Number,
            },
            maxHiddenInput: {
                type: Boolean,
                default: true
            },
            projectId: {
                type: Number,
                default: 0
            },
            noProjectId: {
                type: Number,
                default: 0
            },
        },
        data() {
            return {
                ready: false,
                initialized: false,
                loading: false,
                openLoad: false,
                values: [],
                list: []
            }
        },
        mounted() {
            if ($A.isArray(this.value)) {
                this.values = $A.cloneJSON(this.value);
            } else {
                this.$emit('input', this.value ? [this.value] : []);
            }
            this.$nextTick(() => {
                this.ready = true;
            });
        },
        computed: {
            maxHiddenClass() {
                const {multipleMax, maxHiddenInput, values} = this;
                if (multipleMax && maxHiddenInput) {
                    if (values.length >= multipleMax) {
                        return 'hidden-input'
                    }
                }
                return '';
            }
        },
        watch: {
            value(val) {
                this.values = val;
            },
            values(val) {
                this.$emit('input', val);
            }
        },
        methods: {
            openChange(show) {
                if (show && !this.openLoad) {
                    this.openLoad = true;
                    if (this.list.length == this.values.length || this.list.length <= 1) {
                        this.$nextTick(this.searchUser);
                    }
                }
            },

            setDefaultOptions(options) {
                const userids = [];
                options.forEach(({value, label}) => {
                    this.list.push({
                        userid: value,
                        nickname: label,
                    });
                    userids.push(value);
                });
                //
                this.$store.dispatch("getUserBasic", {
                    userid: userids,
                    complete: () => {
                        this.initialized = true;
                    },
                    success: (user) => {
                        let option = options.find(({value}) => value == user.userid);
                        if (option) {
                            this.$set(option, 'label', user.nickname)
                            this.$set(option, 'avatar', user.userimg)
                        }
                        this.list.some((item, index) => {
                            if (item.userid == user.userid) {
                                this.$set(this.list, index, Object.assign(item, user));
                            }
                        });
                    }
                });
            },

            searchUser(query) {
                if (query !== '') {
                    this.loading = true;
                    this.$store.dispatch("call", {
                        url: 'users/search',
                        data: {
                            keys: {
                                key: query || '',
                                project_id: this.projectId,
                                no_project_id: this.noProjectId,
                            },
                            take: 30
                        },
                    }).then(({data}) => {
                        this.loading = false;
                        this.list = data;
                    }).catch(({msg}) => {
                        this.loading = false;
                        this.list = [];
                        $A.messageWarning(msg);
                    });
                } else {
                    this.list = [];
                }
            },

            isDisabled(userid) {
                if (this.disabledChoice.length === 0) {
                    return false;
                }
                return this.disabledChoice.includes(userid)
            }
        }
    };
</script>
