<template>
    <div>
        <el-upload
            :action="item.config && item.config.action"
            list-type="picture-card" 
            :multiple="item.config && item.config.multiple"
            :data="item.config && item.config.data"
            :headers="item.config && item.config.headers"
            :on-success="onSuccess"
            :file-list="fileList"
            :on-remove="onRemove"
            :before-upload="beforeAvatarUpload">
            <i class="el-icon-plus"></i>
        </el-upload>
        <!-- <el-upload
            class="upload-demo"
            drag
            action="https://jsonplaceholder.typicode.com/posts/"
            multiple>
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
            <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload> -->
    </div>
</template>
<script>
import base from '../../mixins/base.js';
import util from '../../utils/util.js'
/**
 * type: upload-file, upload-image
 * config: {
 *      plain: true 
 *      round: true
 * }
 */
export default {
    //prop, value, config: {}, filter, disabled, link?to router, $data
    props: ['item'],
    mixins: [base],
    name: 'fdupload',
    data() {
        return {
            imageTypes: [],
            fileList: []
        }
    },
    watch: {
        selfValue(nd, od) {
            if (!this.testValue(nd, od))
                this.resetFileList(nd)
        }
    },
    computed: {
        selfValue() {
            return this.item.value
        }
    },
    methods: {
        change() { 
             this.mixin_event({
                type: 'change',
                prop: this.item.prop,
                value: this.item.value
            })
        },
        beforeAvatarUpload(file) {
            return this.item.config.beforeUpload
        },
        onSuccess(response, file, fileList) {
            const fn = (data) => {
                if (data) {
                    if (Array.isArray(data))
                        this.item.value = this.item.value.concat(data)
                    else
                        this.item.value.push(data) 
                    this.change()
                }
            }
            if (this.item.config.onSuccess) {
                let promise = this.item.config.onSuccess(response)
                promise && promise.then(data => {
                    fn(data)
                })
            } else {
                fn(response.data)
            }
        },
        onRemove(file) {
            this.item.value = this.item.value.filter(el => {
                if (el.url) 
                    return file.url.indexOf(el.url) === -1
                else 
                    return file.url.indexOf(el) === -1 
            }) 
            this.change()
        },
        testValue(nd, od) {
            if (typeof nd === typeof od) {
                if (Array.isArray(nd) && nd.length && od.length) {
                    return nd[0] === od[0]
                }
                return true
            } 
            return false
        }, 
        getUrl(url) {
            return this.mixin_filter(this.item.filter, url, this.item.$data)
        },
        resetFileList(value) {
            if (typeof value === 'string') {
                this.fileList = [{url: this.getUrl(value)}]
            } else if (Array.isArray(value)) {
                this.fileList = value.map(el => {
                    if (typeof el === 'string') 
                        return {url: this.getUrl(el)}
                    else 
                        return {...el, url: this.getUrl(el.url)}
                }) 
            }  
        }
    },
    created() {
        if (this.item.value === void 0) {
            this.$set(this.item, 'value', [])
        }
        this.item.$data[this.item.prop] = this.item.value 
        this.resetFileList(this.item.value)
    }
}
</script>
