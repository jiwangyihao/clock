<template>
  <q-page class="flex flex-center column">
    <audio src="/audio/begin.mp3" ref="beginPlayer"></audio>
    <audio src="/audio/end.mp3" ref="endPlayer"></audio>
    <h1>{{ activeTimePoint.name }}</h1>
    <h4>
      {{ timeDistance }}后{{ activeTimePoint.nextPoint.on ? "上课" : "下课" }}
    </h4>
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

const activeTimePoint = ref({
  nextPoint: {
    time: {},
  },
  last: false,
});

const lessonOn = ref(null);

const currentTimePoint = reactive({
  time: {
    hour: new Date().getHours(),
    minute: new Date().getMinutes(),
  },
});

const timeDistance = ref("一段时间");

function calculateTimeDistance() {
  const dateBegin = new Date();
  const dateEnd = new Date();
  dateEnd.setHours(activeTimePoint.value.nextPoint.time.hour);
  dateEnd.setMinutes(activeTimePoint.value.nextPoint.time.minute);
  dateEnd.setSeconds(0);
  if (activeTimePoint.value.last) {
    dateEnd.setDate(dateEnd.getDate() + 1);
  }
  const dateDiff = dateEnd.getTime() - dateBegin.getTime(); //时间差的毫秒数
  const leave1 = dateDiff % (24 * 3600 * 1000); //计算天数后剩余的毫秒数
  const hours = Math.floor(leave1 / (3600 * 1000)); //计算出小时数
  //计算相差分钟数
  const leave2 = leave1 % (3600 * 1000); //计算小时数后剩余的毫秒数
  const minutes = Math.floor(leave2 / (60 * 1000)); //计算相差分钟数
  //计算相差秒数
  const leave3 = leave2 % (60 * 1000); //计算分钟数后剩余的毫秒数
  const seconds = Math.round(leave3 / 1000);
  let timesString = "";

  if (hours !== 0) {
    timesString = `${hours}小时${minutes}分钟${seconds}秒`;
  } else if (hours === 0 && minutes !== 0) {
    timesString = `${minutes}分钟${seconds}秒`;
  } else if (hours === 0 && minutes === 0) {
    timesString = `${seconds}秒`;
  }

  return timesString;
}

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
    timeDistance.value = calculateTimeDistance();
  }, 1000);
});
</script>
