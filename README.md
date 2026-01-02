<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简易天气查询 | 当天+未来一周</title>
    <!-- 引入Tailwind CSS简化样式 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入Font Awesome图标 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- 自定义Tailwind配置 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#3b82f6',
                        secondary: '#64748b',
                        weather: {
                            sunny: '#f97316',
                            cloudy: '#94a3b8',
                            rainy: '#60a5fa',
                            snowy: '#bfdbfe'
                        }
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
    <!-- 顶部导航 -->
    <header class="bg-primary text-white py-4 shadow-md">
        <div class="container mx-auto px-4 flex justify-between items-center">
            <h1 class="text-2xl font-bold flex items-center gap-2">
                <i class="fas fa-cloud-sun"></i>
                简易天气查询
            </h1>
            <p class="text-sm md:text-base opacity-80">实时更新 | 未来7天预报</p>
        </div>
    </header>

    <!-- 主内容区 -->
    <main class="container mx-auto px-4 py-8">
        <!-- 搜索框 -->
        <div class="max-w-2xl mx-auto mb-8">
            <div class="flex gap-2">
                <input 
                    type="text" 
                    id="cityInput" 
                    placeholder="请输入城市名称（如：北京、上海）"
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

        <!-- 天气展示区域（默认隐藏） -->
        <div id="weatherContainer" class="max-w-4xl mx-auto hidden">
            <!-- 今日天气卡片 -->
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

            <!-- 未来7天预报 -->
            <div class="bg-white rounded-xl p-6 card-shadow">
                <h3 class="text-xl font-bold text-gray-800 mb-6 flex items-center gap-2">
                    <i class="fas fa-calendar-week text-primary"></i>
                    未来7天天气预报
                </h3>
                <div id="forecastList" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 lg:grid-cols-7 gap-4">
                    <!-- 未来天数卡片会通过JS动态生成 -->
                </div>
            </div>
        </div>

        <!-- 初始提示 -->
        <div id="initialHint" class="max-w-2xl mx-auto text-center py-12">
            <div class="inline-block bg-white p-8 rounded-xl card-shadow">
                <i class="fas fa-cloud-sun-rain text-5xl text-primary mb-4"></i>
                <h3 class="text-xl font-semibold text-gray-800 mb-2">输入城市名称查询天气</h3>
                <p class="text-secondary">支持全国各城市，实时查询当天及未来7天天气</p>
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="bg-gray-800 text-white py-6 mt-auto">
        <div class="container mx-auto px-4 text-center">
            <p class="opacity-70 text-sm">
                数据来源：OpenWeatherMap API | 本网站仅作学习使用
            </p>
        </div>
    </footer>

    <script>
        // ===================== 配置项 =====================
        // 替换为你自己的OpenWeatherMap API Key（免费申请：https://openweathermap.org/api）
        const API_KEY = "5251a373187a59fde8b56b26aa83489c";
        const CURRENT_WEATHER_API = "https://api.openweathermap.org/data/2.5/weather";
        const FORECAST_API = "https://api.openweathermap.org/data/2.5/forecast";
        const UNITS = "metric"; // 单位：metric=摄氏度，imperial=华氏度
        const LANG = "zh_cn";   // 语言：中文

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
        /**
         * 格式化日期（YYYY-MM-DD 周X）
         * @param {number} timestamp - 时间戳（秒）
         * @returns {string} 格式化后的日期
         */
        function formatDate(timestamp) {
            const date = new Date(timestamp * 1000);
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            const weekDay = weekDays[date.getDay()];
            return `${year}-${month}-${day} ${weekDay}`;
        }

        /**
         * 简化日期（MM-DD 周X）
         * @param {number} timestamp - 时间戳（秒）
         * @returns {string} 简化后的日期
         */
        function formatShortDate(timestamp) {
            const date = new Date(timestamp * 1000);
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            const weekDay = weekDays[date.getDay()];
            return `${month}-${day}<br>${weekDay}`;
        }

        /**
         * 显示错误信息
         * @param {string} msg - 错误信息
         */
        function showError(msg) {
            errorMsg.textContent = msg;
            errorMsg.classList.remove('hidden');
            setTimeout(() => {
                errorMsg.classList.add('hidden');
            }, 3000);
        }

        // ===================== 核心逻辑 =====================
        /**
         * 获取并展示天气数据
         * @param {string} city - 城市名称
         */
        async function getWeatherData(city) {
            if (!city.trim()) {
                showError('请输入有效的城市名称');
                return;
            }

            try {
                // 1. 获取当前天气
                const currentRes = await fetch(`${CURRENT_WEATHER_API}?q=${encodeURIComponent(city)}&appid=${API_KEY}&units=${UNITS}&lang=${LANG}`);
                if (!currentRes.ok) {
                    if (currentRes.status === 404) {
                        throw new Error('未找到该城市的天气数据，请检查城市名称');
                    } else if (currentRes.status === 401) {
                        throw new Error('API密钥无效，请检查配置');
                    } else {
                        throw new Error('获取天气数据失败，请稍后重试');
                    }
                }
                const currentData = await currentRes.json();

                // 2. 获取未来7天预报
                const forecastRes = await fetch(`${FORECAST_API}?q=${encodeURIComponent(city)}&appid=${API_KEY}&units=${UNITS}&lang=${LANG}`);
                if (!forecastRes.ok) throw new Error('获取未来预报失败，请稍后重试');
                const forecastData = await forecastRes.json();

                // 3. 处理未来预报数据（按日期分组，取每天的最高/最低温）
                const dailyForecasts = {};
                forecastData.list.forEach(item => {
                    const date = new Date(item.dt * 1000).toLocaleDateString();
                    if (!dailyForecasts[date]) {
                        dailyForecasts[date] = {
                            temp: { min: item.main.temp_min, max: item.main.temp_max },
                            weather: item.weather[0],
                            dt: item.dt
                        };
                    } else {
                        dailyForecasts[date].temp.min = Math.min(dailyForecasts[date].temp.min, item.main.temp_min);
                        dailyForecasts[date].temp.max = Math.max(dailyForecasts[date].temp.max, item.main.temp_max);
                    }
                });

                // 4. 渲染当前天气
                renderCurrentWeather(currentData);

                // 5. 渲染未来7天预报（取前7天，排除当天）
                renderForecast(Object.values(dailyForecasts).slice(1, 8));

                // 6. 显示天气容器，隐藏初始提示
                weatherContainer.classList.remove('hidden');
                initialHint.classList.add('hidden');

            } catch (err) {
                showError(err.message);
                weatherContainer.classList.add('hidden');
                initialHint.classList.remove('hidden');
            }
        }

        /**
         * 渲染当前天气
         * @param {object} data - 当前天气数据
         */
        function renderCurrentWeather(data) {
            todayCity.textContent = `${data.name} (${data.sys.country})`;
            todayDate.textContent = formatDate(data.dt);
            todayIcon.src = `https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`;
            todayIcon.alt = data.weather[0].description;
            todayWeather.textContent = data.weather[0].description;
            todayTemp.textContent = `${Math.round(data.main.temp)}°C`;
            feelsLike.textContent = `${Math.round(data.main.feels_like)}°C`;
            humidity.textContent = `${data.main.humidity}%`;
            windSpeed.textContent = `${(data.wind.speed * 3.6).toFixed(1)} km/h`;
            pressure.textContent = `${data.main.pressure} hPa`;
        }

        /**
         * 渲染未来7天预报
         * @param {array} forecasts - 未来预报数据数组
         */
        function renderForecast(forecasts) {
            forecastList.innerHTML = '';
            forecasts.forEach(forecast => {
                const forecastCard = document.createElement('div');
                forecastCard.className = 'bg-gray-50 rounded-lg p-4 text-center transition-all-300 hover:shadow-md hover:translate-y-[-2px]';
                forecastCard.innerHTML = `
                    <p class="text-secondary text-sm mb-2">${formatShortDate(forecast.dt)}</p>
                    <img src="https://openweathermap.org/img/wn/${forecast.weather.icon}@2x.png" alt="${forecast.weather.description}" class="w-12 h-12 mx-auto mb-2">
                    <p class="text-sm text-gray-700 mb-1">${forecast.weather.description}</p>
                    <div class="flex justify-center gap-2 text-sm">
                        <span class="text-red-500">↑ ${Math.round(forecast.temp.max)}°C</span>
                        <span class="text-blue-500">↓ ${Math.round(forecast.temp.min)}°C</span>
                    </div>
                `;
                forecastList.appendChild(forecastCard);
            });
        }

        // ===================== 事件监听 =====================
        // 搜索按钮点击事件
        searchBtn.addEventListener('click', () => {
            getWeatherData(cityInput.value);
        });

        // 回车触发搜索
        cityInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                getWeatherData(cityInput.value);
            }
        });

        // 页面加载时聚焦搜索框
        window.addEventListener('load', () => {
            cityInput.focus();
        });
    </script>
</body>
</html>
