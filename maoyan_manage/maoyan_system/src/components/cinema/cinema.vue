<template>

<div>
    <div style="margin-top: 15px;display:flex;">
 <el-button style="width:300px" @click="add_cinema_btn">增加影院</el-button>
 <el-button style="width:300px;margin-right:10px" @click="refresh">刷新列表</el-button>
  <el-input placeholder="请输入搜索的内容" v-model="searchval" class="input-with-select">
    <el-select v-model="select" slot="prepend" placeholder="请选择类型" style="width:130px">
      <el-option label="影院名" value="cinema_name"></el-option>
      <el-option label="地址" value="address"></el-option>
    </el-select>
    <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
  </el-input>
    </div>
    
  <el-table :data="tableData"
    stripe style="width: 100%">
    
    <el-table-column fixed type="index"
      label="序号" width="100">
    </el-table-column>
    
    <el-table-column fixed prop="cinema_name"
      label="影院名称" width="200">
    </el-table-column>
    
    <el-table-column prop="address"
      label="地址" width="400">
    </el-table-column>

    <el-table-column fixed="right"
      label="操作" width="300">
      <template slot-scope="scope">
        <el-button @click="handleClick(scope.row)" type="text" size="small">删除</el-button>
        <el-button @click="handlefix(scope.row)" type="text" size="small">编辑</el-button>
      </template>
    </el-table-column>
  </el-table>

  <el-pagination
  background
  layout="prev, pager, next" id='page' :total=maxpage*10 @current-change='handleCurrentChange'
        @prev-click='uppage'  @next-click='downpage'>
</el-pagination>

<!-- //增加信息的弹出矿 -->
<el-dialog title="增加新的影院信息" :visible.sync="dialogFormVisible">
  <el-form :model="form">
    <!-- 影院名称 -->
    <el-form-item label="影院名称" :label-width="formLabelWidth">
      <el-input v-model="form.name" auto-complete="off" placeholder="请输入新增影院名称"
      style="width:300px"></el-input>
    </el-form-item>
     <!-- 影院地址 -->
    <el-form-item label="影院地址" :label-width="formLabelWidth">
      <el-input v-model="form.address" auto-complete="off" placeholder="请输入新增影院地址"
      style="width:300px"></el-input>
    </el-form-item>
    <!-- 设置影院标签 -->
 <el-form-item label="设置该影院标签" :label-width="formLabelWidth">
    <el-checkbox-group v-model="form.checkboxGroup" size="medium">
      <el-checkbox-button v-for="tag in tags" :label="tag" :key="tag">{{tag}}</el-checkbox-button>
    </el-checkbox-group>
  </el-form-item>
  <!-- 点击新增影厅按钮 -->
  <el-form-item label="新增影厅" :label-width="formLabelWidth">
  <el-button  style="width:300px" @click="click_add_hall">点击新增</el-button>
  </el-form-item>
  <!-- 已新增的影厅表格 -->
 <el-form-item label="已增影厅表格" :label-width="formLabelWidth">
 <el-table
      :data="tableData2" border
      style="width: 100%">
      <el-table-column
        prop="hall_name"
        label="影厅名"
        width="255">
      </el-table-column>
      <el-table-column
        prop="name"
        label="操作"
        width="259">
         <template slot-scope="scope">
        <el-button @click="lookClick(scope.row)" type="primary" size="small">查看</el-button>
        <el-button  @click="del_hall_Click(scope.row)" type="primary" size="small">删除</el-button>
      </template>
      </el-table-column>
    </el-table>
</el-form-item>
  </el-form>
  <!-- 新增影院底部 -->
  <div slot="footer" class="dialog-footer">
    <el-button @click="dialogFormVisible = false">取 消</el-button>
    <el-button type="primary" @click="add_submit">确 定</el-button>
  </div>
</el-dialog>


<!-- 设置影厅的弹框 -->
<el-dialog
  title="影厅座位"
  :visible.sync="dialogVisible"
  width="70%"
  :before-close="handleClose" append-to-body>
  <div style="display:flex;align-items:center;margin-bottom:20px">
    <label style="margin-right:50px">影厅名</label><el-input v-model="set_hall_name" placeholder="请输入内容" style="width:300px"></el-input></div>
  <div style="display:flex;align-items:center;margin-bottom:20px">
    <label style="margin-right:50px">选择行</label><el-input-number v-model="seat_rows" :min="5" :max="20" @change="changge_num"></el-input-number></div>
  <div style="display:flex;align-items:center;margin-bottom:20px">
    <label style="margin-right:50px">选择列</label><el-input-number v-model="seat_cols" :min="5" :max="20" @change="changge_num"></el-input-number></div>

<!-- 影厅的弹框座位部分 -->
<div class="seat_view" style="">
  <div class="screen">screen</div>
  <div class="seat_part">
     <div v-for="(item,index) in creat_seat" :key="index" style="display:flex">
       <div  v-for="(val,i) in item" :key="i" class="eve_seat"
                :class="{ has:val=='0',not:val=='1'}"
               @click="changge_seat(index,i,val)">
       </div>
     </div>

  </div>
</div>
  <!-- 影厅的弹框底部 -->
  <span slot="footer" class="dialog-footer">
    <el-button @click="cancle_add_hall">取 消</el-button>
    <el-button type="primary" @click="sum_add_hall">确 定</el-button>
  </span>
</el-dialog>


<!-- 修改电影院信息的弹框 -->
<el-dialog title="修改影院信息" :visible.sync="dialogFormVisible2">
  <el-form :model="form_fix">
    <!-- 影院名称 -->
    <el-form-item label="影院名称" :label-width="formLabelWidth">
      <el-input v-model="form_fix.cinema_name" auto-complete="off" placeholder="请输入修改影院名称"
      style="width:300px"></el-input>
    </el-form-item>
     <!-- 影院地址 -->
    <el-form-item label="影院地址" :label-width="formLabelWidth">
      <el-input v-model="form_fix.address" auto-complete="off" placeholder="请输入修改影院地址"
      style="width:300px"></el-input>
    </el-form-item>
    <!-- 设置影院标签 -->
 <el-form-item label="设置该影院标签" :label-width="formLabelWidth">
    <el-checkbox-group v-model="form_fix.tag" size="medium">
      <el-checkbox-button v-for="tag in tags" :label="tag" :key="tag">{{tag}}</el-checkbox-button>
    </el-checkbox-group>
  </el-form-item>
  <!-- 点击新增影厅按钮 -->
  <el-form-item label="新增影厅" :label-width="formLabelWidth">
  <el-button  style="width:300px" @click="click_add_hall">点击新增</el-button>
  </el-form-item>
  <!-- 已新增的影厅表格 -->
 <el-form-item label="已建影厅表格" :label-width="formLabelWidth">
 <el-table
      :data="tableData2" border
      style="width: 100%">
      <el-table-column
        prop="hall_name"
        label="影厅名"
        width="255">
      </el-table-column>
      <el-table-column
        prop="name"
        label="操作"
        width="259">
         <template slot-scope="scope">
        <el-button @click="lookClick(scope.row)" type="primary" size="small">查看</el-button>
        <el-button  @click="del_hall_Click(scope.row)" type="primary" size="small">删除</el-button>
      </template>
      </el-table-column>
    </el-table>
</el-form-item>

  </el-form>
  <!-- 新增影院底部 -->
  <div slot="footer" class="dialog-footer">
    <el-button @click="dialogFormVisible2 = false">取 消</el-button>
    <el-button type="primary" @click="fix_submit">确 定</el-button>
  </div>
</el-dialog>


</div>
</template>
<script>
import axios from "axios";
let ip = "http://127.0.0.1:3000";
const tagOptions = ["4DX", "IMAX", "小吃", "折扣卡", "杜比全景声厅", "改签"];
export default {
  data() {
    return {
      searchval: "",
      select: "",
      dialogFormVisible: false,
      form: {
        name: "",
        address: "",
        checkboxGroup: ["4DX"]
      },
      formLabelWidth: "120px",
      tags: tagOptions,
      dialogVisible: false,
      seat_rows: 6,
      seat_cols: 12,
      set_hall_name: "",
      //初始数组
      creat_seat: [],
      //影厅数组
      tableData2: [],
      //如点击了查看，保存此条id，关闭修改框后，清除id
      lookid: {
        lookid: "",
        exist: false
      },
      //修改影院框
      dialogFormVisible2: false,
      form_fix: {
        cinema_name: "",
        address: "",
        tag: ["4DX"]
      },
      //是否为修改状态
      isFix: false,
      fix_cinema_id: ""
    };
  },
  created() {
    this.refresh_cm(1, this.select, this.searchval);
  },
  computed: {
    tableData() {
      return this.$store.state.cinema.cinema_list;
    },
    maxpage() {
      return this.$store.state.cinema.maxpage;
    }
  },
  methods: {
    //修改提交
    fix_submit() {
      this.$store.dispatch({
        type: "fix_cinema",
        hall: this.tableData2,
        cinema: this.form_fix,
        _id: this.fix_cinema_id
      });
      this.refresh_cm(1, this.select, this.searchval);

      this.dialogFormVisible2 = false;
    },
    //增加提交
    add_submit() {
      this.$store.dispatch({
        type: "add_cinema",
        hall: this.tableData2,
        cinema: this.form
      });
      this.dialogFormVisible = false;
    },
    //取消修改
    cancle_add_hall() {
      this.dialogVisible = false;
      this.lookid.exist = false;
    },
    changge_num() {
      this.creat_seat_fn();
    },
    //确定提交增加的影厅
    sum_add_hall() {
      //如果为修改状态
      if (this.isFix) {
        if (!this.lookid.exist) {
          //是新增中的新增提交
          let newarr = [...this.tableData2];
          let obj2 = {
            hall_name: this.set_hall_name,
            hall_seat: this.creat_seat
          };
          if (this.tableData2.length == 0) obj2.hall_id = 0;
          else obj2.hall_id = newarr[newarr.length - 1].hall_id + 1;
          //1 加入显示列表显示
          newarr.push(obj2);
          this.tableData2 = newarr;
        } else {
          //是新增中的查看修改提交
          let newarr2 = [...this.tableData2];
          //如果是查看修改，则找到对应的信息，刷新储存
          for (let i = 0; i < newarr2.length; i++) {
            if (newarr2[i].hall_id == this.lookid.lookid) {
              newarr2[i].hall_name = this.set_hall_name;
              newarr2[i].hall_seat = this.creat_seat;
            }
          }
          this.lookid.exist = false;
        }
      } else {
        //如果为新增状态
        if (this.lookid.exist) {
          let newarr2 = [...this.tableData2];
          //如果是查看修改，则找到对应的信息，刷新储存
          for (let i = 0; i < newarr2.length; i++) {
            if (newarr2[i].hall_id == this.lookid.lookid) {
              newarr2[i].hall_name = this.set_hall_name;
              newarr2[i].hall_seat = this.creat_seat;
            }
          }
          this.lookid.exist = false;
        } else {
          //如果非修改已增加的，则加入新数组
          let newarr = [...this.tableData2];
          let obj2 = {
            hall_name: this.set_hall_name,
            hall_seat: this.creat_seat
          };
          if (this.tableData2.length == 0) obj2.hall_id = 0;
          else obj2.hall_id = newarr[newarr.length - 1].hall_id + 1;
          newarr.push(obj2);
          this.tableData2 = newarr;
        }
      }
      this.dialogVisible = false;
    },
    //改变座位
    changge_seat(index, i, val) {
      let newarr = [...this.creat_seat];
      if (newarr[index][i] == 0) newarr[index][i] = 1;
      else newarr[index][i] = 0;
      this.creat_seat = newarr;
    },
    //新增hall
    click_add_hall() {
      this.isfirst = true;
      this.set_hall_name = "";
      this.seat_rows = 6;
      this.seat_cols = 12;
      this.creat_seat_fn();
      this.dialogVisible = true;
    },
    //关闭影厅弹框
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          this.lookid.exist = false;
          done();
        })
        .catch(_ => {});
    },
    //dispatch
    refresh_cm(page, condition, text) {
      this.$store.dispatch({
        type: "get_cinema_list",
        page,
        con: condition,
        text
      });
    },
    //上一页
    uppage() {
      this.refresh_cm(this.nowpage - 1, this.select, this.searchval);
    },
    //下一页
    downpage() {
      if (this.nowpage < this.maxpage)
        this.refresh_cm(this.nowpage + 1, this.select, this.searchval);
    },
    //点击哪页跳哪页
    handleCurrentChange: function(currentPage) {
      this.refresh_cm(currentPage, this.select, this.searchval);
    },
    //搜索
    search() {
      if (this.select == "" || this.searchval == "") {
        this.$message({
          message: "未选择搜索类型或未输入搜索内容",
          showClose: true,
          type: "warning"
        });
      } else {
        this.refresh_cm(1, this.select, this.searchval);
      }
    }, 
    //刷新
    refresh() {
      this.searchval = "";
      this.select = "";
      this.refresh_cm(1, this.select, this.searchval);
    },
    //删除
    handleClick(index) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$store.dispatch({
            type: "delete_cinema",
            _id: index._id,
            ids: index.hall
          });
          this.$message({
            type: "success",
            message: "删除成功!"
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //点击修改按钮
    handlefix(data) {
      this.isFix = true;
      //通过对应的id放松ajax，且打开修改弹框
      axios.post(ip + "/cinema/get_fix_cinema", { _id: data._id }).then(msg => {
        //赋值影院名，影院地址，影院标签，和表格
        this.form_fix.cinema_name = msg.data.cinema_name;
        this.form_fix.address = msg.data.address;
        this.form_fix.tag = msg.data.tag;
        //存一个fix的影院的id
        this.fix_cinema_id = msg.data._id;
        this.tableData2 = msg.data.hall;
      });
      this.dialogFormVisible2 = true;
    },
    //点击新增影院
    add_cinema_btn() {
      this.tableData2 = [];
      this.dialogFormVisible = true;
      this.isFix = false;
    },
    //查看新增的影厅
    lookClick(data) {
      this.dialogVisible = true;
      let newarr = [...this.tableData2];
      for (let i = 0; i < newarr.length; i++) {
        if (data.hall_id == newarr[i].hall_id) {
          this.creat_seat = newarr[i].hall_seat;
          this.set_hall_name = newarr[i].hall_name;
          this.seat_rows = this.tableData2[i].hall_seat.length;
          this.seat_cols = this.tableData2[i].hall_seat[0].length;
          this.lookid.lookid = newarr[i].hall_id;
          this.lookid.exist = true;
        }
      }
    },
    //删除新增的影厅
    del_hall_Click(data) {
      if (this.isFix) {
        //如果是修改状态的删除影厅，先判断是否存在_id
        if (data._id) {
          //如果存在，需要删除数据源
          axios
            .post(ip + "/cinema/fix_del_hall", {
              _id1: this.fix_cinema_id, //院 id
              _id: data._id //厅 id
            })
            .then(msg => {
              //刷新界面
              let newarr = [...this.tableData2];
              for (let i = 0; i < newarr.length; i++) {
                if (newarr[i].hall_id == data.hall_id) newarr.splice(i, 1);
              }
              this.tableData2 = newarr;
            });
        } else {
          //不存在_id只需要删除js中的数组
          let newarr = [...this.tableData2];
          for (let i = 0; i < newarr.length; i++) {
            if (newarr[i].hall_id == data.hall_id) newarr.splice(i, 1);
          }
          this.tableData2 = newarr;
        }
      } else {
        let newarr = [...this.tableData2];
        for (let i = 0; i < newarr.length; i++) {
          if (newarr[i].hall_id == data.hall_id) newarr.splice(i, 1);
        }
        this.tableData2 = newarr;
      }
    },
    // 初始化的座位
    creat_seat_fn(data) {
      let setzhi = [];
      let zhi = [];
      for (let i = 0; i < this.seat_rows; i++) {
        zhi = [];
        setzhi.push(zhi);
        for (let j = 0; j < this.seat_cols; j++) {
          zhi.push(0);
        }
      }
      this.creat_seat = [...setzhi];
    }
  }
};
</script>

<style scoped>
#page {
  position: absolute;
  bottom: 0;
  left: calc(50% - 200px);
}
.seat_view {
  border: 1px solid rgb(218, 216, 216);
  border-radius: 10px;
  display: flex;
  flex-flow: column;
  align-items: center;
}
.screen {
  border: 1px solid rgb(218, 216, 216);
  background-color: rgb(167, 164, 164);
  width: 300px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin-bottom: 50px;
}
.eve_seat {
  width: 30px;
  height: 30px;
  margin: 5px;
}
.has {
  background: rgb(151, 150, 150);
}
.not {
  background: rgb(236, 18, 18);
}
</style>


