<template>
  <div>
    <!--输入框 & 按钮组-->
    <el-row>
      <el-col :span="3" :offset="4" id="nameText">
        <span>请输入玩家名称：</span>
      </el-col>
      <el-col :span="5">
        <el-input
          type="text"
          v-model="player.name"
          @keyup.enter.native="addPlayer"
        />
      </el-col>
      <el-col :span="11">
        <el-button type="primary" @click="addPlayer">Add</el-button>
        <el-button type="danger" @click="deletePlayer">Delete</el-button>
        <el-button type="danger" @click="removePlayer">Restart</el-button>
        <el-button type="warning" @click="grouping">Grouping</el-button>
        <el-button type="warning" @click="settingPrice">Price</el-button>
        <el-button type="warning" @click="getTotal">Total</el-button>
      </el-col>
    </el-row>

    <!--分组卡片 & 玩家卡片 & 胜场信息-->
    <el-row>
      <el-col :span="8" :offset="4">
        <el-card id="afterGrouping">
          <el-row>
            <el-col :span="11" class="groupOne">
              <el-card>
                <div
                  v-for="(member, index) in firstGroup"
                  :key="index"
                  style="margin-top: 20px; font-size: 25px"
                >
                  {{ firstGroup[index] }}
                </div>
              </el-card>
              <el-button
                type="success"
                round
                style="margin-top: 10px"
                id="firstGroup"
                @click="noteWin($event)"
              >
                获胜
              </el-button>
            </el-col>
            <el-col :span="11" class="groupTwo" :offset="2">
              <el-card>
                <div
                  v-for="(member, index) in secondGroup"
                  :key="index"
                  style="margin-top: 20px; font-size: 25px"
                >
                  {{ secondGroup[index] }}
                </div>
              </el-card>
              <el-button
                type="success"
                round
                style="margin-top: 10px"
                id="secondGroup"
                @click="noteWin($event)"
              >
                获胜
              </el-button>
            </el-col>
          </el-row>
        </el-card>
      </el-col>
      <el-col :span="4" :offset="1">
        <el-card id="playerName">
          <h3 style="margin-top: 1px">人员名单</h3>
          <div
            v-for="(person, index) in players"
            :key="index"
            style="margin-top: 10px"
          >
            {{ players[index].name }}
          </div>
        </el-card>
      </el-col>
      <!--胜场信息-->
      <el-col :span="4" :offset="1">
        <el-card id="playerName">
          <h3 style="margin-top: 1px">胜场信息</h3>
          <div v-if="winMsg.length > 1">
            <div
              v-for="(person, index) in winMsg"
              :key="index"
              style="margin-top: 8px"
            >
              {{ winMsg[index].name }} : {{ winMsg[index].win }} 胜
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!--总计表格-->
    <el-popover placement="right" width="400" trigger="click">
      <el-table :data="tableData" style="width: 91%" :border="true">
        <el-table-column prop="name" label="姓名" width="162"></el-table-column>
        <el-table-column prop="win" label="胜场" width="162"> </el-table-column>
        <el-table-column prop="totalPrice" label="总计奖金" width="181"> </el-table-column>
      </el-table>
    </el-popover>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      player: {
        name: "",
        win: 0,
      },
      price: 0,
      players: [],
      firstGroup: [],
      secondGroup: [],
      winMsg: [],
      tableData: [],
    };
  },
  methods: {
    addPlayer: function () {
      if (this.player.name != "") {
        this.players.push(this.player);
        //清空名字
        this.player = {
          name: "",
          win: 0,
        };
        this.$notify({
          title: "成功",
          message: "添加成功,玩家进入队列",
          type: "success",
        });
      } else {
        this.$notify({
          title: "失败",
          message: "添加失败,玩家名不能为空",
          type: "warning",
        });
      }
    },
    removePlayer: function () {
      this.players.splice(0, this.players.length);
      this.firstGroup.splice(0, this.firstGroup.length);
      this.secondGroup.splice(0, this.secondGroup.length);
      this.$notify({
        title: "成功",
        message: "删除成功,所有玩家离开队列",
        type: "success",
      });
    },
    deletePlayer: function () {
      this.$prompt("请输入要删除的玩家名字", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(({ value }) => {
          for (var i = 0; i < this.players.length; i++) {
            if (this.players[i].name == value) {
              this.players.splice(i, 1);
              this.$notify({
                title: "成功",
                message: "删除成功,此玩家离开队列",
                type: "success",
              });
            }
          }
        })
        .catch(() => {
          this.$notify.info({
            message: "取消删除",
          });
        });
    },
    grouping: function () {
      if (this.players.length == 0) {
        this.$message({
          message: "分组失败,无玩家",
          type: "warning",
        });
      } else {
        //把玩家加入胜场信息组
        this.winMsg = this.players;
        //初始化分组数组
        this.firstGroup = [];
        this.secondGroup = [];
        var number = [];
        //将与玩家数相同的索引插入
        for (var i = 0; i < this.players.length; i++) {
          number.push(i);
        }
        //打乱算法
        for (var j = 0, len = number.length; j < len; j++) {
          var currentRandom = parseInt(Math.random() * (len - 1));
          var current = number[j];
          number[j] = number[currentRandom];
          number[currentRandom] = current;
        }
        var member = Math.ceil(number.length / 2);
        for (var k = 0; k < number.length; k++) {
          if (k < member) {
            this.firstGroup.push(this.players[number[k]].name);
          } else {
            this.secondGroup.push(this.players[number[k]].name);
          }
        }
        this.$message({
          message: "分组成功",
          type: "success",
        });
      }
    },
    noteWin: function (event) {
      var isWinner = event.target.getAttribute("id");
      if ("firstGroup" == isWinner) {
        //再次分组前两组信息不会初始化
        //更新玩家胜场信息
        for (var i = 0; i < this.firstGroup.length; i++) {
          for (var j = 0; j < this.winMsg.length; j++) {
            if (this.winMsg[j].name == this.firstGroup[i]) {
              this.winMsg[j].win = this.winMsg[j].win + 1;
            }
          }
        }
        for (var i = 0; i < this.secondGroup.length; i++) {
          for (var j = 0; j < this.winMsg.length; j++) {
            if (this.winMsg[j].name == this.secondGroup[i]) {
              this.winMsg[j].win = this.winMsg[j].win - 1;
            }
          }
        }
      } else {
        //再次分组前两组信息不会初始化
        //更新玩家胜场信息
        for (var i = 0; i < this.secondGroup.length; i++) {
          for (var j = 0; j < this.winMsg.length; j++) {
            if (this.winMsg[j].name == this.secondGroup[i]) {
              this.winMsg[j].win = this.winMsg[j].win + 1;
            }
          }
        }
        for (var i = 0; i < this.firstGroup.length; i++) {
          for (var j = 0; j < this.winMsg.length; j++) {
            if (this.winMsg[j].name == this.firstGroup[i]) {
              this.winMsg[j].win = this.winMsg[j].win - 1;
            }
          }
        }
      }
    },
    settingPrice: function () {
      this.$prompt("请输入单场奖金", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(({ value }) => {
          this.price = value;
        })
        .catch(() => {
          this.$notify.info({
            message: "取消金额设置",
          });
        });
    },
    getTotal: function () {
      this.tableData = [];
      for (var i = 0; i < this.winMsg.length; i++) {
        this.tableData.push({
          name: this.winMsg[i].name,
          win: this.winMsg[i].win,
          totalPrice: this.price * this.winMsg[i].win,
        });
      }
    },
  },
};
</script>

<style>
#playerName {
  margin-top: 20px;
  height: 360px;
}

#afterGrouping {
  margin-top: 20px;
  height: 360px;
}

#nameText {
  background-color: red;
  height: 40px;
  padding-top: 7px;
  padding-left: 18px;
  font-size: 20px;
}
</style>