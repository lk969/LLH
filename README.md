<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简易天气查询 | 高德API完整版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#409EFF',
                        secondary: '#666E78',
                        hint: '#909399',
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .card-shadow {
                box-shadow: 0 4px 12px rgba(0,0,0,0.08);
            }
            .text-nowrap-pre {
                white-space: pre-line;
            }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen flex flex-col">
    <!-- 头部区域 -->
    <header class="bg-primary text-white py-4 shadow-md">
        <div class="container mx-auto px-4 flex justify-between items-center">
            <h1 class="text-2xl font-bold flex items-center gap-2">
                <i class="fas fa-cloud-sun"></i>
                简易天气查询
            </h1>
            <p class="text-sm opacity-90">实时天气 · 7天预报</p>
        </div>
    </header>

    <!-- 主内容区 -->
    <main class="container mx-auto px-4 py-8 max-w-4xl flex-grow">
        <!-- 搜索区域 -->
        <div class="mb-4 flex gap-3 flex-col sm:flex-row">
            <input 
                type="text" 
                id="cityInput" 
                placeholder="输入城市名称（如：北京、上海、杭州）"
                class="flex-1 px-4 py-3 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-primary/50"
                value="北京"
            >
            <button 
                id="queryBtn"
                class="bg-primary text-white px-6 py-3 rounded-lg hover:bg-primary/90 transition-colors"
            >
                <i class="fas fa-search mr-1"></i>查询
            </button>
        </div>

        <!-- 温馨提示（API限制说明） -->
        <div class="mb-6 text-sm text-hint p-3 bg-gray-100 rounded-lg">
            <i class="fas fa-info-circle mr-2"></i>
            提示：高德免费版API暂不返回部分城市的湿度、风速、气压等详情字段，属于正常现象
        </div>

        <!-- 错误提示（默认隐藏） -->
        <div id="errorTip" class="hidden mb-6 text-red-500 text-sm p-3 bg-red-50 rounded-lg flex items-center">
            <i class="fas fa-exclamation-circle mr-2"></i>
            <span id="errorText">请求失败，请重试</span>
        </div>

        <!-- 加载提示（默认隐藏） -->
        <div id="loadingTip" class="hidden mb-6 text-blue-500 text-sm p-3 bg-blue-50 rounded-lg flex items-center">
            <i class="fas fa-spinner fa-spin mr-2"></i>
            <span>正在查询天气数据...</span>
        </div>

        <!-- 天气展示区域（默认隐藏） -->
        <div id="weatherCard" class="hidden bg-white rounded-xl p-6 card-shadow mb-8">
            <!-- 实时天气头部 -->
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-6">
                <div>
                    <h2 id="cityName" class="text-2xl font-bold text-gray-800">北京市</h2>
                    <p id="currentDate" class="text-secondary text-sm mt-1">2026-01-02 周五</p>
                </div>
                <div class="flex items-center gap-4">
                    <img id="weatherIcon" src="" alt="天气图标" class="w-16 h-16">
                    <div class="text-center">
                        <p id="weatherText" class="text-lg text-secondary">晴</p>
                        <p id="temp" class="text-4xl font-bold text-gray-800 mt-1">2°C</p>
                    </div>
                </div>
            </div>

            <!-- 实时天气详情 -->
            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mt-6">
                <div class="text-center p-3 bg-gray-50 rounded-lg">
                    <p class="text-xs text-secondary">体感温度</p>
                    <p id="feelTemp" class="text-lg font-medium mt-1">2°C</p>
                    <p class="text-xs text-hint mt-1">（参考当前温度）</p>
                </div>
                <div class="text-center p-3 bg-gray-50 rounded-lg">
                    <p class="text-xs text-secondary">湿度</p>
                    <p id="humidity" class="text-lg font-medium mt-1">暂未获取</p>
                </div>
                <div class="text-center p-3 bg-gray-50 rounded-lg">
                    <p class="text-xs text-secondary">风速</p>
                    <p id="windSpeed" class="text-lg font-medium mt-1">暂未获取</p>
                </div>
                <div class="text-center p-3 bg-gray-50 rounded-lg">
                    <p class="text-xs text-secondary">气压</p>
                    <p id="pressure" class="text-lg font-medium mt-1">暂未获取</p>
                </div>
            </div>
        </div>

        <!-- 7天预报区域（默认隐藏） -->
        <div id="forecastCard" class="hidden bg-white rounded-xl p-6 card-shadow">
            <h3 class="text-lg font-bold text-gray-800 mb-4">未来7天预报</h3>
            <div id="forecastList" class="grid grid-cols-2 sm:grid-cols-4 md:grid-cols-7 gap-3">
                <!-- 预报卡片会动态生成 -->
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="py-4 text-center text-sm text-secondary">
        <p>数据来源：高德地图天气API | 免费版API字段有限，仅供学习使用</p>
    </footer>

    <script>
        // 高德API配置（已填入你的密钥）
        const AMAP_KEY = "84b0ed4ededbaac14e90d1f1feac6ae8";
        const GEO_API = "https://restapi.amap.com/v3/geocode/geo";
        const WEATHER_API = "https://restapi.amap.com/v3/weather/weatherInfo";

        // DOM元素
        const cityInput = document.getElementById('cityInput');
        const queryBtn = document.getElementById('queryBtn');
        const errorTip = document.getElementById('errorTip');
        const errorText = document.getElementById('errorText');
        const loadingTip = document.getElementById('loadingTip');
        const weatherCard = document.getElementById('weatherCard');
        const forecastCard = document.getElementById('forecastCard');
        const forecastList = document.getElementById('forecastList');

        // 天气图标映射
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
            '霾': 'https://webapi.amap.com/images/weather/icon/weather/day/21.png',
            '阵雨': 'https://webapi.amap.com/images/weather/icon/weather/day/06.png',
            '冻雨': 'https://webapi.amap.com/images/weather/icon/weather/day/11.png',
            '雨夹雪': 'https://webapi.amap.com/images/weather/icon/weather/day/14.png',
            '暴雪': 'https://webapi.amap.com/images/weather/icon/weather/day/18.png',
            '扬沙': 'https://webapi.amap.com/images/weather/icon/weather/day/19.png'
        };

        // ===================== 核心工具函数 =====================
        /**
         * 格式化日期（兼容无效日期）
         */
        function formatDate(dateStr) {
            try {
                const date = new Date(dateStr);
                if (isNaN(date.getTime())) throw new Error('无效日期');
                const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
                return `${date.getFullYear()}-${(date.getMonth()+1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')} ${weekDays[date.getDay()]}`;
            } catch (err) {
                const now = new Date();
                const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
                return `${now.getFullYear()}-${(now.getMonth()+1).toString().padStart(2, '0')}-${now.getDate().toString().padStart(2, '0')} ${weekDays[now.getDay()]}`;
            }
        }

        /**
         * 简化日期（兼容无效日期）
         */
        function shortDate(dateStr) {
            try {
                const date = new Date(dateStr);
                if (isNaN(date.getTime())) throw new Error('无效日期');
                const weekDays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
                return `${(date.getMonth()+1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')}<br>${weekDays[date.getDay()]}`;
            } catch (err) {
                return '--<br>--';
            }
        }

        /**
         * 字段兜底函数（区分空值和API无数据）
         */
        function formatField(value, unit = '') {
            // 有效数据：返回值+单位
            if (value && value !== '' && value !== 'null' && !isNaN(value)) {
                return `${value}${unit}`;
            }
            // 无数据：返回友好提示
            return '暂未获取';
        }

        /**
         * 显示错误提示
         */
        function showError(msg) {
            errorText.textContent = msg;
            errorTip.classList.remove('hidden');
            loadingTip.classList.add('hidden');
            weatherCard.classList.add('hidden');
            forecastCard.classList.add('hidden');
            console.error('查询错误：', msg);
        }

        /**
         * 显示/隐藏加载提示
         */
        function showLoading() { loadingTip.classList.remove('hidden'); }
        function hideLoading() { loadingTip.classList.add('hidden'); }
        function hideError() { errorTip.classList.add('hidden'); }

        // ===================== 核心业务逻辑 =====================
        /**
         * 城市名称转adcode（强化容错）
         */
        async function getCityAdcode(cityName) {
            const cleanCity = cityName.replace(/市|区|县|省/g, '').trim();
            if (!cleanCity) throw new Error('城市名称不能为空');

            // 省份映射提升解析率
            const provinceMap = {
                北京: '北京市', 上海: '上海市', 天津: '天津市', 重庆: '重庆市',
                杭州: '浙江省', 广州: '广东省', 深圳: '广东省', 成都: '四川省',
                南京: '江苏省', 武汉: '湖北省', 西安: '陕西省', 郑州: '河南省'
            };

            // 第一轮解析
            let res = await fetch(`${GEO_API}?address=${encodeURIComponent(cleanCity)}&key=${AMAP_KEY}`);
            let data = await res.json();

            // 第二轮：补充省份重试
            if (data.status !== '1' || data.geocodes.length === 0) {
                const province = provinceMap[cleanCity] || '';
                if (province) {
                    res = await fetch(`${GEO_API}?address=${encodeURIComponent(`${province}${cleanCity}`)}&key=${AMAP_KEY}`);
                    data = await res.json();
                }
            }

            if (data.status !== '1' || data.geocodes.length === 0) {
                throw new Error(`未找到“${cityName}”的地理信息，请输入省会/地级市（如：北京、杭州）`);
            }

            return {
                adcode: data.geocodes[0].adcode,
                city: data.geocodes[0].city || data.geocodes[0].province || cleanCity
            };
        }

        /**
         * 获取天气数据（兼容空字段）
         */
        async function getWeatherData(adcode) {
            const res = await fetch(`${WEATHER_API}?city=${adcode}&extensions=all&key=${AMAP_KEY}`);
            const data = await res.json();
            console.log('高德API原始返回：', data); // 调试用，查看实际返回字段

            if (data.status !== '1') {
                throw new Error(`天气接口错误：${data.info || '未知错误'}`);
            }

            if (!data.forecasts || data.forecasts.length === 0 || !data.forecasts[0].casts) {
                throw new Error(`暂无“${adcode}”的天气预报数据`);
            }

            return {
                realtime: data.lives && data.lives.length > 0 ? data.lives[0] : {},
                forecast: data.forecasts[0].casts,
                reportTime: data.forecasts[0].reporttime || new Date().toString(),
                cityName: data.forecasts[0].city || ''
            };
        }

        /**
         * 渲染实时天气（重点优化湿度/风速/气压兜底）
         */
        function renderRealtimeWeather(city, realtime, forecastToday, reportTime) {
            // 核心字段：温度/天气类型
            const weatherType = realtime.weather || forecastToday.dayweather || '晴';
            const temp = realtime.temperature || forecastToday.daytemp;

            // 基础信息渲染
            document.getElementById('cityName').textContent = city;
            document.getElementById('currentDate').textContent = formatDate(reportTime);
            document.getElementById('weatherIcon').src = weatherIconMap[weatherType] || weatherIconMap['晴'];
            document.getElementById('weatherText').textContent = weatherType;
            document.getElementById('temp').textContent = formatField(temp, '°C');

            // 详情字段渲染（重点优化）
            document.getElementById('feelTemp').textContent = formatField(realtime.temperature || forecastToday.daytemp, '°C');
            document.getElementById('humidity').textContent = formatField(realtime.humidity, '%');
            document.getElementById('windSpeed').textContent = formatField(realtime.windpower, '级');
            document.getElementById('pressure').textContent = formatField(realtime.pressure, ' hPa');

            weatherCard.classList.remove('hidden');
        }

        /**
         * 渲染7天预报
         */
        function renderForecast(forecastListData) {
            forecastList.innerHTML = '';
            const validForecast = forecastListData.slice(0, 7);

            if (validForecast.length === 0) {
                forecastList.innerHTML = '<div class="col-span-full text-center py-4 text-secondary">暂无预报数据</div>';
                forecastCard.classList.remove('hidden');
                return;
            }

            validForecast.forEach(day => {
                const card = document.createElement('div');
                card.className = 'text-center p-3 bg-gray-50 rounded-lg';
                card.innerHTML = `
                    <p class="text-xs text-secondary text-nowrap-pre">${shortDate(day.date)}</p>
                    <img 
                        src="${weatherIconMap[day.dayweather] || weatherIconMap['晴']}" 
                        alt="${day.dayweather}" 
                        class="w-10 h-10 mx-auto my-2"
                    >
                    <p class="text-xs">${day.dayweather || '--'}</p>
                    <p class="text-sm mt-1">
                        <span class="text-red-500">${formatField(day.daytemp || day.highest, '°')}</span>
                        <span class="text-blue-500 ml-1">${formatField(day.nighttemp || day.lowest, '°')}</span>
                    </p>
                `;
                forecastList.appendChild(card);
            });

            forecastCard.classList.remove('hidden');
        }

        /**
         * 主查询逻辑
         */
        async function queryWeather() {
            const city = cityInput.value.trim();
            if (!city) {
                showError('请输入城市名称');
                return;
            }

            try {
                showLoading();
                hideError();

                // 1. 城市转adcode
                const { adcode, city: cityName } = await getCityAdcode(city);
                // 2. 获取天气数据
                const { realtime, forecast, reportTime, cityName: apiCityName } = await getWeatherData(adcode);
                const finalCityName = apiCityName || cityName;
                // 3. 渲染数据
                renderRealtimeWeather(finalCityName, realtime, forecast[0], reportTime);
                renderForecast(forecast);

                hideLoading();
            } catch (err) {
                showError(err.message);
                hideLoading();
            }
        }

        // ===================== 事件绑定 =====================
        queryBtn.addEventListener('click', queryWeather);
        cityInput.addEventListener('keydown', (e) => e.key === 'Enter' && queryWeather());
        window.addEventListener('load', queryWeather); // 页面加载自动查询默认城市
    </script>
</body>
</html>
