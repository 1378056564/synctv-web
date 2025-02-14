<script setup lang="ts">
import { onMounted, ref } from "vue";
import { ElNotification } from "element-plus";
import { roomListApi } from "@/services/apis/room";
import type { RoomList } from "@/types/Room";
import JoinRoom from "@/views/JoinRoom.vue";
const __roomList = ref<RoomList[]>([]);

const JoinRoomDialog = ref(false);

const formData = ref({
  roomId: "",
  password: "",
  username: localStorage.getItem("uname") || "",
  userPassword: localStorage.getItem("uPasswd") || ""
});

const openJoinRoomDialog = (item: RoomList) => {
  // if (!hasPwd && hasUname) return joinRoom(localStorage.uname, RoomID, "");

  formData.value.roomId = item.roomId;
  JoinRoomDialog.value = true;
};

const { state: roomList, execute: reqRoomList } = roomListApi();
const totalItems = ref(0);
const currentPage = ref(1);
const pageSize = ref(10);
const sort = ref("peopleNum");
const order = ref("desc");

const getRoomList = async (showMsg = false) => {
  __roomList.value = [];
  try {
    await reqRoomList({
      params: {
        page: currentPage.value,
        max: pageSize.value,
        sort: sort.value,
        order: order.value
      }
    });
    if (roomList.value && roomList.value.list) {
      totalItems.value = roomList.value.total;
      for (let i = 0; i < roomList.value.list.length; i++) {
        __roomList.value.push(roomList.value.list[i]);
      }
    }

    showMsg &&
      ElNotification({
        title: `更新列表成功`,
        type: "success"
      });
  } catch (err: any) {
    console.error(err.message);
    ElNotification({
      title: "错误",
      message: err.response.data.error || err.message,
      type: "error"
    });
  }
};

onMounted(() => {
  getRoomList();
});
</script>

<template>
  <div class="card mx-auto lg:w-9/12 max-sm:rounded-none">
    <div class="card-title flex flex-wrap justify-between">
      <div>房间列表（{{ __roomList.length }}）</div>
      <div class="text-base">
        排序方式：<el-select
          v-model="sort"
          class="m-2"
          placeholder="排序方式"
          @change="getRoomList(false)"
        >
          <el-option label="房间ID" value="roomId" />
          <el-option label="房间人数" value="peopleNum" />
          <el-option label="创建人首字母" value="creator" />
          <el-option label="创建时间" value="createdAt" />
          <el-option label="是否有密码" value="needPassword" />
        </el-select>
        <button
          class="btn btn-dense"
          @click="
            order === 'desc' ? (order = 'asc') : (order = 'desc');
            getRoomList();
          "
        >
          {{ order === "asc" ? "👆" : "👇" }}
        </button>
      </div>
    </div>
    <div class="card-body flex flex-wrap justify-center">
      <el-empty v-if="__roomList.length === 0" description="无房间，去创建一个吧~" />
      <div
        v-else
        v-for="item in __roomList"
        :key="item.roomId"
        class="flex flex-wrap m-2 rounded-lg bg-stone-50 hover:bg-white transition-all dark:bg-zinc-800 hover:dark:bg-neutral-800 max-w-[225px]"
      >
        <div class="overflow-hidden text-ellipsis m-auto p-2 w-full">
          <b class="block text-base font-semibold truncate"> {{ item["roomId"] }}</b>
        </div>
        <div class="overflow-hidden text-ellipsis text-sm p-2">
          <div>
            在线人数：<span :class="item.peopleNum > 0 ? 'text-green-500' : 'text-red-500'">{{
              item["peopleNum"]
            }}</span>
          </div>
          <div class="truncate">创建者：{{ item.creator }}</div>
          <div>创建时间：{{ new Date(item.createdAt).toLocaleString() }}</div>
        </div>
        <div class="flex p-2 w-full justify-between items-center">
          <el-tag disabled :type="item.needPassword ? 'danger' : 'success'">
            {{ item.needPassword ? "有密码" : "无密码" }}
          </el-tag>
          <button class="btn btn-dense" @click="openJoinRoomDialog(item)">
            加入房间
            <PlayIcon class="inline-block" width="18px" />
          </button>
        </div>
      </div>
    </div>

    <div class="card-footer justify-between flex-wrap overflow-hidden">
      <button class="btn btn-success max-sm:mb-4" @click="getRoomList(true)">更新列表</button>
      <el-pagination
        v-if="__roomList.length > 0"
        v-model:current-page="currentPage"
        v-model:page-size="pageSize"
        :pager-count="4"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totalItems"
        @size-change="getRoomList(false)"
        @current-change="getRoomList(false)"
      />
    </div>
  </div>

  <el-dialog v-model="JoinRoomDialog" class="rounded-lg dark:bg-zinc-800 w-[443px] max-sm:w-[90%]">
    <template #title>
      <div class="overflow-hidden text-ellipsis">
        <span class="truncate">加入房间 {{ formData.roomId }}</span>
      </div>
    </template>
    <JoinRoom :item="formData" />
  </el-dialog>
</template>
