<template>
  <div class="container">
    <div class="calendar-center">
      <div class="calendar-container">
        <div id="calendar-heatmap">
          <div v-for="row in calendarRows" :key="row.id" class="calendar-row">
            <div v-for="dayData in row.data" :key="dayData.date" class="calendar-day"
              :style="{ backgroundColor: getHeatmapColor(dayData.contributions) }" :title="getTooltipText(dayData)"></div>
          </div>
        </div>
      </div>
      <div class="legend-container">
        <div class="legend">
          <div class="legend-label">Less</div>
          <div class="color-square" style="background-color: #EDEDED"></div>
          <div class="color-square" style="background-color: #ACD5F2"></div>
          <div class="color-square" style="background-color: #7FA8C9"></div>
          <div class="color-square" style="background-color: #527BA0"></div>
          <div class="color-square" style="background-color: #254"></div>
          <div class="legend-label">More</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  async mounted() {
    try {
      const response = await fetch('https://dpg.gg/test/calendar.json');
      const data = await response.json();
      const contributions = data;

      const today = new Date();
      const startDate = new Date(today.getTime() - 50 * 7 * 24 * 60 * 60 * 1000);
      const dayIndexOffset = startDate.getDay();

      const calendarData = [];
      for (let i = 0; i < 357; i++) {
        const currentDate = new Date(startDate.getTime() + (i - dayIndexOffset) * 24 * 60 * 60 * 1000);
        const isoDateString = currentDate.toISOString().split('T')[0];
        calendarData.push({ date: isoDateString, contributions: 0 });
      }

      for (const date in contributions) {
        if (contributions.hasOwnProperty(date)) {
          const dayData = calendarData.find((data) => data.date === date);
          if (dayData) {
            dayData.contributions = contributions[date];
          }
        }
      }

      this.calendarRows = [];
      for (let i = 0; i < 7; i++) {
        const row = { id: i, data: [] };
        for (let j = 0; j < 51; j++) {
          const dayIndex = j * 7 + i;
          row.data.push(calendarData[dayIndex]);
        }
        this.calendarRows.push(row);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  },


  methods: {
    getHeatmapColor(contributions) {
      if (contributions === 0) {
        return '#EDEDED';
      } else if (contributions >= 1 && contributions <= 9) {
        return '#ACD5F2';
      } else if (contributions >= 10 && contributions <= 19) {
        return '#7FA8C9';
      } else if (contributions >= 20 && contributions <= 29) {
        return '#527BA0';
      } else {
        return '#254';
      }
    },
    getTooltipText(dayData) {
      const contributions = dayData.contributions;
      const date = new Date(dayData.date);

      return `${contributions} contribution${contributions !== 1 ? 's' : ''}, ${date.toLocaleDateString('en-US', {
        weekday: 'long',
        month: 'long',
        day: 'numeric',
        year: 'numeric'
      })}`;
    },
  },

  data() {
    return {
      calendarRows: [],
    };
  },
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.calendar-center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.calendar-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.calendar-row {
  display: flex;
}

.calendar-day {
  width: 10px;
  height: 10px;
  margin: 1px;
}

.weekday-labels,
.month-labels {
  display: flex;
  flex-direction: column;
}

.label {
  text-align: center;
}

.color-legend {
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

.color-square {
  width: 10px;
  height: 10px;
  margin-right: 5px;
}

.legend-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 20px;
}

.legend {
  display: flex;
  align-items: center;
  justify-content: center;
}

.legend-label {
  margin-right: 10px;
  color: #959494;
  font-family: Inter;
  font-size: 12px;
  font-style: normal;
  font-weight: 400;
  line-height: normal;
}
</style>