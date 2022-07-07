<template>
  <q-page class="flex flex-center column">
    <audio src="/audio/begin.mp3" ref="beginPlayer"></audio>
    <audio src="/audio/end.mp3" ref="endPlayer"></audio>
    <h1>{{ activeTimePoint.name }}</h1>
    <q-btn @click="tip">使用前戳我一下</q-btn>
  </q-page>
</template>

<script setup>
import { onMounted, reactive, ref, watch } from "vue";

const beginPlayer = ref(null);
const endPlayer = ref(null);

//数据
const lessons = [
  {
    name: "上午第0节",
    begin: {
      hour: 7,
      minute: 0,
    },
    end: {
      hour: 7,
      minute: 30,
    },
  },
  {
    name: "上午第1节",
    begin: {
      hour: 7,
      minute: 40,
    },
    end: {
      hour: 8,
      minute: 20,
    },
  },
  {
    name: "上午第2节",
    begin: {
      hour: 8,
      minute: 30,
    },
    end: {
      hour: 9,
      minute: 10,
    },
  },
  {
    name: "上午第3节",
    begin: {
      hour: 9,
      minute: 40,
    },
    end: {
      hour: 10,
      minute: 20,
    },
  },
  {
    name: "上午第4节",
    begin: {
      hour: 10,
      minute: 30,
    },
    end: {
      hour: 11,
      minute: 10,
    },
  },
  {
    name: "上午第5节",
    begin: {
      hour: 11,
      minute: 20,
    },
    end: {
      hour: 12,
      minute: 0,
    },
  },
  {
    name: "下午第1节",
    begin: {
      hour: 14,
      minute: 30,
    },
    end: {
      hour: 15,
      minute: 10,
    },
  },
  {
    name: "下午第2节",
    begin: {
      hour: 15,
      minute: 25,
    },
    end: {
      hour: 16,
      minute: 5,
    },
  },
  {
    name: "下午第3节",
    begin: {
      hour: 16,
      minute: 15,
    },
    end: {
      hour: 16,
      minute: 55,
    },
  },
  {
    name: "下午第4节",
    begin: {
      hour: 17,
      minute: 0,
    },
    end: {
      hour: 17,
      minute: 40,
    },
  },
  {
    name: "晚自习第1节",
    begin: {
      hour: 18,
      minute: 40,
    },
    end: {
      hour: 19,
      minute: 40,
    },
  },
  {
    name: "晚自习第2节",
    begin: {
      hour: 19,
      minute: 50,
    },
    end: {
      hour: 20,
      minute: 50,
    },
  },
  {
    name: "晚自习第3节",
    begin: {
      hour: 21,
      minute: 0,
    },
    end: {
      hour: 21,
      minute: 30,
    },
  },
];

const activeTimePoint = ref({});

const lessonOn = ref(null);

const currentTimePoint = reactive({
  time: {
    hour: new Date().getHours(),
    minute: new Date().getMinutes(),
  },
});

function tip() {
  console.log("被戳了");
}

onMounted(() => {
  //初始化
  const firstTimePoint = {
    name: lessons[0].name,
    time: lessons[0].begin,
    on: true,
  };

  let point;

  lessons.forEach((value) => {
    if (!point) {
      firstTimePoint.nextPoint = {
        name: "课间",
        time: value.end,
        on: false,
      };
      point = firstTimePoint.nextPoint;
    } else {
      point.nextPoint = {
        name: value.name,
        time: value.begin,
        on: true,
        nextPoint: {
          name: "课间",
          time: value.end,
          on: false,
        },
      };
      point = point.nextPoint.nextPoint;
    }
  });

  point.last = true;
  point.nextPoint = firstTimePoint;
  point.nextPoint.lastPoint = point;
  point = point.nextPoint;
  point.first = true;

  function comparePoint(point1, point2) {
    //若point2在point1之后则返回true
    return point2.time.hour === point1.time.hour
      ? point2.time.minute >= point1.time.minute
      : point2.time.hour > point1.time.hour;
  }
  function getPosition(point) {
    if (point.last) {
      //这是最后一个时间点
      if (comparePoint(point, currentTimePoint)) {
        activeTimePoint.value = point;
      }
    } else {
      if (comparePoint(point, currentTimePoint)) {
        activeTimePoint.value = point;
        lessonOn.value = point.on;
        getPosition(point.nextPoint);
      } else if (point.first) {
        activeTimePoint.value = point.lastPoint;
        lessonOn.value = point.lastPoint.on;
      }
    }
  }
  getPosition(point);
  watch(currentTimePoint, (time) => {
    if (activeTimePoint.value["last"]) {
      if (
        comparePoint(time, activeTimePoint.value) &&
        comparePoint(activeTimePoint.value["nextPoint"], time)
      ) {
        activeTimePoint.value = activeTimePoint.value["nextPoint"];
      }
    } else if (comparePoint(activeTimePoint.value["nextPoint"], time)) {
      activeTimePoint.value = activeTimePoint.value["nextPoint"];
    }
  });
  watch(activeTimePoint, (point) => {
    if (point["on"]) {
      beginPlayer.value.play();
    } else {
      endPlayer.value.play();
    }
  });
  setInterval(() => {
    currentTimePoint.time.hour = new Date().getHours();
    currentTimePoint.time.minute = new Date().getMinutes();
  }, 1000);
});
</script>