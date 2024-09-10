<template>
    <div class="upload-home">
        <img id="bg1" class="background-image1" alt="Background Image"/>
        <img id="bg2" class="background-image2" alt="Background Image"/>
        <div class="toolbar">
          <el-tooltip content="上传方式" placement="left">
                <el-button class="toolbar-button" size="large" color="#626aef" @click="changeUploadMethod" circle>
                    <el-icon size="large"><UploadFilled /></el-icon>
                </el-button>
            </el-tooltip>
            <el-tooltip content="链接格式" placement="left">
                <el-button class="toolbar-button" size="large" type="success" @click="openUrlDialog" circle>
                    <el-icon size="large"><Connection /></el-icon>
                </el-button>
            </el-tooltip>
            <el-tooltip content="管理页面" placement="left">
                <el-button class="toolbar-button" size="large" type="primary" @click="handleManage" circle>
                    <el-icon size="large"><UserFilled /></el-icon>
                </el-button>
            </el-tooltip>
        </div>
        <div class="header">
            <a href="https://github.com/MarSeventh/CloudFlare-ImgBed">
                <img class="logo" alt="AreYouOk logo" :src="logoUrl"/>
            </a> 
            <h1><a class="main-title" href="https://github.com/MarSeventh/CloudFlare-ImgBed" target="_blank">{{ ownerName }}</a> PicBed</h1>
        </div>
        <UploadForm :selectedUrlForm="selectedUrlForm" :uploadMethod="uploadMethod" class="upload"/>
        <el-dialog title="选择复制链接格式" v-model="showUrlDialog" width="40%" :show-close="false">
            <el-radio-group v-model="selectedUrlForm">
                <el-radio value="url">原始链接</el-radio>
                <el-radio value="md">MarkDown</el-radio>
                <el-radio value="html">HTML</el-radio>
                <el-radio value="ubb">BBCode</el-radio>
            </el-radio-group>
            <div class="dialog-action">
                <el-button type="primary" @click="showUrlDialog = false">确定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import UploadForm from '@/components/UploadForm.vue'
import { ref } from 'vue'
import { mapGetters } from 'vuex'
import { Refresh, UploadFilled, UserFilled } from '@element-plus/icons-vue'
export default {
    name: 'UploadHome',
    data() {
        return {
            selectedUrlForm: ref('url'),
            uploadMethod: ref('drag'),
            showUrlDialog: false,
            bingWallPaperIndex: 0,
            customWallPaperIndex: 0
        }
    },
    computed: {
        ...mapGetters(['userConfig', 'bingWallPapers']),
        ownerName() {
            return this.userConfig?.ownerName || 'AreYouOk'
        },
        logoUrl() {
            return this.userConfig?.logoUrl || require('../assets/logo.png')
        }
    },
    mounted() {
        const bg1 = document.getElementById('bg1')
        const bg2 = document.getElementById('bg2')
        if (this.userConfig?.uploadBkImg === 'bing') {
            //bing壁纸轮播
            this.$store.dispatch('fetchBingWallPapers').then(() => {
                bg1.src = this.bingWallPapers[this.bingWallPaperIndex]?.url
                bg1.onload = () => {
                    bg1.style.opacity = 1
                }
                setInterval(() => {
                    //如果bing壁纸组为空，跳过
                    let curBg = bg1.style.opacity != 0 ? bg1 : bg2
                    let nextBg = bg1.style.opacity != 0 ? bg2 : bg1
                    curBg.style.opacity = 0
                    this.bingWallPaperIndex = (this.bingWallPaperIndex + 1) % this.bingWallPapers.length
                    nextBg.src = this.bingWallPapers[this.bingWallPaperIndex]?.url
                    nextBg.onload = () => {
                        nextBg.style.opacity = 1
                    }
                }, 3000)
            })
        } else if (this.userConfig?.uploadBkImg instanceof Array && this.userConfig?.uploadBkImg?.length > 1) {
            //自定义壁纸组轮播
            bg1.src = this.userConfig.uploadBkImg[this.customWallPaperIndex]
            bg1.onload = () => {
                bg1.style.opacity = 1
            }
            setInterval(() => {
                let curBg = bg1.style.opacity != 0 ? bg1 : bg2
                let nextBg = bg1.style.opacity != 0 ? bg2 : bg1
                curBg.style.opacity = 0
                this.customWallPaperIndex = (this.customWallPaperIndex + 1) % this.userConfig.uploadBkImg.length
                nextBg.src = this.userConfig.uploadBkImg[this.customWallPaperIndex]
                nextBg.onload = () => {
                    nextBg.style.opacity = 1
                }
            }, 3000)
        } else if (this.userConfig?.uploadBkImg instanceof Array && this.userConfig?.uploadBkImg.length == 1) {
            //单张自定义壁纸
            bg1.src = this.userConfig.uploadBkImg[0]
            bg1.onload = () => {
                bg1.style.opacity = 1
            }
        } else {
            //默认壁纸
            bg1.src = 'https://imgbed.sanyue.site/file/0dbd5add3605a0b2e8994.jpg'
            bg1.onload = () => {
                bg1.style.opacity = 1
            }
        }
    },
    components: {
        UploadForm
    },
    methods: {
        handleManage() {
            window.location.href = '/admin'
        },
        openUrlDialog() {
            this.showUrlDialog = true
        },
        changeUploadMethod() {
            this.uploadMethod = this.uploadMethod === 'drag' ? 'paste' : 'drag'
        }
    }
}
</script>

<style scoped>
.toolbar {
    display: flex;
    flex-direction: column;
    position: fixed;
    justify-content: center;
    bottom: 40vh;
    right: 1vw;
    z-index: 10000;
}
.toolbar-button {
    border: none;
    transition: all 0.3s ease;
    margin-bottom: 20px;
    margin-left: 0;
}
.toolbar-button:hover {
    box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.1);
    transform: translateY(-3px);
}
:deep(.el-dialog) {
    border-radius: 12px;
    background-color: rgba(255, 255, 255, 0.7);
    backdrop-filter: blur(10px);
    box-shadow: 0 0 10px 2px rgba(0, 0, 0, 0.1);
}
.dialog-action {
    display: flex;
    justify-content: center;
    margin-top: 20px;
}
.header {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 15px;
    position: fixed;
    top: 5vh;
    color: blanchedalmond;
    user-select: none;
    text-decoration: none;
}
.main-title {
    background: linear-gradient(to right, rgb(239, 250, 195), #f3a060);
    background-clip: text;
    color: transparent;
    text-decoration: none;
}
.logo {
    height: 80px;
    width: 80px;
    margin-right: 5px;
}
.upload-home {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    transition: background-image 1s ease-in-out;
    background-size: cover;
    background-attachment: fixed;
    height: 100vh;
}
.upload {
    position: fixed;
    top: 20vh;
}
.background-image1 {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: -1;
    opacity: 0;
    transition: all 1s ease-in-out;
}
.background-image2 {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: -1;
    opacity: 0;
    transition: all 1s ease-in-out;
}
</style>