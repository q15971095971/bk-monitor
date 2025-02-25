<!-- eslint-disable vue/no-deprecated-dollar-listeners-api -->
<!--
* Tencent is pleased to support the open source community by making
* 蓝鲸智云PaaS平台 (BlueKing PaaS) available.
*
* Copyright (C) 2021 THL A29 Limited, a Tencent company.  All rights reserved.
*
* 蓝鲸智云PaaS平台 (BlueKing PaaS) is licensed under the MIT License.
*
* License for 蓝鲸智云PaaS平台 (BlueKing PaaS):
*
* ---------------------------------------------------
* Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
* documentation files (the "Software"), to deal in the Software without restriction, including without limitation
* the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and
* to permit persons to whom the Software is furnished to do so, subject to the following conditions:
*
* The above copyright notice and this permission notice shall be included in all copies or substantial portions of
* the Software.
*
* THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO
* THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
* AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF
* CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
* IN THE SOFTWARE.
-->

<template>
  <div>
    <bk-table
      ref="resultTable"
      :class="[
        'bklog-origin-list',
        { 'is-hidden-index-column': !tableShowRowIndex, 'is-show-index-column': tableShowRowIndex },
      ]"
      :data="tableList"
      :outer-border="false"
      :show-header="false"
      @header-dragend="handleHeaderDragend"
      @row-click="tableRowClick"
    >
      <!-- 展开详情 -->
      <bk-table-column
        width="30"
        align="center"
        type="expand"
      >
        <template #default="{ $index, row }">
          <LazyRender :delay="1">
            <expand-view
              :kv-show-fields-list="kvShowFieldsList"
              :data="originTableList[$index]"
              :list-data="tableList[$index]"
              :retrieve-params="retrieveParams"
              :total-fields="totalFields"
              :visible-fields="visibleFields"
              @value-click="
                (type, content, isLink, field, depth) => handleIconClick(type, content, field, row, isLink, depth)
              "
            >
            </expand-view>
          </LazyRender>
        </template>
      </bk-table-column>

      <bk-table-column
        :width="tableShowRowIndex ? 50 : 0"
        type="index"
        class-name="bklog-result-list-col-index"
      ></bk-table-column>

      <!-- 显示字段 -->
      <template>
        <bk-table-column :width="originFieldWidth">
          <template #default="{ row }">
            <LazyRender :visibleOnly="!tableJsonFormat">
              <span
                class="time-field"
                :title="isWrap ? '' : getOriginTimeShow(row[timeField])"
              >
                {{ getOriginTimeShow(row[timeField]) }}
              </span>
            </LazyRender>
          </template>
        </bk-table-column>
        <bk-table-column>
          <!-- eslint-disable-next-line -->
          <template slot-scope="{ row, column, $index }">
            <LazyRender
              :delay="1"
              :visibleOnly="!tableJsonFormat"
            >
              <JsonFormatter
                :jsonValue="row"
                :fields="getShowTableVisibleFields"
                :formatJson="formatJson"
                class="bklog-column-container"
                @menu-click="({ option, isLink }) => handleMenuClick(option, isLink)"
              ></JsonFormatter>
            </LazyRender>
          </template>
        </bk-table-column>
      </template>
      <!-- 操作按钮 -->
      <bk-table-column
        v-if="showHandleOption"
        :label="$t('操作')"
        :resizable="false"
        :width="getOperatorToolsWidth"
        align="right"
        fixed="right"
      >
        <!-- eslint-disable-next-line -->
        <template slot-scope="{ row, column, $index }">
          <LazyRender>
            <operator-tools
              :handle-click="event => handleClickTools(event, row, operatorConfig)"
              :index="$index"
              :operator-config="operatorConfig"
              :row-data="row"
              log-type="origin"
            />
          </LazyRender>
        </template>
      </bk-table-column>
      <template #empty>
        <empty-view />
      </template>
    </bk-table>
  </div>
</template>

<script>
  import resultTableMixin from '../../mixins/result-table-mixin';
  import JsonFormatter from '../../../../global/json-formatter.vue';
  import { mapState } from 'vuex';

  export default {
    name: 'OriginalList',
    mixins: [resultTableMixin],
    components: { JsonFormatter },
    inheritAttrs: false,
    computed: {
      ...mapState({
        formatJson: state => state.tableJsonFormat,
      }),
      scrollContent() {
        return document.querySelector('.result-scroll-container');
      },
    },
    activated() {
      // keep-alive之后再进入到原始日志时需要重新布局表格
      this.$refs.resultTable.doLayout();
    },
  };
</script>
<style lang="scss">
  .bklog-origin-list {
    .bk-table-row {
      td {
        vertical-align: top;
      }
    }

    .time-field {
      font-weight: 700;
    }
  }

  .bklog-column-container {
    padding: 8px 0;
  }
</style>
