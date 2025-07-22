<script setup lang="ts">
import { h, ref } from 'vue';

import { CodeMirror, Page } from '@vben/common-ui';
import { DictEnum } from '@vben/constants';

import {
  Alert,
  Card,
  Descriptions,
  DescriptionsItem,
  RadioGroup,
  Select,
  Space,
} from 'ant-design-vue';
import { repeat } from 'lodash-es';

import { DictTag } from '#/components/dict';
import { getDictOptions } from '#/utils/dict';

const options = getDictOptions(DictEnum.SYS_COMMON_STATUS);

const value = ref('1');

const getOptionsCode = `
const options = getDictOptions(DictEnum.SYS_COMMON_STATUS);
`;
const mountCode = `
const options = reactive([]);
onMounted(async () => {
  const resp = await dictDataInfo(DictEnum.SYS_COMMON_STATUS);
  options.push(...resp);
})
`;
</script>

<template>
  <Page content-class="flex flex-col gap-4">
    <Card size="small" title="核心逻辑">
      <p class="mb-2">
        你可以简单理解为getDictOptions是以下代码的快捷方式(还包括对并发和缓存的处理)
      </p>
      <div class="grid grid-cols-2 gap-4">
        <CodeMirror readonly :model-value="getOptionsCode" language="js" />
        <CodeMirror readonly :model-value="mountCode" language="js" />
      </div>
    </Card>

    <Card size="small">
      <template #title>
        选择器组件使用
        <a
          href="https://dapdap.top/function/dict.html"
          class="text-primary"
          target="_blank"
        >
          文档参考
        </a>
      </template>
      <Space>
        <Select v-model:value="value" :options="options" />
        <RadioGroup v-model:value="value" :options="options" />
        渲染: <DictTag :value="value" :dicts="options" />
      </Space>
      <Alert
        class="mt-2"
        show-icon
        type="success"
        message="getDictOptions返回值为reactive 可直接绑定使用!"
      />
      <Alert
        class="mt-2"
        show-icon
        type="error"
        message="getDictOptions内部为异步实现 不要使用它参与业务运算!"
      />
    </Card>

    <Card size="small" title="字典标签 - 未匹配到值的fallback">
      <Descriptions :column="1">
        <DescriptionsItem label="默认为unknown">
          <DictTag :dicts="options" value="error" />
        </DescriptionsItem>

        <DescriptionsItem label="直接返回string">
          <DictTag :dicts="options" value="error" fallback="自定义的fallback" />
        </DescriptionsItem>

        <DescriptionsItem label="函数返回string">
          <DictTag
            :dicts="options"
            value="error"
            :fallback="(v) => repeat(String(v), 5)"
          />
        </DescriptionsItem>

        <DescriptionsItem label="函数返回VNode">
          <DictTag
            :dicts="options"
            value="error"
            :fallback="
              (v) => h('span', { class: 'text-red-500' }, `${v} 没有匹配到值`)
            "
          />
        </DescriptionsItem>

        <DescriptionsItem label="使用fallback插槽">
          <DictTag :dicts="options" value="error">
            <template #fallback="v">
              <span class="text-red-500"> {{ v }} 跟上面相同的写法 </span>
            </template>
          </DictTag>
        </DescriptionsItem>
      </Descriptions>
    </Card>
  </Page>
</template>
