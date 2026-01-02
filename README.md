<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简易天气查询 | 当天+未来一周</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#3b82f6',
                        secondary: '#64748b',
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .card-shadow {
                box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
            }
            .transition-all-300 {
                transition: all 0.3s ease;
            }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    <header class="bg-primary text-white py-4 shadow-md">
        <div class="container mx-auto px-4 flex justify-between items-center">
            <h1 class="text-2xl font-bold flex items-center gap-2">
                <i class="fas fa-cloud-sun"></i>
                简易天气查询
            </h1>
            <p class="text-sm md:text-base opacity-80">实时更新 | 未来7天预报</p>
        </div>
    </header>

    <main class="container mx-auto px-4 py-8">
        <div class="max-w-2xl mx-auto mb-8">
            <div class="flex gap-2">
                <input 
                    type="text" 
                    id="cityInput" 
                    placeholder="请输入城市名称（如：北京、上海、杭州）"
                    class="flex-1 px-4 py-3 rounded-lg border border-gray-200 card-shadow focus:outline-none focus:ring-2 focus:ring-primary/50 transition-all-300"
                >
                <button 
                    id="searchBtn"
                    class="bg-primary text-white px-6 py-3 rounded-lg card-shadow hover:bg-primary/90 active:scale-95 transition-all-300"
                >
                    <i class="fas fa-search mr-1"></i> 查询
                </button>
            </div>
            <p id="errorMsg" class="text-red-500 text-sm mt-2 hidden"></p>
        </div>

        <div id="weatherContainer" class="max-w-4xl mx-auto hidden">
            <div class="bg-white rounded-xl p-6 card-shadow mb-8">
                <div class="flex flex-col md:flex-row justify-between items-center gap-6">
                    <div>
                        <h2 id="todayCity" class="text-3xl font-bold text-gray-800 mb-2"></h2>
                        <p id="todayDate" class="text-secondary mb-4"></p>
                        <div class="flex items-center gap-4">
                            <img id="todayIcon" src="" alt="天气图标" class="w-16 h-16">
                            <div>
                                <p id="todayWeather" class="text-xl text-gray-700 mb-1"></p>
                                <p id="todayTemp" class="text-4xl font-bold text-primary"></p>
                            </div>
                        </div>
                    </div>
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 w-full md:w-auto">
                        <div class="bg-gray-50 rounded-lg p-4 text-center">
                            <p class="text-secondary text-sm mb-1"><i class="fas fa-temperature-low mr-1"></i> 体感温度</p>
                            <p id="feelsLike" class="text-xl font-semibold"></p>
                        </div>
                        <div class="bg-gray-50 rounded-lg p-4 text-center">
                            <p class="text-secondary text-sm mb-1"><i class="fas fa-tint mr-1"></i> 湿度</p>
                            <p id="humidity" class="text-xl font-semibold"></p>
                        </div>
                        <div class="bg-gray-50 rounded-lg p-4 text-center">
                            <p class="text-secondary text-sm mb-1"><i class="fas fa-wind mr-1"></i> 风速</p>
                            <p id="windSpeed" class="text-xl font-semibold"></p>
                        </div>
                        <div class="bg-gray-50 rounded-lg p-4 text-center">
                            <p class="text-secondary text-sm mb-1"><i class="fas fa-globe mr-1"></i> 气压</p>
                            <p id="pressure" class="text-xl font-semibold"></p>
                        </div>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl p-6 card-shadow">
                <h3 class="text-xl font-bold text-gray-800 mb-6 flex items-center gap-2">
                    <i class="fas fa-calendar-week text-primary"></i>
                    未来7天天气预报
                </h3>
                <div id="forecastList" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 lg:grid-cols-7 gap-4">
                </div>
            </div>
        </div>

        <div id="initialHint" class="max-w-2xl mx-auto text-center py-12">
            <div class="inline-block bg-white p-8 rounded-xl card-shadow">
                <i class="fas fa-cloud-sun-rain text-5xl text-primary mb-4"></i>
                <h3 class="text-xl font-semibold text-gray-800 mb-2">输入城市名称查询天气</h3>
                <p class="text-secondary">支持全国各城市，实时查询当天及未来7天天气</p>
            </div>
        </div>
    </main>

    <footer class="bg-gray-800 text-white py-6 mt-auto">
        <div class="container mx-auto px-4 text-center">
            <p class="opacity-70 text-sm">
                数据来源：高德地图API | 本网站仅作学习使用
            </p>
        </div>
    </footer>

    <script>
        // ===================== 高德API配置（替换为你的密钥） =====================
        const AMAP_KEY = "84b0ed4ededbaac14e90d1f1feac6ae8"; // 替换为申请的高德Key
        const AMAP_GEO_API = "https://restapi.amap.com/v3/geocode/geo"; // 地理编码（城市转经纬度）
        const AMAP_WEATHER_API = "https://restapi.amap.com/v3/weather/weatherInfo"; // 天气API

        // ===================== 元素获取 =====================
        const cityInput = document.getElementById('cityInput');
        const searchBtn = document.getElementById('searchBtn');
        const errorMsg = document.getElementById('errorMsg');
        const weatherContainer = document.getElementById('weatherContainer');
        const initialHint = document.getElementById('initialHint');
        const todayCity = document.getElementById('todayCity');
        const todayDate = document.getElementById('todayDate');
        const todayIcon = document.getElementById('todayIcon');
        const todayWeather = document.getElementById('todayWeather');
        const todayTemp = document.getElementById('todayTemp');
        const feelsLike = document.getElementById('feelsLike');
        const humidity = document.getElementById('humidity');
        const windSpeed = document.getElementById('windSpeed');
        const pressure = document.getElementById('pressure');
        const forecastList = document.getElementById('forecastList');

        // ===================== 工具函数 =====================
        // 格式化日期（YYYY-MM-DD 周X）
        function formatDate(dateStr) {
            const date = new Date(dateStr);
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            const weekDay = weekDays[date.getDay()];
            return `${year}-${month}-${day} ${weekDay}`;
        }

        // 简化日期（MM-DD 周X）
        function formatShortDate(dateStr) {
            const date = new Date(dateStr);
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            const weekDay = weekDays[date.getDay()];
            return `${month}-${day}<br>${weekDay}`;
        }

        // 显示错误信息
        function showError(msg) {
            errorMsg.textContent = msg;
            errorMsg.classList.remove('hidden');
            setTimeout(() => errorMsg.classList.add('hidden'), 3000);
        }

        // 天气图标映射（高德天气类型→图标）
        const weatherIconMap = {
            '晴': 'https://webapi.amap.com/images/weather/icon/weather/day/01.png',
            '多云': 'https://webapi.amap.com/images/weather/icon/weather/day/02.png',
            '阴': 'https://webapi.amap.com/images/weather/icon/weather/day/03.png',
            '小雨': 'https://webapi.amap.com/images/weather/icon/weather/day/07.png',
            '中雨': 'https://webapi.amap.com/images/weather/icon/weather/day/08.png',
            '大雨': 'https://webapi.amap.com/images/weather/icon/weather/day/09.png',
            '暴雨': 'https://webapi.amap.com/images/weather/icon/weather/day/10.png',
            '雷阵雨': 'https://webapi.amap.com/images/weather/icon/weather/day/13.png',
            '小雪': 'https://webapi.amap.com/images/weather/icon/weather/day/15.png',
            '中雪': 'https://webapi.amap.com/images/weather/icon/weather/day/16.png',
            '大雪': 'https://webapi.amap.com/images/weather/icon/weather/day/17.png',
            '雾': 'https://webapi.amap.com/images/weather/icon/weather/day/20.png',
            '霾': 'https://webapi.amap.com/images/weather/icon/weather/day/21.png'
        };

        // ===================== 核心逻辑 =====================
        // 1. 城市名称转经纬度（高德天气API需要adcode）
        async function getCityAdcode(city) {
            const res = await fetch(`${AMAP_GEO_API}?address=${encodeURIComponent(city)}&key=${AMAP_KEY}`);
            const data = await res.json();
            if (data.status !== '1' || !data.geocodes.length) {
                throw new Error(`未找到「${city}」的地理信息，请检查城市名称`);
            }
            // 获取城市的adcode（行政区划代码）
            return {
                adcode: data.geocodes[0].adcode,
                cityName: data.geocodes[0].city || data.geocodes[0].province // 兼容直辖市
            };
        }

        // 2. 根据adcode获取天气数据
        async function getWeatherByAdcode(adcode) {
            // 获取实时+7天预报
            const res = await fetch(`${AMAP_WEATHER_API}?city=${adcode}&extensions=all&key=${AMAP_KEY}`);
            const data = await res.json();
            if (data.status !== '1') {
                throw new Error(`获取天气失败：${data.info || '未知错误'}`);
            }
            return {
                now: data.lives[0], // 实时天气
                forecast: data.forecasts[0].casts // 7天预报
            };
        }

        // 3. 主查询函数
        async function getWeatherData(city) {
            if (!city.trim()) {
                showError('请输入有效的城市名称');
                return;
            }

            try {
                // 步骤1：城市转adcode
                const { adcode, cityName } = await getCityAdcode(city);
                // 步骤2：获取天气数据
                const { now, forecast } = await getWeatherByAdcode(adcode);
                
                // 步骤3：渲染今日天气
                renderTodayWeather(now, cityName);
                // 步骤4：渲染7天预报
                render7dForecast(forecast);
                
                // 显示天气容器
                weatherContainer.classList.remove('hidden');
                initialHint.classList.add('hidden');

            } catch (err) {
                showError(err.message);
                weatherContainer.classList.add('hidden');
                initialHint.classList.remove('hidden');
            }
        }

        // 渲染今日天气
        function renderTodayWeather(now, cityName) {
            todayCity.textContent = `${cityName} (中国)`;
            todayDate.textContent = formatDate(now.reporttime);
            // 匹配天气图标
            todayIcon.src = weatherIconMap[now.weather] || weatherIconMap['晴'];
            todayIcon.alt = now.weather;
            todayWeather.textContent = now.weather;
            todayTemp.textContent = `${now.temperature}°C`;
            feelsLike.textContent = `${now.temperature}°C`; // 高德实时天气无体感温度，用当前温度替代
            humidity.textContent = `${now.humidity}%`;
            windSpeed.textContent = `${now.windpower}级 (${now.winddirection})`;
            pressure.textContent = `${now.pressure} hPa`;
        }

        // 渲染7天预报
        function render7dForecast(forecast) {
            forecastList.innerHTML = '';
            // 取前7天预报
            forecast.slice(0,7).forEach(day => {
                const card = document.createElement('div');
                card.className = 'bg-gray-50 rounded-lg p-4 text-center transition-all-300 hover:shadow-md hover:translate-y-[-2px]';
                card.innerHTML = `
                    <p class="text-secondary text-sm mb-2">${formatShortDate(day.date)}</p>
                    <img src="${weatherIconMap[day.dayweather] || weatherIconMap['晴']}" alt="${day.dayweather}" class="w-12 h-12 mx-auto mb-2">
                    <p class="text-sm text-gray-700 mb-1">${day.dayweather}</p>
                    <div class="flex justify-center gap-2 text-sm">
                        <span class="text-red-500">↑ ${day.highest}°C</span>
                        <span class="text-blue-500">↓ ${day.lowest}°C</span>
                    </div>
                `;
                forecastList.appendChild(card);
            });
        }

        // ===================== 事件监听 =====================
        searchBtn.addEventListener('click', () => getWeatherData(cityInput.value));
        cityInput.addEventListener('keydown', (e) => e.key === 'Enter' && getWeatherData(cityInput.value));
        window.addEventListener('load', () => cityInput.focus());
    </script>
</body>
</html>
