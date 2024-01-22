<template>
  <template v-if="chat && chat.id">
    <ChatGc
      v-if="qa && qa.length && qa[0].content.indexOf('(') == 0"
      :name="qa[0].name"
      :prompt="qa[0].content"
    />
    <Chat
      v-else
      :chat-id="chat.id"
      :data="qa"
      @qa="addChatItems"
      @replaceAllChatItems="replaceAllChatItems"
    ></Chat>
  </template>
</template>
<script setup>
import Chat from "@/components/Chat";
import ChatGc from "@/components/ChatGc";
import { listChatItem, del } from "@/repo/chatItemRepository";
import { get, save } from "@/repo/chatRepository";
import { saveChatItems } from "@/service/chatService";
import { computedAsync } from "@vueuse/core";
const props = defineProps(["id"]);

const chat = computedAsync(async () => {
  return props.id && (await get(Number.parseInt(props.id)));
}, {});

const qa = computedAsync(
  async () => {
    return props.id && (await listChatItem(Number.parseInt(props.id)));
  },
  [] // initial state
);

async function addChatItems(items) {
  // 更新会话名称
  if (qa.value.length == 0) {
    const o = Object.assign({}, chat.value, { name: items[0].content });
    await save(o);
    chat.value = o;
  }
  await saveChatItems(
    items.map((msg) => ({
      chatId: Number.parseInt(msg.chatId),
      role: msg.role,
      content: msg.content,
    }))
  );
  qa.value = await listChatItem(Number.parseInt(props.id));
}

async function replaceAllChatItems(items) {
  del(Number.parseInt(items[0].chatId));
  addChatItems(items);
}
</script>