<template>
  <div class="layout-container" id="app-chat">
    <router-view></router-view>
    <searchfriend></searchfriend>
    <creategroup></creategroup>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import searchfriend from './page/friend/searchfriend'
import creategroup from './page/group/creategroup'
export default {
   name : 'AppChat',
   components: {
     searchfriend,
     creategroup,
   }
}
</script>

<style scoped>
.layout-container {
    width: 100%;
    height: 100%;
    min-width: 920px;
    min-height: 550px;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column
}

</style>


