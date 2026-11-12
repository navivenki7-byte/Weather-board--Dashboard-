<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Weather Board Dashboard</title>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: Arial, sans-serif;
}

body {
    min-height: 100vh;
    background: linear-gradient(135deg, #071b38, #123f68, #087e9c);
    color: white;
    padding: 25px;
}

.container {
    max-width: 1100px;
    margin: auto;
}

header {
    text-align: center;
    margin-bottom: 30px;
}

header h1 {
    font-size: 32px;
    margin-bottom: 8px;
}

.search {
    display: flex;
    gap: 10px;
    max-width: 500px;
    margin: 25px auto;
}

input {
    flex: 1;
    padding: 14px;
    border: none;
    border-radius: 10px;
    outline: none;
    font-size: 16px;
}

button {
    padding: 14px 22px;
    border: none;
    border-radius: 10px;
    background: #00c6ff;
    color: #071b38;
    font-weight: bold;
    cursor: pointer;
}

button:hover {
    background: #65e0ff;
}

.main-card {
    background: rgba(255,255,255,0.13);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(255,255,255,0.2);
    border-radius: 20px;
    padding: 30px;
    text-align: center;
    margin-bottom: 25px;
}

.city {
    font-size: 26px;
    font-weight: bold;
}

.temperature {
    font-size: 65px;
    font-weight: bold;
    margin: 15px 0;
}

.weather-icon {
    font-size: 65px;
}

.description {
    font-size: 20px;
    margin-bottom: 10px;
}

.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 18px;
    margin-bottom: 30px;
}

.card {
    background: rgba(255,255,255,0.13);
    padding: 25px;
    border-radius: 16px;
    text-align: center;
    border: 1px solid rgba(255,255,255,0.15);
}

.card h3 {
    font-size: 16px;
    margin-bottom: 12px;
    color: #bceeff;
}

.card p {
    font-size: 25px;
    font-weight: bold;
}

.forecast {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 12px;
}

.forecast-card {
    background: rgba(255,255,255,0.13);
    padding: 18px 8px;
    text-align: center;
    border-radius: 14px;
}

.forecast-card h4 {
    margin-bottom: 12px;
}

.forecast-card p {
    margin-top: 8px;
}

.error {
    text-align: center;
    color: #ffcccc;
    margin: 15px;
}

footer {
    text-align: center;
    margin-top: 35px;
    color: #cceeff;
    font-size: 13px;
}

@media(max-width: 700px) {
    .grid {
        grid-template-columns: repeat(2, 1fr);
    }

    .forecast {
        grid-template-columns: repeat(2, 1fr);
    }

    .temperature {
        font-size: 50px;
    }
}

@media(max-width: 400px) {
    .search {
        flex-direction: column;
    }
}
</style>
</head>

<body>

<div class="container">

<header>
    <h1>🌦 Weather Dashboard</h1>
    <p>Live Weather Monitoring System</p>
</header>

<div class="search">
    <input type="text" id="cityInput"
           placeholder="Enter city name..."
           value="Chennai">
    <button onclick="searchWeather()">Search</button>
</div>

<p class="error" id="error"></p>

<div class="main-card">
    <div class="city" id="city">Chennai</div>
    <div class="weather-icon" id="icon">🌤️</div>
    <div class="temperature" id="temp">--°C</div>
    <div class="description" id="description">
        Loading weather...
    </div>
    <p id="date"></p>
</div>

<h2 style="margin-bottom:18px;">Weather Details</h2>

<div class="grid">

    <div class="card">
        <h3>💧 Humidity</h3>
        <p id="humidity">--%</p>
    </div>

    <div class="card">
        <h3>💨 Wind Speed</h3>
        <p id="wind">-- km/h</p>
    </div>

    <div class="card">
        <h3>🌡 Feels Like</h3>
        <p id="feels">--°C</p>
    </div>

    <div class="card">
        <h3>🌧 Rain Probability</h3>
        <p id="rain">--%</p>
    </div>

    <div class="card">
        <h3>☁ Cloud Cover</h3>
        <p id="cloud">--%</p>
    </div>

    <div class="card">
        <h3>🌡 Pressure</h3>
        <p id="pressure">-- hPa</p>
    </div>

</div>

<h2 style="margin-bottom:18px;">5-Day Forecast</h2>

<div class="forecast" id="forecast">
    <p>Loading forecast...</p>
</div>

<footer>
    <p>Weather Board Dashboard | Powered by Open-Meteo API</p>
    <p>Student Mini Project</p>
</footer>

</div>

<script>

const weatherCodes = {
    0: ["Clear Sky", "☀️"],
    1: ["Mainly Clear", "🌤️"],
    2: ["Partly Cloudy", "⛅"],
    3: ["Overcast", "☁️"],
    45: ["Foggy", "🌫️"],
    48: ["Foggy", "🌫️"],
    51: ["Light Drizzle", "🌦️"],
    53: ["Drizzle", "🌦️"],
    55: ["Heavy Drizzle", "🌧️"],
    61: ["Light Rain", "🌧️"],
    63: ["Moderate Rain", "🌧️"],
    65: ["Heavy Rain", "🌧️"],
    71: ["Snow", "❄️"],
    80: ["Rain Showers", "🌦️"],
    81: ["Rain Showers", "🌧️"],
    82: ["Heavy Showers", "⛈️"],
    95: ["Thunderstorm", "⛈️"]
};

function getWeather(code) {
    return weatherCodes[code] || ["Variable Weather", "🌡️"];
}

async function searchWeather() {

    const cityInput = document.getElementById("cityInput");
    const cityName = cityInput.value.trim();

    const error = document.getElementById("error");

    if (!cityName) {
        error.textContent = "Please enter a city name.";
        return;
    }

    error.textContent = "Loading weather data...";

    try {

        // Find city coordinates
        const geoURL =
        `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cityName)}&count=1&language=en&format=json`;

        const geoResponse = await fetch(geoURL);
        if (!geoResponse.ok) throw new Error("Location search failed");

        const geoData = await geoResponse.json();

        if (!geoData.results || geoData.results.length === 0) {
            throw new Error("City not found. Try another city.");
        }

        const location = geoData.results[0];

        const lat = location.latitude;
        const lon = location.longitude;

        // Get live weather and forecast
        const weatherURL =
        `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relative_humidity_2m,apparent_temperature,is_day,precipitation,weather_code,cloud_cover,surface_pressure,wind_speed_10m&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=auto&forecast_days=5`;

        const response = await fetch(weatherURL);
        if (!response.ok) throw new Error("Weather service unavailable");

        const data = await response.json();

        const current = data.current;

        const weather = getWeather(current.weather_code);

        document.getElementById("city").textContent =
            `${location.name}, ${location.country}`;

        document.getElementById("icon").textContent =
            weather[1];

        document.getElementById("temp").textContent =
            `${Math.round(current.temperature_2m)}°C`;

        document.getElementById("description").textContent =
            weather[0];

        document.getElementById("humidity").textContent =
            `${current.relative_humidity_2m}%`;

        document.getElementById("wind").textContent =
            `${current.wind_speed_10m} km/h`;

        document.getElementById("feels").textContent =
            `${Math.round(current.apparent_temperature)}°C`;

        document.getElementById("rain").textContent =
            `${current.precipitation} mm`;

        document.getElementById("cloud").textContent =
            `${current.cloud_cover}%`;

        document.getElementById("pressure").textContent =
            `${Math.round(current.surface_pressure)} hPa`;

        document.getElementById("date").textContent =
            `Local time: ${current.time.replace("T", " ")}`;

        // Display forecast
        const forecast = document.getElementById("forecast");
        forecast.innerHTML = "";

        data.daily.time.forEach((day, index) => {

            const dayWeather = getWeather(
                data.daily.weather_code[index]
            );

            const date = new Date(day + "T12:00:00");

            const dayName = date.toLocaleDateString(
                "en-US", { weekday: "short" }
            );

            const card = document.createElement("div");
            card.className = "forecast-card";

            const heading = document.createElement("h4");
            heading.textContent = dayName;

            const emoji = document.createElement("div");
            emoji.style.fontSize = "30px";
            emoji.textContent = dayWeather[1];

            const condition = document.createElement("p");
            condition.textContent = dayWeather[0];

            const temperature = document.createElement("p");
            temperature.textContent =
                `${Math.round(data.daily.temperature_2m_max[index])}° / ${Math.round(data.daily.temperature_2m_min[index])}°C`;

            const rain = document.createElement("p");
            rain.textContent =
                `Rain: ${data.daily.precipitation_probability_max[index] ?? "N/A"}%`;

            card.append(heading, emoji, condition, temperature, rain);
            forecast.appendChild(card);
        });

        error.textContent = "";

    } catch (err) {

        error.textContent = err.message ||
            "Unable to load weather data. Please try again.";

    }
}

// Search using Enter key
document.getElementById("cityInput")
    .addEventListener("keydown", function(event) {
        if (event.key === "Enter") {
            searchWeather();
        }
    });

// Load default city
searchWeather();

</script>

</body>
</html># Weather-board--Dashboard-