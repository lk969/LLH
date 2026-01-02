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
                    placeholder="请输入城市名称（如：北京、上海、杭州市西湖区）"
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
                <p class="text-secondary">支持全国各城市/区县，实时查询当天及未来7天天气</p>
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="bg-gray-800 text-white py-6 mt-auto">
        <div class="container mx-auto px-4 text-center">
            <p class="opacity-70 text-sm">
                数据来源：和风天气 API | 本网站仅作学习使用
            </p>
        </div>
    </footer>

    <script>
        // ===================== 和风天气配置项（替换为你的密钥） =====================
        const API_KEY = "001d2426ced54715911c1c693e370e87"; // 替换为你申请的和风天气开发版KEY
        const CURRENT_WEATHER_API = "https://devapi.qweather.com/v7/weather/now"; // 实时天气接口
        const FORECAST_API = "https://devapi.qweather.com/v7/weather/7d"; // 7天预报接口

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

        /**
         * 格式化今日日期（YYYY-MM-DD 周X）
         */
        function formatTodayDate() {
            const today = new Date();
            const year = today.getFullYear();
            const month = String(today.getMonth() + 1).padStart(2, '0');
            const day = String(today.getDate()).padStart(2, '0');
            const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            const weekDay = weekDays[today.getDay()];
            return `${year}-${month}-${day} ${weekDay}`;
        }

        // ===================== 核心逻辑 =====================
        /**
         * 获取并展示天气数据（适配和风天气API）
         * @param {string} city - 城市名称（中文）
         */
        async function getWeatherData(city) {
            if (!city.trim()) {
                showError('请输入有效的城市名称');
                return;
            }

            try {
                // 1. 获取实时天气
                const currentRes = await fetch(`${CURRENT_WEATHER_API}?location=${encodeURIComponent(city)}&key=${API_KEY}`);
                if (!currentRes.ok) throw new Error('网络请求失败，请检查网络连接');
                
                const currentData = await currentRes.json();
                // 检查和风天气接口返回状态码
                if (currentData.code !== "200") {
                    handleQWeatherError(currentData.code, city);
                    return;
                }

                // 2. 获取未来7天预报
                const forecastRes = await fetch(`${FORECAST_API}?location=${encodeURIComponent(city)}&key=${API_KEY}`);
                if (!forecastRes.ok) throw new Error('获取未来预报失败，请稍后重试');
                
                const forecastData = await forecastRes.json();
                if (forecastData.code !== "200") {
                    throw new Error(`获取未来预报失败：${forecastData.msg || '未知错误'}`);
                }

                // 3. 渲染当前天气
                renderCurrentWeather(currentData, city);

                // 4. 渲染未来7天预报
                renderForecast(forecastData.daily);

                // 5. 显示天气容器，隐藏初始提示
                weatherContainer.classList.remove('hidden');
                initialHint.classList.add('hidden');

            } catch (err) {
                showError(err.message);
                weatherContainer.classList.add('hidden');
                initialHint.classList.remove('hidden');
            }
        }

        /**
         * 处理和风天气的错误码
         * @param {string} code - 错误码
         * @param {string} city - 查询的城市名
         */
        function handleQWeatherError(code, city) {
            let msg = '';
            switch(code) {
                case "401":
                    msg = 'API密钥无效！请检查密钥是否正确，或确认实名认证已通过';
                    break;
                case "404":
                    msg = `未找到「${city}」的天气数据！请检查城市名（如：需填「杭州市」而非「杭州」，小众区县加地级市）`;
                    break;
                case "429":
                    msg = 'API请求次数超限！免费版每天3000次、每分钟60次，请稍后重试';
                    break;
                case "500":
                    msg = '服务器内部错误，请稍后重试';
                    break;
                default:
                    msg = `查询失败：${code} - ${currentData.msg || '未知错误'}`;
            }
            showError(msg);
        }

        /**
         * 渲染当前天气（适配和风天气数据格式）
         * @param {object} data - 实时天气数据
         * @param {string} city - 城市名
         */
        function renderCurrentWeather(data, city) {
            const now = data.now;
            
            todayCity.textContent = `${city} (中国)`;
            todayDate.textContent = formatTodayDate();
            // 和风天气图标CDN
            todayIcon.src = `https://cdn.qweather.com/img/weather/${now.icon}.png`;
            todayIcon.alt = now.text;
            todayWeather.textContent = now.text;
            todayTemp.textContent = `${now.temp}°C`;
            feelsLike.textContent = `${now.feelsLike}°C`;
            humidity.textContent = `${now.humidity}%`;
            windSpeed.textContent = `${now.windSpeed} km/h`;
            pressure.textContent = `${now.pressure} hPa`;
        }

        /**
         * 渲染未来7天预报（适配和风天气数据格式）
         * @param {array} dailyData - 7天预报数据数组
         */
        function renderForecast(dailyData) {
            forecastList.innerHTML = '';
            // 遍历7天预报数据
            dailyData.forEach(day => {
                const forecastCard = document.createElement('div');
                forecastCard.className = 'bg-gray-50 rounded-lg p-4 text-center transition-all-300 hover:shadow-md hover:translate-y-[-2px]';
                
                // 格式化日期：MM-DD 周X
                const date = new Date(day.fxDate);
                const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
                const weekDay = weekDays[date.getDay()];
                const shortDate = `${String(date.getMonth()+1).padStart(2,0)}-${String(date.getDate()).padStart(2,0)}<br>${weekDay}`;

                forecastCard.innerHTML = `
                    <p class="text-secondary text-sm mb-2">${shortDate}</p>
                    <img src="https://cdn.qweather.com/img/weather/${day.iconDay}.png" alt="${day.textDay}" class="w-12 h-12 mx-auto mb-2">
                    <p class="text-sm text-gray-700 mb-1">${day.textDay}</p>
                    <div class="flex justify-center gap-2 text-sm">
                        <span class="text-red-500">↑ ${day.tempMax}°C</span>
                        <span class="text-blue-500">↓ ${day.tempMin}°C</span>
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
