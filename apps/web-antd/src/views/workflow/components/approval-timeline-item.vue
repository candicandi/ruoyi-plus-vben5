<script setup lang="ts">
import type { Flow } from '#/api/workflow/instance/model';

import { onMounted, ref } from 'vue';

import { VbenAvatar } from '@vben/common-ui';
import { DictEnum } from '@vben/constants';
import { cn } from '@vben/utils';

import { MessageOutlined } from '@ant-design/icons-vue';
import { TimelineItem } from 'ant-design-vue';

import { ossInfo } from '#/api/system/oss';
import { renderDict } from '#/utils/render';

defineOptions({
  name: 'ApprovalTimelineItem',
});

const props = defineProps<{ item: Flow }>();

interface AttachmentInfo {
  ossId: string;
  url: string;
  name: string;
}

/**
 * 处理附件信息
 */
const attachmentInfo = ref<AttachmentInfo[]>([]);
onMounted(async () => {
  if (!props.item.ext) {
    return null;
  }
  const resp = await ossInfo(props.item.ext.split(','));
  attachmentInfo.value = resp.map((item) => ({
    ossId: item.ossId,
    url: item.url,
    name: item.originalName,
  }));
});
</script>

<template>
  <TimelineItem>
    <template #dot>
      <div class="relative rounded-full border">
        <VbenAvatar
          :alt="item?.approveName ?? 'unknown'"
          class="bg-primary size-[36px] rounded-full text-white"
          src=""
        />
        <div
          :class="
            cn(
              'absolute bottom-0 right-[-2px]',
              'size-[12px] rounded-full bg-green-500',
              'border-[2px] border-white',
            )
          "
        ></div>
      </div>
    </template>
    <div class="ml-2 flex flex-col gap-0.5">
      <div class="flex items-center gap-1">
        <div class="font-bold">{{ item.nodeName }}</div>
        <component :is="renderDict(item.flowStatus, DictEnum.WF_TASK_STATUS)" />
      </div>
      <div>{{ item.approveName }}</div>
      <div>{{ item.updateTime }}</div>
      <div
        v-if="item.message"
        class="rounded-lg border px-3 py-1"
        :class="cn('flex gap-2')"
      >
        <MessageOutlined />
        <div class="text-foreground/75 break-all">{{ item.message }}</div>
      </div>
      <div v-if="attachmentInfo.length > 0" class="flex flex-wrap gap-2">
        <!-- 这里下载的文件名不是原始文件名 -->
        <a
          v-for="attachment in attachmentInfo"
          :key="attachment.ossId"
          :href="attachment.url"
          class="text-primary"
          target="_blank"
        >
          <div class="flex items-center gap-1">
            <span class="icon-[mingcute--attachment-line] size-[18px]"></span>
            <span>{{ attachment.name }}</span>
          </div>
        </a>
      </div>
    </div>
  </TimelineItem>
</template>
