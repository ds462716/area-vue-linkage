<template>
    <div class="area-select-wrap">
        <v-select v-model="curProvinceCode" :placeholder="placeholders[0] ? placeholders[0] : '请选择'" :size="size" :disabled="disabled">
            <v-option v-for="(val, key) in provinces"
                :key="key"
                :label="val"
                :value="key">
            </v-option>
        </v-select>

         <v-select v-model="curCityCode" :placeholder="placeholders[1] ? placeholders[1] : '请选择'" v-if="level>=1 && isShow1" :size="size" :disabled="disabled">
            <p v-if="!Object.keys(citys).length" class="area-select-empty">暂无数据</p>
            <v-option v-else v-for="(val, key) in citys"
                :key="key"
                :label="val"
                :value="key">
            </v-option>
        </v-select>

        <v-select v-model="curAreaCode" :placeholder="placeholders[2] ? placeholders[2] : '请选择'" v-if="level>=2 && isShow2" :size="size" :disabled="disabled">
            <p v-if="!Object.keys(areas).length" class="area-select-empty">暂无数据</p>
            <v-option v-else v-for="(val, key) in areas"
                :key="key"
                :label="val"
                :value="key">
            </v-option>
        </v-select>

        <v-select v-model="curStreetCode" :placeholder="placeholders[3] ? placeholders[3] : '请选择'" v-if="level>=3 && isShow3" :size="size" :disabled="disabled">
            <p v-if="!Object.keys(streets).length" class="area-select-empty">暂无数据</p>
            <v-option v-else v-for="(val, key) in streets"
                      :key="key"
                      :label="val"
                      :value="key">
            </v-option>
        </v-select>

    </div>
</template>

<script>
    import find from 'lodash.find';

    import Select from './select/index.vue';
    import Option from './select/option.vue';

    import { assert, isArray } from '@src/utils';

    export default {
        name: 'area-select',
        components: {
            'v-select': Select,
            'v-option': Option
        },
        props: {
            value: {
                type: Array,
                required: true
            },
            type: {
                type: String,
                default: 'code', //  code-返回行政区域代码 text-返回文本 all-返回 code 和 text
                validator: (val) => ['all', 'code', 'text'].indexOf(val) > -1
            },
            placeholders: {
                type: Array,
                default: () => []
            },
            level: {
                type: Number,
                default: 1, // 0-->一联 1->二联 2->三联 3->四联
                validator: (val) => [0, 1, 2, 3].indexOf(val) > -1
            },
            size: {
                type: String,
                default: 'medium',
                validator: (val) => ['small', 'medium', 'large'].indexOf(val) > -1
            },
            disabled: {
                type: Boolean,
                default: false
            },

            data: {
                type: Object,
                required: true
            },
            disableLinkage: {
                type: Boolean,
                default: false
            }
        },

        data () {
            if (!this.data || !this.data['86']) {
                throw new Error('[area-linkage-vue]: 需要提供地区数据，格式参考见：https://github.com/liangzibo/area-data-vue');
            }

            return {
                // 区域数据
                provinces: this.data['86'],
                citys: {},
                areas: {},
                streets: {},
                curProvince: '', // text
                curProvinceCode: '', // code
                curCity: '',
                curCityCode: '',
                curArea: '',
                curAreaCode: '',
                curStreet: '',
                curStreetCode: '',

                // 设置默认值的判断
                defaults: [],
                isCode: false,
                isSetDefault: true,
                isShow1: false,
                isShow2: false,
                isShow3: false
            };
        },

        watch: {
            curProvinceCode (val, oldVal) {
                this.curProvince = this.provinces[val];
                this.provinceChange(val, oldVal === val);
            },

            curCityCode (val, oldVal) {
                this.curCity = this.citys[val];
                this.cityChange(val, oldVal === val);
            },

            curAreaCode (val, oldVal) {
                this.curArea = this.areas[val];
                this.areaChange(val, oldVal === val);
            },
            curStreetCode (val, oldVal) {
                this.curStreet = this.streets[val];
                this.streetChange(val, oldVal === val);
            },
            value (val) {
                if (!this.isSetDefault && isArray(val) && val.length === this.level + 1) {
                    this.beforeSetDefault();
                    this.setDefaultValue();
                }

                if (!this.isSetDefault && isArray(val) && val.length && val.length !== this.level + 1) {
                    assert(false, `设置的默认值和 level 值不匹配`);
                }
            }
        },

        methods: {
            provinceChange (val, isEqual) {
                if (this.level === 0) {
                    this.selectChange();
                } else if (this.level >= 1) {
                    this.citys = this.data[val];
                    if (this.citys === undefined) {
                        this.isShow1 = false;
                        this.selectChange();
                    } else {
                        this.isShow1 = true;
                        if (!this.citys) {
                            this.citys = {
                                [this.curProvinceCode]: this.curProvince
                            };
                            if (!this.disableLinkage) {
                                this.curCity = this.curProvince;
                                this.curCityCode = this.curCityCode;
                            }
                            // return;
                        }
                        let curCity = Object.values(this.citys)[0];
                        let curCityCode = Object.keys(this.citys)[0];
                        if (this.defaults[1]) {
                            if (this.isCode) {
                                curCityCode = find(Object.keys(this.citys), (item) => item === this.defaults[1]);
                                assert(curCityCode, `城市 ${this.defaults[1]} 不存在于省份 ${this.defaults[0]} 中`);
                                curCity = this.citys[curCityCode];
                            } else {
                                curCity = find(this.citys, (item) => item === this.defaults[1]);
                                assert(curCity, `城市 ${this.defaults[1]} 不存在于省份 ${this.defaults[0]} 中`);
                                curCityCode = find(Object.keys(this.citys), (item) => this.citys[item] === this.defaults[1]);
                            }
                        }

                        if (!this.disableLinkage) {
                            this.curCity = curCity;
                            this.curCityCode = curCityCode;
                        } else if (!isEqual) {
                            this.curCity = '';
                            this.curCityCode = '';
                            this.curArea = '';
                            this.curAreaCode = '';
                            this.curStreet = '';
                            this.curStreetCode = '';
                            this.selectChange();
                        }
                    }
                }
            },

            cityChange (val, isEqual) {
                if (this.level === 1) {
                    this.selectChange();
                } else if (this.level >= 2) {
                    this.areas = this.data[val];
                    if (this.areas === undefined) {
                        this.isShow2 = false;
                        this.selectChange();
                    } else {
                        this.isShow2 = true;
                        if (!this.areas) {
                            // fix 市级下不存在城区(#7)
                            this.areas = {
                                [this.curCityCode]: this.curCity
                            };
                            if (!this.disableLinkage) {
                                this.curArea = this.curCity;
                                this.curAreaCode = this.curCityCode;
                            }
                            // return;
                        }

                        let curArea = Object.values(this.areas)[0];
                        let curAreaCode = Object.keys(this.areas)[0];

                        if (this.defaults[2]) {
                            if (this.isCode) {
                                curAreaCode = find(Object.keys(this.areas), (item) => item === this.defaults[2]);
                                assert(curAreaCode, `县区 ${this.defaults[2]} 不存在于城市 ${this.defaults[1]} 中`);
                                curArea = this.areas[curAreaCode];
                            } else {
                                curArea = find(this.areas, (item) => item === this.defaults[2]);
                                assert(curArea, `县区 ${this.defaults[2]} 不存在于城市 ${this.defaults[1]} 中`);
                                curAreaCode = find(Object.keys(this.areas), (item) => this.areas[item] === this.defaults[2]);
                            }
                        }

                        if (!this.disableLinkage) {
                            this.curArea = curArea;
                            this.curAreaCode = curAreaCode;
                        } else if (!isEqual) {
                            this.curArea = '';
                            this.curAreaCode = '';
                            this.curStreet = '';
                            this.curStreetCode = '';
                            this.selectChange();
                        }
                    }
                }
            },

            areaChange (val, isEqual) {
                if (this.level === 2) {
                    this.selectChange();
                } else if (this.level === 3) {
                    this.streets = this.data[val];
                    if (this.streets === undefined) {
                        this.isShow3 = false;
                        this.selectChange();
                    } else {
                        this.isShow3 = true;
                        if (!this.streets) {
                            // fix 县区级下不存在街道(#7)
                            this.streets = {
                                [this.curAreaCode]: this.curArea
                            };
                            if (!this.disableLinkage) {
                                this.curStreet = this.curStreet;
                                this.curStreetCode = this.curStreetCode;
                            }
                            // return;
                        }

                        let curStreet = Object.values(this.streets)[0];
                        let curStreetCode = Object.keys(this.streets)[0];

                        if (this.defaults[3]) {
                            if (this.isCode) {
                                curStreetCode = find(Object.keys(this.streets), (item) => item === this.defaults[3]);
                                assert(curStreetCode, `街道 ${this.defaults[2]} 不存在于县区 ${this.defaults[2]} 中`);
                                curStreet = this.streets[curStreetCode];
                            } else {
                                curStreet = find(this.streets, (item) => item === this.defaults[3]);
                                assert(curStreet, `街道 ${this.defaults[2]} 不存在于县区 ${this.defaults[2]} 中`);
                                curStreetCode = find(Object.keys(this.streets), (item) => this.streets[item] === this.defaults[3]);
                            }
                        }

                        if (!this.disableLinkage) {
                            this.curStreet = curStreet;
                            this.curStreetCode = curStreetCode;
                        } else if (!isEqual) {
                            this.curStreet = '';
                            this.curStreetCode = '';
                            this.selectChange();
                        }
                    }
                }
            },
            streetChange (val) {
                this.curStreetCode = val;
                this.selectChange();
            },
            getAreaCode () {
                let codes = [];

                switch (this.level) {
                    case 0:
                        codes = [this.curProvinceCode];
                        break;
                    case 1:
                        codes = [this.curProvinceCode, this.curCityCode];
                        break;
                    case 2:
                        // fix #32 710000是台湾省
                        if (this.curProvinceCode === '820000') {
                            codes = [this.curProvinceCode, this.curCityCode];
                        } else {
                            codes = [this.curProvinceCode, this.curCityCode, this.curAreaCode];
                        }
                        break;
                    case 3:
                        // fix #32 710000是台湾省
                        if (this.curProvinceCode === '820000') {
                            codes = [this.curProvinceCode, this.curCityCode];
                        } else if (this.curProvinceCode === '810000' || this.curProvinceCode === '710000') {
                            codes = [this.curProvinceCode, this.curCityCode, this.curAreaCode];
                        } else {
                            if (this.isShow3) {
                                codes = [this.curProvinceCode, this.curCityCode, this.curAreaCode, this.curStreetCode];
                            } else {
                                codes = [this.curProvinceCode, this.curCityCode, this.curAreaCode];
                            }
                        }
                        break;
                }
                return codes;
            },

            getAreaText () {
                let texts = [];

                switch (this.level) {
                    case 0:
                        texts = [this.curProvince];
                        break;
                    case 1:
                        // fix #32 710000是台湾省
                        texts = [this.curProvince, this.curCity];
                        break;
                    case 2:
                        if (this.curProvince === '澳门') {
                            texts = [this.curProvince, this.curCity];
                        } else {
                            texts = [this.curProvince, this.curCity, this.curArea];
                        }
                        break;
                    case 3:
                        if (this.curProvince === '澳门') {
                            texts = [this.curProvince, this.curCity];
                        } else if (this.curProvince === '香港' || this.curProvince === '台湾') {
                            texts = [this.curProvince, this.curCity, this.curArea];
                        } else {
                            texts = [this.curProvince, this.curCity, this.curArea, this.curStreet];
                        }

                        break;
                }
                return texts;
            },

            getAreaCodeAndText (selected) {
                let textCodes = [];

                switch (this.level) {
                    case 0:
                        textCodes = [{ [this.curProvinceCode]: this.curProvince }];
                        break;
                    case 1:

                        textCodes = [{ [this.curProvinceCode]: this.curProvince }, { [this.curCityCode]: this.curCity }];
                        break;
                    case 2:
                        textCodes = [{ [this.curProvinceCode]: this.curProvince }, { [this.curCityCode]: this.curCity }, { [this.curAreaCode]: this.curArea }];
                        break;
                    case 3:
                        textCodes = [{ [this.curProvinceCode]: this.curProvince }, { [this.curCityCode]: this.curCity }, { [this.curAreaCode]: this.curArea }, { [this.curStreetCode]: this.curStreet }];
                        break;
                }

                return textCodes;
            },

            beforeSetDefault () {
                const chinese = /^[\u4E00-\u9FA5\uF900-\uFA2D]{2,}$/;
                const num = /^\d{6,}$/;
                const isCode = num.test(this.value[0]);
                let isValid;

                if (!isCode) {
                    isValid = this.value.every((item) => chinese.test(item));
                } else {
                    isValid = this.value.every((item) => num.test(item));
                }

                assert(isValid, '传入的默认值参数有误');
                // 映射默认值，避免直接更改props
                this.defaults = [].concat(this.value);
                this.isCode = isCode;
                this.isSetDefault = true;
            },

            setDefaultValue () {
                let provinceCode = '';

                if (this.isCode) {
                    provinceCode = this.defaults[0];
                } else {
                    const province = find(this.provinces, (item) => item === this.defaults[0]);
                    assert(province, `省份 ${this.defaults[0]} 不存在`);
                    provinceCode = find(Object.keys(this.provinces), (item) => this.provinces[item] === this.defaults[0]);
                }
                this.curProvinceCode = provinceCode;
                // 还原默认值，避免用户选择出错
                this.$nextTick(() => {
                    this.defaults = [];
                    // this.isCode = false;
                    this.isSetDefault = false;
                });
            },

            selectChange () {
                this.isSetDefault = true;
                let res = [];
                if (this.type === 'code') {
                    res = this.getAreaCode();
                } else if (this.type === 'text') {
                    res = this.getAreaText();
                } else if (this.type === 'all') {
                    res = this.getAreaCodeAndText();
                }
                this.$emit('input', res);
                this.$emit('change', res);
            }
        },

        created () {
            if (isArray(this.value) && this.value.length === this.level + 1) {
                this.beforeSetDefault();
                this.setDefaultValue();
            }

            if (isArray(this.value) && this.value.length && this.value.length !== this.level + 1) {
                assert(false, `设置的默认值和 level 值不匹配`);
            }
        }
    };

</script>

<style>
    .area-select-wrap .area-select{
        margin-left: 10px;
    }

    .area-select-wrap .area-select-empty{
        padding: 4px 0;
        margin: 0;
        text-align: center;
        color: #999;
        font-size: 14px;
    }
</style>
