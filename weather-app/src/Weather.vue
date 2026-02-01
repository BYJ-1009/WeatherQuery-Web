<template>
    <div class="weather-check">
        <h1 id="Title">实时天气查询</h1>
        <el-input v-model="city" class="responsive-input" style="width: 220px;margin-right: 2px;" placeholder="请输入城市名称"
            :prefix-icon="Search" clearable @keyup.enter="getWeather" />
        <el-button @click="getWeather" type="primary" :icon="Search">搜索</el-button>

        <transition>
            <div v-if="iserrorshow">
                <el-alert :title="error" type="error" center show-icon effect="dark"
                    style="width:310px;position: relative;left: 50%;transform: translate(-50%);margin-top: 10px;" />
            </div>
        </transition>
        <transition>
            <div class="Card" v-if="weather">
                <el-card id="card" style="max-width: 500px" shadow="hover">
                    <template #header>
                        <div class="card-header">
                            <span id="title"><span id="city">{{ weather.lives[0].city }}</span>的天气</span>
                        </div>
                    </template>
                    <div class="weather_box">
                        <img id="myimg" :src="imgurl" alt="图片获取失败">
                        <div>
                            <p id="weather_condition">{{ weather.lives[0].weather }}</p>
                            <p id="temperature">{{ weather.lives[0].temperature }}℃</p>
                        </div>
                        <div>
                            <img id="wet" src="./assets/weather_icon/湿度.png" alt="图片获取失败">
                            <img id="wind" src="./assets/weather_icon/风向.png" alt="图片获取失败">
                        </div>
                        <p id="airwet">空气湿度:{{ weather.lives[0].humidity }}</p>
                        <p id="airwind">风向:{{ weather.lives[0].winddirection }}<br>风力:{{ weather.lives[0].windpower }}级
                        </p>
                    </div>
                </el-card>
            </div>
        </transition>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { Search } from '@element-plus/icons-vue';
const weatherIconMap = import.meta.glob('./assets/weather_icon/*.png', {
    eager: true,
    as: 'url'
});
const city = ref('');
const weather = ref(null);
const error = ref('');
const iserrorshow = ref(false);
let errortimer = null;
const imgurl = ref(``);
const getWeather = () => {
    if (city.value.trim() === '') {
        iserrorshow.value = true;
        error.value = "请输入城市名称";
        weather.value = null;
        if (errortimer) {
            clearTimeout(errortimer);
        }
        errortimer = setTimeout(() => {
            iserrorshow.value = false;
            errortimer = null;
        }, 2000)
        return;
    }
    error.value = '';
    const apiKey = "05a97511c99bfffcf6eee51d1b0f9164";
    axios.get(`https://restapi.amap.com/v3/weather/weatherInfo?key=${apiKey}&city=${city.value}`)
        .then(response => {
            if (response.data.lives.length > 0 && Array.isArray(response.data.lives[0]) === false) {
                if (weather.value === null) {
                    weather.value = response.data;
                    const iconKey = `./assets/weather_icon/${weather.value.lives[0].weather}.png`;
                    imgurl.value = weatherIconMap[iconKey];
                } else {
                    setTimeout(() => {
                        weather.value = response.data;
                        const iconKey = `./assets/weather_icon/${weather.value.lives[0].weather}.png`;
                        imgurl.value = weatherIconMap[iconKey];
                    }, 800);
                    weather.value=null;
                }
            } else {
                iserrorshow.value = true;
                error.value = "无法获取天气数据,请检查城市名称";
                weather.value = null;
                if (errortimer) {
                    clearTimeout(errortimer);
                }
                errortimer = setTimeout(() => {
                    iserrorshow.value = false;
                    errortimer = null;
                }, 2000)
            }
        })
        .catch(err => {
            iserrorshow.value = true;
            error.value = "网络异常,请稍后重试";
            weather.value = null;
            if (errortimer) {
                clearTimeout(errortimer);
            }
            errortimer = setTimeout(() => {
                iserrorshow.value = false;
                errortimer = null;
            }, 2000)
        })
}
</script>

<style scoped>
.v-enter-active,
.v-leave-active {
    transition: opacity 1s ease;
}

.v-enter-from,
.v-leave-to {
    opacity: 0;
}
</style>