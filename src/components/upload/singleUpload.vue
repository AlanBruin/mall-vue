<template>
    <el-upload
        class="upload-demo"
        action="https://mall-morro.oss-cn-beijing.aliyuncs.com/"
        :data="dataObj"
        :on-preview="handlePreview"
        :on-success="handleSuccess"
        :on-remove="handleRemove"
        :file-list="fileList"
        :multiple=false
        :limit=1
        :before-upload="beforeUpload"
        list-type="picture">
        <el-button size="small"
                   type="primary">点击上传
        </el-button>
        <div slot="tip"
             class="el-upload__tip">只能上传jpg/png文件，且不超过500kb
        </div>
    </el-upload>
</template>

<script>

import { policy } from "./policy";
import { getUUID } from "../../utils";

export default {
    name: "singleUpload",
    props: {
        value: String
    },
    computed: {
        imageUrl() {
            return this.value;
        },
        imageName() {
            if (this.value != null && this.value !== "") {
                return this.value.substr(this.value.lastIndexOf("/") + 1);
            } else {
                return null;
            }
        },
        fileList() {
            return [{ name: this.imageUrl, url: this.imageUrl }];
        },
        showFileList: {
            get: function () {
                return this.value !== null && this.value !== "" && this.value !== undefined;
            },
            set: function (newValue) {

            }
        }
    },
    data() {
        return {
            dataObj: {
                key: "",
                dir: "",
                policy: "",
                kOSSAccessKeyIdey: "",
                success_action_status: "",
                signature: ""
            }
        };
    },
    methods: {
        emitParentInputUrl(url) {
            this.$emit("input", url);
        },

        beforeUpload(file) {
            let _self = this;
            return new Promise(((resolve, reject) => {
                policy().then(response => {
                    _self.dataObj.key = response.dir + "/" + getUUID() + "_${filename}";
                    _self.dataObj.dir = response.dir;
                    _self.dataObj.policy = response.policy;
                    _self.dataObj.OSSAccessKeyId = response.accessid;
                    _self.dataObj.success_action_status = "200";
                    _self.dataObj.signature = response.signature;
                    _self.dataObj.host = response.host;
                    resolve(true);
                }).catch(err => {
                    reject(false);
                });
            }));
        },

        handlePreview(file) {

        },

        handleRemove(file, fileList) {

        },

        handleSuccess(response, file, fileList) {
            console.log(response, file, fileList);
            this.emitParentInputUrl(this.dataObj.host + "/" + this.dataObj.key.replace("${filename}", file.name));
        }
    }
};
</script>

<style scoped>

</style>
