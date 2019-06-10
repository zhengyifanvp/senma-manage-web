<template>
  <v-form v-model="valid" ref="myGoodsForms">
    <v-text-field v-model="goods.itemName" label="请输入商品名称" required :rules="nameRules"/>
    <v-text-field v-model="goods.itemCategory" label="请输入商品的分类" required :rules="nameRules"/>
    <v-text-field v-model="goods.itemNum" label="请输入出货/进货的数量" required :rules="numRules"/>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  export default {
    name: "goods-forms",
    props: {
      oldGoods: {
        type:Object,
      },
      isEdit: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        valid: false, // 表单校验结果标记
        goods: {
          itemId: '', //商品id
          itemName: '', //商品名称
          itemCategory: '', //商品分类数组
          itemNum: '',//商品的数量
        },
        nameRules: [//表单校验填的规则
          v => !!v || "所填入的数据不能为空",
          v => v.length > 1 || "至少2位"

        ],
        numRules: [
          v => !!v || "货物的数量不能为空",
          v => /^([1-9]|[1-9]\d|[1-9]\d{2}|[1-9]\d{3}|[1-9]\d{4}|[1-9]\d{5})$/.test(v) || "只能输入1~99999之内的数字"
        ],
      }
    },
    methods: {
      submit() {
        // 1、表单校验
        if (this.$refs.myGoodsForms.validate()) {
          // 2、定义一个请求参数对象，通过解构表达式来获取brand中的属性
          const params = this.goods;
          console.log(params);
          // 3、将数据提交到后台
          if(this.isEdit == true){
            this.$http({
              method:'put',
              url: '/item',
              data:params
            }).then(() => {
              // 关闭窗口
              this.$emit("close");
              this.$message.success("出货成功！");
            })
              .catch(() => {
                this.$message.error("出货失败！");
              });
          }else{
          this.$http({
            method: 'post',
            url: '/item',
            data: this.$qs.stringify(params)
          }).then(() => {
            // 关闭窗口
            this.$emit("close");
            this.$message.success("保存成功！");
          })
            .catch(() => {
              this.$message.error("保存失败！");
            });
          }
        }
      },
      clear() {
        // 重置表单
        this.$refs.myGoodsForms.reset();
      }
    },
    watch: {
      oldGoods: {// 监控oldGoods的变化
        handler(val) {
          if (val) {
            // 注意不要直接复制，否则这边的修改会影响到父组件的数据，copy属性即可
            this.goods = Object.deepCopy(val)
          } else {
            // 为空，goods
            this.goods = {
              itemId: '',
              itemName: '',
              itemCategory: '',
              itemNum: '',
            }
          }
        },
        deep: true
      }
    }
  }
</script>

<style scoped>

</style>
