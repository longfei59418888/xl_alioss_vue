<template>
    <div class="vui_alioss_upload">
        <div @click="uloadImg()">
            <slot></slot>
        </div>
        <div class="vui_alioss_file">
            <input ref="vui_alioss_file" type="file"  @change="doUpload()"/>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'

    export default {
        name: 'upload',
        props: ['url'],
        data() {
            return {}
        },
        methods: {
            uloadImg() {
                this.$refs['vui_alioss_file'].click()
            },
            doUpload(e) {
                let _this = this, file = this.$refs['vui_alioss_file'].files[0]
                if (!file || file.length < 1) return
                axios({
                    method: 'get',
                    url: _this.url,
                    withCredentials: true
                }).then(res => {
                    return res.data
                }).then(data => {
                    if(_this._events.dealData){
                        _this.$emit('upload',data,file,_this.upLoadOss)
                        return
                    }
                    _this.upLoadOss(Object.assign({}, data, {file: file}))
                })
            },
            upLoadOss(data) {
                const key = data.dir + new Date().getTime() +'_' + data.file.name
                var request = new FormData(), _this = this;
                request.append("OSSAccessKeyId", data.accessid);//Bucket 拥有者的Access Key Id。
                request.append("policy", data.policy);//policy规定了请求的表单域的合法性
                request.append("Signature", data.signature);//根据Access Key Secret和policy计算的签名信息，OSS验证该签名信息从而验证该Post请求的合法性
                request.append("key", key);//文件名字，可设置路径
                request.append("success_action_status", '200');// 让服务端返回200,不然，默认会返回204
                request.append('file', data.file);//需要上传的文件 file
                axios({
                    method: 'post',
                    url: data.host,
                    data: request,
                }).then(res => {
                    return res.data
                }).then(rst => {
                    _this.$emit('upLoadImgEnd', data.host + '/' + key)
                })
            }
        }
    }

</script>

<style scoped>
    .vui_alioss_upload {
        display: inline-block;
    }

    .vui_alioss_upload > div {
        display: inline-block;
    }

    .vui_alioss_upload .vui_alioss_file {
        height: 0;
        overflow: hidden
    }
</style>
