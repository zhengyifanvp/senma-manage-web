<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <v-card>
    <v-card-title>
      <v-btn color="primary" @click="addGoods">新增商品</v-btn>
      <v-spacer/>
    </v-card-title>
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="goods"
      :pagination.sync="pagination"
      :total-items="totalGoods"
      :loading="loading"
      class="elevation-1"
    >
      <template v-slot:items="props">
        <td class="text-xs-center">{{ props.item.itemId }}</td>
        <td class="text-xs-center">{{ props.item.itemName }}</td>
        <td class="text-xs-center">{{ props.item.itemCategory }}</td>
        <td class="text-xs-center">{{ props.item.itemNum }}</td>
        <td class="justify-center layout px-0">
          <v-btn color="info" @click="editGoods(props.item)">出货
          </v-btn>
        </td>
      </template>
    </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="500" v-model="show" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? '出' : '新增'}}货管理</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5" style="height:400px">
          <goods-forms @close="closeWindow" :oldGoods="oldGoods" :isEdit="isEdit" />
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  // 导入自定义的表单组件
  import GoodsForms from './GoodsForms'

  export default {
    name: "goods",
    data() {
      return {
        totalGoods: 0, // 总条数
        goods: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: '编号', align: 'center', value: 'item_id', sortable: true},
          {text: '名称', align: 'center', sortable: false, value: 'item_name'},
          {text: '类型', align: 'center', sortable: false, value: 'item_category'},
          {text: '数量', align: 'center', value: 'item_num', sortable: true},
          {text: '操作', align: 'center',  sortable: false}
        ],
        show: false,// 控制对话框的显示
        oldGoods: {}, // 即将被编辑的品牌数据
        isEdit: false, // 是否是编辑
      }
    },
    mounted() { // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    watch: {
      pagination: { // 监视pagination属性的变化
        deep: true, // deep为true，会监视pagination的属性及属性中的对象属性变化
        handler() {
          // 变化后的回调函数，这里我们再次调用getDataFromServer即可
          this.getDataFromServer();
        }
      },
      search: { // 监视搜索字段
        handler() {
          this.getDataFromServer();
        }
      }
    },
    methods: {
      getDataFromServer() { // 从服务的加载数的方法。
        // // 发起请求
        this.$http.get("/item/list/page", {
          params: {
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending// 是否降序
          }
        }).then(resp => { // 这里使用箭头函数
          this.goods = resp.data.items;
          this.totalGoods = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        })
        // this.goods = [
        //   {item_id:1, item_name: '三星', item_category:'123', item_num:'S'},
        //   {item_id:1, item_name: '三星', item_category:'123', item_num:'S'},
        //   {item_id:1, item_name: '三星', item_category:'321', item_num:'S'},
        // ];
      },
      addGoods() {
        // 修改标记
        this.isEdit = false;
        // 控制弹窗可见：
        this.show = true;
        // 把oldGoods变为null
        this.oldGoods = null;
      },
      editGoods(oldGoods){
        // // 根据品牌信息查询商品分类
        // this.$http.get("/item/list/gid/" + oldGoods.id)
        //   .then(({data}) => {
        //     // 修改标记
            this.isEdit = true;
            // 控制弹窗可见：
            this.show = true;
            // 获取要编辑的brand
            this.oldGoods = oldGoods;
            console.log("此处是oldGoods")
            console.log(this.oldGoods);
        //     // 回显商品分类
        //     this.oldGoods.categories = data;
        //   })
      },
      closeWindow(){
        // 重新加载数据
        this.getDataFromServer();
        // 关闭窗口
        this.show = false;
      }
    },
    components:{
        GoodsForms
    }
  }
</script>

<style scoped>

</style>
