<template>
  <div class="navbar">
    <hamburger
      :is-active="sidebar.opened"
      class="hamburger-container"
      @toggleClick="toggleSideBar"
    />

    <breadcrumb class="breadcrumb-container" />

    <div class="right-menu">
      <el-dropdown class="avatar-container" trigger="click">
        <div class="avatar-wrapper">
          <img :src="avatar + '?imageView2/1/w/80/h/80'" class="user-avatar">
          <span class="user-name">{{ name }}</span>
          <i class="el-icon-caret-bottom" />
        </div>
        <el-dropdown-menu slot="dropdown" class="user-dropdown">
          <div @click="getInfo()">
            <el-dropdown-item><i class="el-icon-document" /> 基本资料
            </el-dropdown-item>
          </div>
          <div @click="dialogChangePassword = true">
            <el-dropdown-item><i class="el-icon-lock" /> 修改密码
            </el-dropdown-item>
          </div>
          <div @click="dialogImg = true">
            <el-dropdown-item><i class="el-icon-user" /> 修改头像
            </el-dropdown-item>
          </div>
          <!-- 处理退出，删除token -->
          <el-dropdown-item @click.native="logout"><i class="el-icon-switch-button" /> 退出
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <el-dialog
      title="基本资料"
      :visible.sync="dialogUserInfo"
      :modal-append-to-body="false"
      width="50%"
    >
      <el-form
        ref="userInfoForm"
        :model="userInfoForm"
        label-width="80px"
        label-position="left"
        class="news-form"
      >
        <el-form-item label="用户名" prop="username" placeholder="请输入用户名">
          <el-input v-model="userInfoForm.username" />
        </el-form-item>
        <el-form-item label="真实姓名" prop="realname" placeholder="请输入真实姓名">
          <el-input v-model="userInfoForm.realname" />
        </el-form-item>
        <el-form-item label="昵称" prop="nickname" placeholder="请输入昵称">
          <el-input v-model="userInfoForm.nickname" />
        </el-form-item>

        <el-form-item label="地市" prop="cityName" placeholder="请输入地市">
          <el-input v-model="userInfoForm.cityName" />
        </el-form-item>
        <el-form-item label="单位" prop="group" placeholder="请输入单位">
          <el-input v-model="userInfoForm.group" />
        </el-form-item>
        <el-form-item label="工号" prop="jobNo" placeholder="请输入工号">
          <el-input v-model="userInfoForm.jobNo" />
        </el-form-item>
        <el-form-item label="手机号码" prop="phone" placeholder="请输入手机号码">
          <el-input v-model.number="userInfoForm.phone" />
        </el-form-item>

        <el-form-item label="类型" prop="utype" placeholder="现场工作人员">
          <el-select v-model="userInfoForm.utype" placeholder="请选择发现地域">
            <el-option label="区域一" value="shanghai" />
            <el-option label="区域二" value="beijing" />
          </el-select>
        </el-form-item>
        <div style="text-align:center">
          <el-button type="primary" @click="dialogUserInfo = false">关 闭</el-button>
        </div>
      </el-form>
    </el-dialog>

    <el-dialog
      title="修改密码"
      :visible.sync="dialogChangePassword"
      :modal-append-to-body="false"
      width="50%"
    >
      <el-form
        ref="formChangePassword"
        :model="formChangePassword"
        :rules="formChangePasswordRules"
        label-width="120px"
        label-position="left"
        class="news-form"
      >
        <el-form-item label="当前密码2" prop="oldPass" placeholder="请输入当前密码">
          <el-input v-model="formChangePassword.oldPass" />
        </el-form-item>
        <el-form-item label="新密码" prop="newPass" placeholder="请输入新密码">
          <el-input v-model="formChangePassword.newPass" />
        </el-form-item>
        <el-form-item label="确认新密码" prop="newPass2" placeholder="请输入确认新密码">
          <el-input v-model="formChangePassword.newPass2" />
        </el-form-item>

        <div style="text-align:center">
          <el-button type="primary" @click="changePassword()">确认修改</el-button>
        </div>
      </el-form>
    </el-dialog>

    <el-dialog
      title="修改头像"
      :visible.sync="dialogImg"
      :modal-append-to-body="false"
      width="35%"
    >
      <div style="text-align:center; ">
        <el-image class="img-info">
          <div slot="error" class="image-slot">
            <i class="el-icon-picture-outline" />
          </div>
        </el-image>
        <img v-if="imageUrl" :src="imageUrl" class="avatar">
        <div style="text-align:center;">

          <el-upload
            style="display: inline-block;"
            action="string"
            :http-request="uploadAvatar"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >

            <el-button type="primary" style="margin-right:20px">
              上传图片
              <i class="el-icon-plus avatar-uploader-icon" />
            </el-button>
          </el-upload>
          <el-button type="primary" @click="setAvatar">确认更改</el-button>
        </div>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Breadcrumb from '@/components/Breadcrumb'
import Hamburger from '@/components/Hamburger'
import { changePassword, uploadAvatar, setAvatar } from '@/api/admin'

export default {
  components: {
    Breadcrumb,
    Hamburger
  },
  data() {
    const validatePassNew = (rule, value, callback) => {
      if (value.length < 4 || value.length > 32) {
        callback(new Error('密码的长度在4-32位！请输入正确的新密码'))
      } else {
        callback()
      }
    }
    const validatePass = (rule, value, callback) => {
      if (value !== this.formChangePassword.newPass) {
        callback(new Error('两次密码需要一致'))
      } else {
        callback()
      }
    }
    return {
      dialogUserInfo: false,
      dialogChangePassword: false,
      dialogImg: false,
      imageUrl: '',
      fileData: null,
      userInfoForm: {
        username: '', // 用户名
        realname: '', // 真实姓名
        nickname: '', // 昵称
        cityName: '', // 地市
        group: '', // 单位 todo 字段待定
        jobNo: '', // 工号
        phone: '', // 手机号
        utype: '' // 类型
      },
      formChangePassword: {
        oldPass: '',
        newPass: '',
        newPass2: ''
      },
      formChangePasswordRules: {
        oldPass: [{ required: true, message: '请输入原来的密码', trigger: 'blur' }],
        newPass: [{ required: true, message: '请输入新密码', trigger: 'blur' },
          { trigger: 'blur', validator: validatePassNew }
        ],
        newPass2: [{ required: true, message: '请重新输入新密码', trigger: 'blur' },
          { trigger: 'blur', validator: validatePass }]
      }

    }
  },
  computed: {
    ...mapGetters([
      'sidebar',
      'avatar',
      'name'
    ])
  },
  methods: {
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw)
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    toggleSideBar() {
      this.$store.dispatch('app/toggleSideBar')
    },
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push(`/login`)
    },
    async getInfo() {
      const data = await this.$store.dispatch('user/getInfo')
      this.userInfoForm = data.user
      this.dialogUserInfo = true
    },
    changePassword() {
      this.$refs.formChangePassword.validate(async valid => {
        if (valid) {
          // 验证通过
          await changePassword({ oldPass: this.formChangePassword.oldPass, newPass: this.formChangePassword.newPass })
          this.dialogChangePassword = false
        } else {
          console.log('重置密码验证未通过!!')
          return false
        }
      })
    },
    async uploadAvatar(file) {
      const params = new FormData()
      params.append('file', file.file)
      const { data } = await uploadAvatar(params)
      this.fileData = data.result // 返回的是表单格式
    },
    async setAvatar() {
      await setAvatar({ json: JSON.stringify(this.fileData) })
      this.dialogImg = false
    }
  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
  position: relative;
  background: #fff;
  box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08);

  .hamburger-container {
    line-height: 46px;
    height: 100%;
    float: left;
    cursor: pointer;
    transition: background 0.3s;
    -webkit-tap-highlight-color: transparent;

    &:hover {
      background: rgba(0, 0, 0, 0.025);
    }
  }

  .breadcrumb-container {
    float: left;
  }

  .right-menu {
    float: right;
    height: 100%;
    line-height: 50px;

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 8px;
      height: 100%;
      font-size: 18px;
      color: #5a5e66;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: background 0.3s;

        &:hover {
          background: rgba(0, 0, 0, 0.025);
        }
      }
    }

    .avatar-container {
      margin-right: 30px;

      .avatar-wrapper {
        position: relative;
        .user-name {
          font-size: 15px;
          display: inline-block;
          margin-left: 10px;
          font-weight: bold;
          cursor: pointer;
        }
        .user-avatar {
          cursor: pointer;
          width: 40px;
          height: 40px;
          border-radius: 50%;
          vertical-align: middle;
        }

        .el-icon-caret-bottom {
          cursor: pointer;
          position: absolute;
          right: -20px;
          top: 25px;
          font-size: 12px;
        }
      }
    }
  }
}
.user-dropdown {
  ::v-deep .el-dropdown-menu__item i {
    font-size: 16px;
  }
}
.news-form {
  ::v-deep.el-select{
    width: 100%;
  }
}
.img-info{
    height: 240px;
    width: 350px;
    margin-bottom:20px;
    ::v-deep.image-slot{
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    background: #f5f7fa;
    color: #909399;
    font-size: 24px;
  }
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
    width: 150px;
    height: 150px;
    display: block;
    margin: 0px auto;
    margin-bottom: 20px;
  }

</style>
