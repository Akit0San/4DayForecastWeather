<template>
  <v-layout class="rounded rounded-md main ">
    <v-app-bar title="WeatherFture"></v-app-bar>
    <v-main class="d-flex align-center  main backgroundBlue " style="min-height: 300px;">
      <div class="whether_input">
        <v-autocomplete v-model="searchInput" :items="cities.map(city => city.name)" label="City"
          @auto-select-first="false" :loading="loading">
          <template v-slot:append>
            <v-btn size="large" class="ma-2 " color="primary" @click="fetchWeatherData">Search</v-btn>
          </template>
        </v-autocomplete>
      </div>
      <v-card>
        <v-tabs mandatory class="tabs" v-model="tab" bg-color="red-lighten-2">
          <v-tab class="tab" v-for="item in weatherList" :key="item.weatherTodayData.adcode"
            :value="item.weatherTodayData.city">
            <div> {{ item.weatherTodayData.city }}</div>
            <div> {{ item.weatherTodayData.temperature }}</div>
            <div> {{ item.weatherTodayData.weather }}</div>
            <div> <img @click="deleteTab(item.weatherTodayData.adcode)" src="../assets/delete.svg" alt=""></div>
          </v-tab>
        </v-tabs>
        <v-window v-model="tab">
          <v-window-item v-for="area in weatherList" :key="area.weatherTodayData.adcode"
            :value="area.weatherTodayData.city">
            <v-card color="basil" flat>
              <v-card-text>
                <v-expand-transition>
                  <div class="weather_location" v-show="expand">
                    <v-breadcrumbs :items="[`${area.weatherTodayData.province}`, ``]">
                      {{ area.weatherTodayData.city }} </v-breadcrumbs>
                    <v-divider></v-divider>
                  </div>
                </v-expand-transition>
                <v-expand-transition>
                  <div class="wehther_Item" v-show="expand">
                    <v-card class="wehther_Itemcard" title="当前天气">
                      <v-card-subtitle>时间:{{ area.weatherTodayData.reporttime.slice(10, 16) }}</v-card-subtitle>
                      <v-card-text class="temperatureText">
                        <img :src="weatherTUrl" alt="">
                        <div class="temperature">{{ area.weatherTodayData.temperature }}<span>°C</span></div>
                        <div class="weather">{{ area.weatherTodayData.weather }}</div>
                      </v-card-text>
                      <v-card-text class="anotherInfo">
                        <div class="humidity">湿度：{{ area.weatherTodayData.humidity }}</div>
                        <div class="windpower">风力：{{ area.weatherTodayData.windpower }}</div>
                        <div class="winddirection">风向：{{ area.weatherTodayData.winddirection }}</div>
                      </v-card-text>
                    </v-card>
                    <v-card class="wehther_Itemcard">
                      <v-parallax src="https://cdn.vuetifyjs.com/images/backgrounds/vbanner.jpg">
                        <div class="d-flex flex-column fill-height justify-center align-center text-white">
                          <h1 class="text-h4 font-weight-thin mb-4">
                            Vuetify
                          </h1>
                          <h4 class="subheading">
                            Build your application today!
                          </h4>
                        </div>
                      </v-parallax>
                    </v-card>
                  </div>
                </v-expand-transition>
                <v-expand-transition>
                  <div class="Fruther">
                    <div class=Ftitle v-show="expand">预报</div>
                    <v-row align="center" justify="center" v-show="expand">
                      <v-col v-for="item in area.weatherFutureData.casts" :key="item.week" cols="auto">
                        <v-card class="mx-auto Fcard" max-width="297" :color="color">
                          <v-card-item class="FtherInfo">
                            <div v-if="area.weatherFutureData.reporttime.slice(5, 10) == item.date.slice(5, 10)"
                              class="date">
                              今天
                            </div>
                            <div v-else>
                              {{ item.date.slice(8, 10) }}日 周{{ weekdaysMap[item.week] }}
                            </div>
                            <div class="moreInfo">
                              <div class="FtureImg"> <img :src="item.url" alt=""></div>
                              <div class="info">
                                <div class="daytemp">
                                  <div class="daytemp_t1">{{ item.daytemp }}<span>°C</span></div>
                                  <div class="daytemp_t2">{{ item.dayweather }}</div>
                                </div>
                                <div class="nighttemp">
                                  <div>{{ item.nighttemp }}<span>°C</span></div>
                                  <div class="daytemp_t2">{{ item.nightweather }}</div>
                                </div>
                              </div>
                            </div>
                            <v-expand-transition>
                              <div v-if="expands" class="expands">
                                <div>白天风力：{{ item.daypower }}级 <span> </span></div>
                                <div>夜晚风力：{{ item.daypower }}级</div>
                                <div></div>
                              </div>
                            </v-expand-transition>
                            <v-divider></v-divider>
                            <v-card-actions>
                              <v-btn @click="expands = !expands">
                                {{ !expands ? 'Full Report' : 'Hide Report' }}
                              </v-btn>
                            </v-card-actions>
                          </v-card-item>
                        </v-card>
                      </v-col>
                    </v-row>
                  </div>
                </v-expand-transition>
              </v-card-text>
            </v-card>
          </v-window-item>
        </v-window>
      </v-card>
    </v-main>
    <v-footer class="bg-grey-lighten-1 footer">
      <v-row justify="center" no-gutters>
        <v-col class="text-center mt-4 footerDiv " cols="12">
          <div> <strong>{{ new Date().getFullYear() }}</strong></div>
          <div> <strong>Powerby—Akito</strong></div>
        </v-col>
      </v-row>
    </v-footer>
  </v-layout>
</template>

<script setup lang="ts">
import { reactive, ref, watch } from "vue"
import axios from 'axios'

interface IconUrls {
  [key: string]: string
}

const color: string = 'indigo'
const apikey: string = 'de4430b0b91f9248ee94f806b972dce9'
const weekdaysMap: IconUrls = {
  '1': '一',
  '2': '二',
  '3': '三',
  '4': '四',
  '5': '五',
  '6': '六',
  '7': '天',
};
const cities = ref([
  { name: '成都', code: '510100' },
  { name: '重庆', code: '500000' },
  { name: '北京', code: '110000' },
  { name: '上海', code: '310000' },
  { name: '广州', code: '440100' },
  { name: '天津', code: '120000' },
  { name: '深圳', code: '440300' },
  { name: '上海', code: '310000' },
  { name: '深圳', code: '440300' },
  { name: '石家庄', code: '130100' },
  { name: '太原', code: '140100' },
  { name: '呼和浩特', code: '150100' },
  { name: '沈阳', code: '210100' },
  { name: '哈尔滨', code: '230100' },
  { name: '南京', code: '320100' },
  { name: '杭州', code: '330100' },
  { name: '合肥', code: '340100' },
  { name: '南昌', code: '340100' },
  { name: '武汉', code: '420100' },
  { name: '长沙', code: '430100' },
  { name: '贵阳', code: '520100' },
  { name: '昆明', code: '530100' },
  { name: '西安', code: '610100' },
  { name: '台湾', code: '710000' },
  { name: '香港', code: '810000' },
])
const length = ref<number>(0)
const tab = ref<number | null>(null)
const loading = ref(false);
const expand = ref(false);
const expands = ref(false);
const searchInput = ref('');
const weatherTUrl = ref('');
let weatherTodayData = reactive({
  adcode: '',
  city: '',
  humidity: '',
  province: '',
  reporttime: '',
  temperature: '',
  weather: '',
  winddirection: '',
  windpower: '',
})
let weatherFutureData = reactive({
  adcode: '',
  casts: [{
    date: '',
    daypower: '',
    daytemp: '',
    daytemp_float: '',
    dayweather: '',
    daywind: '',
    nightpower: '',
    nighttemp: '',
    nighttemp_float: '',
    nightweather: '',
    nightwind: '',
    week: '',
    url: '',
  }],
  city: '',
  province: '',
  reporttime: ''
})
type WeatherData = {
  weatherTodayData: typeof weatherTodayData;
  weatherFutureData: typeof weatherFutureData;
}

const weatherList = reactive<WeatherData[]>([]);

// 查询天气信息 添加新的标签卡
const fetchWeatherData = () => {
  const selectedCity = cities.value.find(city => city.name == searchInput.value);
  if (selectedCity) {
    expand.value = false; // 隐藏天气信息
    loading.value = true; // 显示加载动画
    const cityCode = selectedCity.code;
    fetchWeatherTData(cityCode);
    fetchWeatherFData(cityCode);
    setTimeout(() => {
      // 判断逻辑  如果weatherList.weatherTodayData里以及存在了这个新的天气信息 就覆盖 而不是push
      // 如果不存在 才push      weatherList.push({ weatherTodayData, weatherFutureData })
      let cityIndex = weatherList.findIndex(item => item.weatherTodayData.adcode == selectedCity.code); // city.name不行？？？？？？？
      if (cityIndex >= 0) {
        // 如果城市已经存在，就更新数据  
        weatherList[cityIndex] = { weatherTodayData, weatherFutureData };
      } else {
        // 如果城市不存在，就添加新的数据  
        weatherList.push({ weatherTodayData, weatherFutureData });
        length.value += 1;
      }
    }, 1500);
    setTimeout(() => {
      expand.value = true; //显示天气信息
      loading.value = false; // 隐藏加载动画
    }, 1600);
  } else {
    searchInput.value = '';
  }
};
// 删除标签卡
const deleteTab = (adcode: string) => {
  const cityCode = adcode;
  console.log(adcode);
  // 查找选项卡数据在数组中的索引
  const tabIndex = weatherList.findIndex((item) => item.weatherTodayData.adcode === cityCode);
  if (tabIndex !== -1) {
    // 找到选项卡数据并从数组中删除
    weatherList.splice(tabIndex, 1);
    length.value -= 1;
  }
}
// 天气图片获取对象
function getWeatherIconUrl(weather: string, isDaytime: boolean): string {
  const daytimeIconUrls: IconUrls = {
    '晴': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/SunnyDayV3.svg',
    '多云': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/MostlyCloudyDayV2.svg',
    '阴': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/CloudyV3.svg',
    '小雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/D210LightRainShowersV2.svg',
    '雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/HeavyDrizzle.svg',
    '阵雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/LightRainShowerDay.svg',
    '中雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/ModerateRainV2.svg',
    '多雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/ModerateRainV2.svg',
    '大雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/ModerateRainV2.svg',
  }

  const nighttimeIconUrls: IconUrls = {
    '晴': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/ClearNightV3.svg',
    '多云': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/MostlyCloudyNightV2.svg',
    '阴': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/CloudyV3.svg',
    '小雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/N310LightRainShowersV2.svg',
    '阵雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/N310LightRainShowersV2.svg',
    '大雨': 'https://assets.msn.cn/weathermapdata/1/static/weather/Icons/taskbar_v10/Condition_Card/ModerateRainV2.svg',
  }

  const iconUrls = isDaytime ? daytimeIconUrls : nighttimeIconUrls
  return iconUrls[weather] || ''
}

// Axios获取天气数据
const fetchWeatherTData = (cityCode: string) => {
  axios.get('https://restapi.amap.com/v3/weather/weatherInfo', {
    params: {
      key: apikey,
      city: cityCode, // 使用传递的城市代码
      extensions: 'base'
    }
  })
    .then((response) => {
      weatherTodayData = response.data.lives[0];
      console.log(weatherTodayData);
      // reporttime为 "2023-10-16 08:08:45"
      const isDaytime = Number(weatherTodayData.reporttime.slice(11, 13)) >= 7 && Number(weatherTodayData.reporttime.slice(11, 13)) <= 20;
      const weatherIconUrl = getWeatherIconUrl(weatherTodayData.weather, isDaytime)
      weatherTUrl.value = weatherIconUrl;
    })
    .catch((error) => {
      console.error('获取今日天气数据时出错：', error)
    })
}

const fetchWeatherFData = (cityCode: string) => {
  axios.get('https://restapi.amap.com/v3/weather/weatherInfo', {
    params: {
      key: apikey,
      city: cityCode,
      extensions: 'all'
    }
  })
    .then((response) => {
      weatherFutureData = response.data.forecasts[0]
      console.log(weatherFutureData);
      const isDaytime = Number(weatherTodayData.reporttime.slice(11, 13)) >= 7 && Number(weatherTodayData.reporttime.slice(11, 13)) <= 20;
      weatherFutureData.casts.forEach((item) => {
        const weatherIconUrl = getWeatherIconUrl(item.dayweather, isDaytime)
        item.url = weatherIconUrl;
      })

    })
    .catch((error) => {
      console.error('获取预报天气数据时出错：', error)
    })
}

// 监听标签卡的数量
watch(length, val => {
  tab.value = val - 1;
})
</script>
<style scoped>
.main {
  display: flex;
  flex-direction: column;
}

.whether_input {
  margin-top: 20px;
  width: 600px;
}

.weather_location {
  width: 1300px;
  padding-left: 20px;
  font-size: 25px;
}

.wehther_Item {
  display: flex;
  flex-direction: row;
}

.wehther_Itemcard {
  margin: 20px;
  width: 610px;
  height: 250px;
}

.footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 20px;
}

.v-breadcrumbs-item {
  opacity: unset;
}

.temperatureText {
  display: flex;

  margin-top: -10px;
}

.temperatureText>img {
  width: 72px;
  height: 72px;
}

.temperature {
  font-size: 64px;
  margin-top: 33px;
  margin-left: 10px;
}

.weather {
  font-size: 18px;
  margin-left: 20px;
  margin-top: 20px;
}

.anotherInfo {
  display: flex;
}


.humidity,
.windpower,
.winddirection {
  margin-left: 20px;
  font-size: 18px
}

.humidity {
  margin-left: 5px
}

.Fruther {
  width: 1280px;
}

.FtherInfo {
  display: flex;
  font-size: 18px;
}

.FtureImg {
  width: 60px;
  height: 60px;
}

.FtureImg>img {
  width: 60px;
  height: 60px;
}

.moreInfo {
  margin-top: 20px;
  display: flex;
}

.daytemp {
  display: flex;
  margin-top: 5px;
  width: 180px;
}

.nighttemp {
  display: flex;
  margin-top: 12px;
  width: 180px;
}

.info {
  margin-left: 10px;
  position: relative;
}

.daytemp_t2 {
  position: absolute;
  left: 143px;
  width: 143px;
}

.Fcard {
  width: 320px;
}

.loading {
  margin: 150px;
  margin-top: 300px;
  transform: translate(-50%, -50%);
}

.footerDiv {
  display: flex;
  justify-content: space-between;
}

.Ftitle {
  font-size: 22px;
  margin-left: 15px;
  margin-bottom: 20px;
}

.expands {
  margin-top: 7px;
  font-size: 15px;
}

.tabs {
  width: 1332px;
  height: 60px;
  font-size: 20px;
}

.tab {
  display: flex;
  font-size: 18px;
  height: auto !important;
  ;
}

.tab div:nth-child(2) {
  margin-top: 3px;
  margin-left: 10px;
}

.tab div:nth-child(4) {
  color: wheat;
  margin-top: 7px;
  margin-left: 10px;
}
</style>