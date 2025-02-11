<script setup>
import { ref } from "vue";
import { statuses } from '@/const/statuses';

let items = ref(JSON.parse(localStorage.getItem("items")) || []);
let inputContent = ref(); // タスクの内容
let inputLimit = ref(); // タスクの期限
let inputState = ref(); // タスクのステータス
let isErrMsg = ref(false);
let isShowModel = ref(false);
let deleteItemId = ''; // 削除対象のItemのID
let isOnEditOther = false;
let errMsg = ref(''); // エラーメッセージの内容

function onEdit(id) {
    // タスクの編集を行う
    // 他に編集モードのタスクがないか調べる
    items.value.map((item) => {
        if (item.onEdit) {
            isOnEditOther = true;
            return;
        }
    });
    if (isOnEditOther) {
        // エラーメッセージを表示する
        errMsg.value = "他に編集中のタスクがあります";
        isErrMsg.value = true;
        return;
    }
    inputContent.value = items.value[id].content;
    inputLimit.value = items.value[id].limit;
    inputState.value = items.value[id].state;
    items.value[id].onEdit = true;
}

function onUpdate(id) {
    // タスクの上書き保存を行う
    if (inputContent.value == "" || inputLimit.value == "") {
        errMsg.value = "タスク・期限を空にできません"
        isErrMsg.value = true;
        return;
    }
    const newItem = {
        id: id,
        content: inputContent.value,
        limit: inputLimit.value,
        state: inputState.value,
        onEdit: false,
    };
    items.value.splice(id, 1, newItem);
    localStorage.setItem("items", JSON.stringify(items.value))
    isErrMsg.value = false;
    isOnEditOther = false;
}

function showDeleteModal(id) {
    // モーダルを開く処理
    isShowModel.value = true;
    deleteItemId = id;
}

function onDeleteItem() {
    // タスクを削除する処理
    items.value.splice(deleteItemId, 1);
    // IDを振りなおす
    items.value = items.value.map((item, index) => ({
        id: index,
        content: item.content,
        limit: item.limit,
        state: item.state,
        onEdit: item.onEdit,
    }));
    localStorage.setItem("items", JSON.stringify(items.value));
    isShowModel.value = false;
}

function onHideModal() {
    // モーダルを閉じる処理
    isShowModel.value = false;
}
</script>

<template>
    <div v-if="isShowModel" class="modal">
        <div class="modal-content">
            <p>削除してもよろしいですか？</p>
            <button @click="onDeleteItem()">はい</button>
            <button @click="onHideModal()">いいえ</button>
        </div>
    </div>
    <p v-if="isErrMsg">{{ errMsg }}</p>
    <div>
        <table>
            <tr>
                <th class="th-id">ID</th>
                <th class="th-value">やること</th>
                <th class="th-limit">期限</th>
                <th class="th-state">状態</th>
                <th class="th-edit">編集</th>
                <th class="th-delete">削除</th>
            </tr>
            
            <!-- タスクの件数分表示 -->
            <tr v-for="item in items" :key="item.id">
                <td>{{ item.id }}</td>
                <td>
                    <span v-if="!item.onEdit">{{ item.content }}</span>
                    <input v-else v-model="inputContent" type="text" />
                </td>
                <td>
                    <span v-if="!item.onEdit">{{ item.limit }}</span>
                    <input v-else v-model="inputLimit" type="date">
                </td>
                <td>
                    <span v-if="!item.onEdit">{{ item.state.value }}</span>
                    <select v-else v-model="inputState">
                        <option
                            v-for="state in statuses"
                            :key="state.id"
                            :value="state"
                            :selected="state.id == item.state.id"
                        >
                            {{ state.value }}
                        </option>
                    </select>
                </td>
                <td>
                    <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
                    <button v-else @click="onUpdate(item.id)">完了</button>
                </td>
                <td><button @click="showDeleteModal(item.id)">削除</button></td>
            </tr>
        </table>
    </div>
</template>

<style scoped>
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal-content {
    background: #fff;
    color: gray;
    padding: 20px;
    border-radius: 8px;
}
</style>