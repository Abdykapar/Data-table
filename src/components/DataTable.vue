<script setup>
import moment from "moment";
import data from "../assets/data.json";
import { ref, computed } from "vue";

const isStatusEdit = ref(false);
const isPercentCompletionEdit = ref(false);
const states = ref(data);

const currentRecord = ref({});

const statuses = [
  "initiation",
  "planning",
  "execution",
  "monitoring",
  "closure",
];

const columns = [
  {
    title: "Client",
    dataIndex: "client",
    sorter: (a, b) => a.client.localeCompare(b.client),
    width: "20%",
  },
  {
    title: "Project",
    dataIndex: "project",
    sorter: (a, b) => a.project.localeCompare(b.project),
    width: "20%",
  },
  {
    title: "Start Date",
    dataIndex: "startDate",
    sorter: true,
    width: "20%",
  },
  {
    title: "Revenue",
    dataIndex: "revenue",
    sorter: true,
    width: "22%",
  },
  {
    title: "Status",
    dataIndex: "status",
    filters: [...statuses.map((i) => ({ text: i.toUpperCase(), value: i }))],
    width: "20%",
  },
  {
    title: "Percent of completion",
    dataIndex: "percentCompletion",
    sorter: true,
    width: "20%",
  },
  {
    title: "Margin",
    dataIndex: "margin",
    sorter: true,
  },
];

const totalRevenue = computed(() =>
  states.value.reduce((acc, i) => acc + i.revenue, 0)
);

const handleTableChange = (pagination, filters, sorter) => {
  console.log(filters, sorter);

  const sortNumber = (a, b, order) => {
    return order === "ascend" ? a - b : order === "descend" ? b - a : 0;
  };

  states.value = data
    .filter((i) =>
      Object.entries(filters)
        .filter(([, value]) => value)
        .every(([key, value]) => value.includes(i[key]))
    )
    .sort((a, b) => {
      switch (sorter.field) {
        case "percentCompletion":
        case "margin":
        case "revenue":
          return sortNumber(a[sorter.field], b[sorter.field], sorter.order);
        case "startDate":
          const first = moment(a[sorter.field], "DD.MM.YYYY");
          const second = moment(b[sorter.field], "DD.MM.YYYY");
          if (!sorter.order || first === second) return 0;
          if (first > second) return sorter.order === "ascend" ? 1 : -1;
          else return sorter.order === "ascend" ? -1 : 1;
        default:
          return (a, b) => a[sorter.field].localeCompare(b[sorter.field]);
      }
    });
};

const onStatusEdit = (record) => {
  isStatusEdit.value = true;
  currentRecord.value = record;
};

const onStatusSelect = (status) => {
  currentRecord.value.status = status;
  isStatusEdit.value = false;
};

const onPercentCompletionEdit = (record) => {
  isPercentCompletionEdit.value = true;
  currentRecord.value = record;
};
</script>

<template>
  <a-table
    :columns="columns"
    :row-key="(record) => record.key"
    :data-source="states"
    @change="handleTableChange"
  >
    <template #headerCell="{ column }">
      <template v-if="column.dataIndex === 'revenue'">
        <span>Revenue (total: {{ totalRevenue }})</span>
      </template>
    </template>
    <template #bodyCell="{ column, text, record }">
      <template v-if="column.dataIndex === 'status'">
        <div @click="onStatusEdit(record)" class="status-item" :class="text">
          {{ text }}
        </div>
      </template>
      <template v-if="column.dataIndex === 'percentCompletion'">
        <button class="button" @click="onPercentCompletionEdit(record)">
          <a-progress
            :width="80"
            type="circle"
            :percent="record.percentCompletion"
          />
        </button>
      </template>
      <template v-if="column.dataIndex === 'margin'">
        <a-tag :color="record.margin >= 0 ? 'blue' : 'red'">{{
          record.margin
        }}</a-tag>
      </template>
    </template>
  </a-table>
  <a-modal
    v-model:visible="isStatusEdit"
    :footer="null"
    title="Select a status"
    width="700px"
  >
    <div class="status-wrapper">
      <div
        @click="onStatusSelect(status)"
        v-for="status in statuses"
        :key="status"
        class="status-item"
        :class="status"
      >
        {{ status }}
      </div>
    </div>
  </a-modal>
  <a-modal
    v-model:visible="isPercentCompletionEdit"
    title="Change a percent of completion value"
    width="700px"
    @ok="isPercentCompletionEdit = false"
  >
    <a-slider v-model:value="currentRecord.percentCompletion" />
  </a-modal>
</template>

<style scoped>
.button {
  border: none;
  background: inherit;
  cursor: pointer;
}
.status-wrapper {
  display: flex;
}

.status-item {
  clip-path: polygon(75% 0%, 100% 48%, 75% 100%, 0% 100%, 25% 50%, 0% 0%);
  width: 130px;
  height: 60px;
  background: blue;
  font-size: 12px;
  display: flex;
  align-items: center;
  text-transform: uppercase;
  color: #fff;
  font-family: bold;
  padding-left: 38px;
  cursor: pointer;
}

.status-item.initiation {
  background: #ff595e;
}
.status-item.planning {
  background: #ffca3a;
}
.status-item.execution {
  background: #8ac926;
}
.status-item.monitoring {
  background: #1982c4;
}
.status-item.closure {
  background: #6a4c93;
}
</style>
