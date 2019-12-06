<template>
  <div id="patient-card-qrcode">
    <h5-title @clickLeft="$router.go(-1)" background="#108EE9" color="#ffffff"
      >二维码
    </h5-title>
    <div class="section-title">
      河北东软电子就诊卡
    </div>
    <div class="section-qrcode">
      <qrcode :value="patientCardQRCode" :options="{ width: 287 }"></qrcode>
      <div class="code">{{ patientCardQRCode }}</div>
    </div>
  </div>
</template>

<script>
import Qrcode from "@chenfengyuan/vue-qrcode";
import { getWithAuth, postWithAuth } from "../../../../utils/util.http";
import debug from "../../../../utils/util.debug";
import H5Title from "../../../../components/title/h5-title";
import { MessageBox } from "mint-ui";

export default {
  name: "patient-card-qrcode",
  components: {
    H5Title,
    Qrcode
  },
  data() {
    return {
      patientCardQRCode: "暂无",
      patientCardQRCodeState: ""
    };
  },
  created() {
    this.getPatientCardQRCode();
    setInterval(() => {
      this.getPatientCardQRCode();
    }, 5000);
  },
  methods: {
    getPatientCardQRCode() {
      const config = {
        url: `api/v1/card`
      };
      getWithAuth(config)
        .then(res => {
          debug.log("患者获取就诊卡二维码信息成功", res);
          this.patientCardQRCode = res.code;
          this.patientCardQRCodeState = res.state;
          if (res.code === "未申领二维码" && res.state === "notCreate") {
            this.onRegisterPatientCardQRCode();
            // } else if (res.state === "waiting") {
          } else if (res.state === "scaned") {
            // 需要患者授权
            MessageBox.alert("授权医生查看就诊信息").then(() => {
              this.onRegisterPatientCardQRCode();
            });
          }
        })
        .catch(err => {
          debug.error("患者获取就诊卡二维码信息失败", err);
        });
    },
    onRegisterPatientCardQRCode() {
      const config = {
        url: `api/v1/card`
      };
      postWithAuth(config)
        .then(res => {
          debug.log("患者申领就诊卡二维码成功", res);
          this.getPatientCardQRCode();
        })
        .catch(err => {
          debug.error("患者申领就诊卡二维码失败", err);
        });
    }
  }
};
</script>

<style lang="less" scoped>
@import "patient-card";
</style>
