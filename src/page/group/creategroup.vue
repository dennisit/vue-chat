<template>
   <div class="create-group">
       <el-dialog
        :visible.sync="showCreateGroupDialog"
        width="45%"
        :show-close="false"
        @close="handleClose"
        :distroy-on-close="true"
        center>
            <div class="content" >
                <div class="friendlist-area">
                    <div class="search-input">
                        <el-input clearable v-model="friendInput" prefix-icon="el-icon-search" placeholder="搜索" @keydown.enter.native="searchUser"></el-input>
                    </div>
                    <div class="friendlist" :style="{height: (appHeight-170) + 'px'}">
                        <ul>
                            <li v-bind:key = index v-for="(item, index) in waitCheckedFriendList" class="frienditem"  :class="{ noborder: !item.initial}">
                                <div class="list_title" v-if="item.initial">{{item.initial}}</div>
                                <div class="friend-info" :class="{ active: item.id === selectFriendId }" @click.stop="selectFriend(item.id)">
                                    <img class="avatar" :src="item.img" onerror="this.src='static/images/vue.jpg'">
                                    <div class="remark">{{item.remark}}</div>
                                    <div class="friend-check">
                                        <el-checkbox :true-label="item.id+':1'" :false-label="item.id+':0'" @change="friendChangeChange"  @click.stop.native="" v-model="item.checked"></el-checkbox>
                                    </div>
                                </div>
                                
                            </li>
                        </ul>
                    </div>
                    
                </div>
                <div class="checkedlist">
                    <div class="checked-title">
                        <div class="create-group-title">创建群聊</div>
                        <div class="check-statu-title">{{checkFriendTips}}</div>
                    </div>
                    <div class="friendlist" :style="{height: (appHeight-200) + 'px'}">
                        <ul>
                            <li v-bind:key = index v-for="(item, index) in selectedFriends" class="frienditem">
                                <div class="friend-info" >
                                    <img class="avatar" :src="item.img" onerror="this.src='static/images/vue.jpg'">
                                    <div class="remark">{{item.remark}}</div>
                                    <div class="delete" @click="delfriend(item.id)"></div>
                                </div>
                            </li>
                        </ul>
                    </div>
                    <div class="check-operate">
                        <div class="check-btns">
                            <el-button class="cancel-btn" size="medium" type="info" plain round @click="cancel">取 消</el-button>
                            <el-button class="confirm-btn"  size="medium" type="success" plain round @click="confirm" :disabled="confirmEnable">确 定</el-button>
                        </div>
                        
                    </div>
                </div>
            </div>
       </el-dialog>
   </div>
    
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import Logger from '../../websocket/utils/logger'
export default {
    name: 'creategroup',
    data() {
       return {
           friendInput: '',
           selectedFriends: [],
           checkFriendTips: '未选择联系人',
           confirmEnable: true,
           selectFriendId: 0
       }
    },
    computed: {
        ...mapState([
            'appHeight',
        ]),
        ...mapGetters([
            'searchedFriendlist'
        ]),
        showCreateGroupDialog : {
            get () {
                Logger.log("showCreateGroupDialog"+this.$store.state.showCreateGroupDialog);
                return this.$store.state.showCreateGroupDialog;
            },
            set(val) {
                this.$store.state.showCreateGroupDialog = val;
            }
        },
        waitCheckedFriendList(){
            let friends = this.searchedFriendlist.slice(1,this.searchedFriendlist.length);
            var listunCheckedFriends = [];
            for(var friend of friends){
                // console.log("friend only initial "+friend.initial);
                listunCheckedFriends.push({
                    id: friend.id,
                    wxid: friend.wxid,
                    remark: friend.remark,
                    img: friend.img,
                    initial: friend.initial,
                    checked: false
                });
            }
            return listunCheckedFriends;
        }
   },
   methods:{
       friendChangeChange(event) {
           Logger.log("check change "+event);
           if(event && event != ""){
                var friendIdAndChecked = event.split(":");
                Logger.log("select friendid "+friendIdAndChecked[0]+" checked "+friendIdAndChecked[1]);
                var friend = this.waitCheckedFriendList.find(friend => friend.id == friendIdAndChecked[0]);
                Logger.log("friend change "+friend.checked);
                if(friendIdAndChecked[1] == 1){
                    if(friend){
                        this.selectFriendId = friend.id;
                        var existFriend = this.selectedFriends.find(friend => friend.id == this.selectFriendId);
                        if(!existFriend){
                            this.selectedFriends.push(friend);
                        }
                    }
                } else if(friendIdAndChecked[1] == 0){
                    if(friend){
                        this.removeCheckedFriend(friend.id);
                    }
                }    
           }
       },
       selectFriend(friendId){
           this.selectFriendId = friendId;
           var friend = this.waitCheckedFriendList.find(friend => friend.id == friendId);
           Logger.log("selectFriend friendId "+friendId+" checked "+friend.checked);
           friend.checked = !friend.checked;
           if(!friend.checked){
               this.removeCheckedFriend(friend.id);
           } else {
               this.selectedFriends.push(friend);
           }
       },
       cancel(){ 
            this.exit();
       },
       confirm(){
            this.exit();
       },
       handleClose(){
            this.exit();
       },
       delfriend(id){
          var friend = this.waitCheckedFriendList.find(friend => friend.id == id);
          friend.checked = false;
          this.removeCheckedFriend(id);
       },
       removeCheckedFriend(id){
            var index = -1;
            for(var i=0;i<this.selectedFriends.length;i++){
                if(id == this.selectedFriends[i].id){
                    index = i;
                    break;
                }
            }
            if(index != -1){
                this.selectedFriends.splice(index,1);
            }
            if(id == this.selectFriendId){
               this.selectFriendId = 0;
            }
       },
       exit(){
           this.$store.state.showCreateGroupDialog = false;
           this.selectedFriends = [];
           for(var friend of this.waitCheckedFriendList){
                friend.checked = false;
           }           
       }
   },
   watch: {
       selectedFriends() {
           if(this.selectedFriends.length == 0){
              this.selectFriendId = 0;
           }
           if(this.selectedFriends.length > 1){
              this.confirmEnable = false;
           } else {
               this.confirmEnable = true;
           }
           if(this.selectedFriends.length == 0){
               this.checkFriendTips = "未选择联系人";
           } else {
               this.checkFriendTips = "已选择"+this.selectedFriends.length+"位联系人";
           }
       }
   }
}
</script>

<style lang="stylus" scoped>
.content
  display: flex
  width: 100%
  height: 50%
  .friendlist-area
    width: 50%
    padding: 2px 0px 0px 0px
    background: #fff
    border-right: 1px solid #e7e7e7
    .search-input
        margin-right: 10px
    .friendlist
        overflow-y: auto
        .frienditem
            border-top: 1px solid #dadada
            &:first-child,&.noborder
                border-top: none
            .list_title
                box-sizing: border-box
                width: 100%
                font-size: 12px
                padding: 15px 0 3px 12px
                color: #999
            .friend-info
                display: flex
                padding: 5px
                transition: background-color .1s
                font-size: 0
                &:hover 
                    background-color: rgb(220,220,220)
                &.active 
                    background-color: #c4c4c4
                .avatar
                    border-radius: 2px
                    margin-right: 12px
                    width: 35px
                    height: 35px
                .remark
                    font-size: 14px
                    line-height: 26px
                    display: flex
                    align-items: center 
                .friend-check
                    margin-left : auto 
                    display: flex
                    padding-right: 5px
                    align-items: center
  .checkedlist
    display: flex
    flex-flow: row wrap
    padding: 2px 0px 2px 10px
    width: 50%
    .checked-title
        display: flex
        padding: 5px 0px 10px 0px
        width: 100%
        height: 30px
        .create-group-title
            font-size: 14px
            color: black 
        .check-statu-title
            font-size: 10px
            color: #999
            line-height: 15px
            margin-left : auto
    .check-operate
        width: 100%
        align-self: flex-end
        .check-btns
            width: 164px
            margin-left : auto 
            display: flex
    .friendlist
        overflow-y: auto
        width: 100%
        .frienditem
            padding: 5px 0px 5px 0px
            .friend-info
                display: flex
                padding: 5px
                .avatar
                    border-radius: 2px
                    margin-right: 12px
                    width: 35px
                    height: 35px
                .delete
                    margin-left : auto 
                    outline: none
                    width: 35px
                    height: 35px
                    background-size: 26px
                    background-position: center
                    background-repeat: no-repeat
                    background-image: url(delete.png)
                    cursor: pointer
                .remark
                    font-size: 14px
                    line-height: 26px
                    color: black
                    display: flex
                    align-items: center   

.el-dialog__wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>