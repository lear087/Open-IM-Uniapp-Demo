<template>
  <view class="register_container content_with_back">
    <view class="back_icon">
      <u-icon
        name="arrow-leftward"
        bold
        size="36"
        color="#1D6BED"
        @click="back"
      />
    </view>
    <view class="title">新用户注册</view>
    <u-form
      labelPosition="top"
      :model="userInfo"
      :rules="rules"
      :labelStyle="{
        fontSize: '14px',
        marginTop: '20rpx',
        minWidth: '200rpx',
      }"
      ref="registerForm"
    >
      <u-form-item prop="phoneNumber" label="手机号码" borderBottom>
        <u-input
          v-model="userInfo.phoneNumber"
          border="none"
          placeholder="请输入手机号码"
          clearable
        >
          <view slot="prefix" class="phoneNumber_areacode" @click="showPicker">
            <text class="areacode_content">+{{ userInfo.areaCode }}</text>
            <u-icon class="arrow_down" name="arrow-down"></u-icon>
          </view>
        </u-input>
      </u-form-item>
      <u-form-item prop="invitationCode" label="邀请码" borderBottom>
        <u-input
          v-model="userInfo.invitationCode"
          border="none"
          placeholder="请输入邀请码(选填)"
          clearable
        >
        </u-input>
      </u-form-item>
    </u-form>
    <view class="action_btn">
      <u-button @click="sendSms" type="primary" :disabled="!checked[0]">
        立即注册
      </u-button>
    </view>
    <view class="agreement">
      <u-checkbox-group v-model="checked">
        <u-checkbox
          iconSize="12"
          labelSize="12"
          shape="circle"
          label="我已阅读并同意："
          :name="true"
        ></u-checkbox>
      </u-checkbox-group>
      <text class="detail">《服务协议》,</text>
      <text class="detail">《隐私政策》</text>
    </view>
    <AreaPicker ref="AreaPicker" @chooseArea="chooseArea" />
  </view>
</template>

<script>
import AreaPicker from "@/components/AreaPicker";
import { businessSendSms } from "@/api/login";
import { SmsUserFor } from "@/constant";
import { checkLoginError } from "@/util/common";
export default {
  components: {
    AreaPicker,
  },
  data() {
    return {
      userInfo: {
        phoneNumber: "",
        email: "",
        areaCode: "86",
        invitationCode: null,
      },
      checked: [true],
      rules: {
        phoneNumber: [
          {
            type: "string",
            required: true,
            message: "请输入手机号码",
            trigger: ["blur", "change"],
          },
        ],
      },
      pageStatus: "normal",
    };
  },
  methods: {
    sendSms() {
      this.$refs.registerForm.validate().then((valid) => {
        const options = {
          phoneNumber: this.userInfo.phoneNumber,
          areaCode: `+${this.userInfo.areaCode}`,
          usedFor: SmsUserFor.Register,
          invitationCode: this.userInfo.invitationCode,
        };
        businessSendSms(options)
          .then(() => {
            uni.$u.toast("验证码已发送！");
            setTimeout(
              () =>
                uni.$u.route("/pages/login/verifyCode/index", {
                  userInfo: JSON.stringify(this.userInfo),
                }),
              1000
            );
          })
          .catch((err) => {
            console.error(err);
            uni.$u.toast(checkLoginError(err));
          });
      });
    },
    back() {
      uni.$u.route("/pages/login/index");
    },
    showPicker() {
      this.$refs.AreaPicker.init();
    },
    chooseArea(areaCode) {
      this.userInfo.areaCode = areaCode;
    },
  },
};
</script>
<style lang="scss" scoped>
.register_container {
  margin-top: var(--status-bar-height);

  .action_btn {
    padding-top: 20vh;
  }
}
</style>
