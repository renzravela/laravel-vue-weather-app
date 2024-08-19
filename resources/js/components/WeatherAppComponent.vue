<template>
    <div id="main-app" :class="backgroundClass">
        <main>
            <div class="form-group">
                <input type="text" class="mt-5 w-50 offset-3 form-control" v-model="query" @keypress="fetchWeather"
                    placeholder="Search for a city">
            </div>
            <div class="weather-wrap">
                <div class="location-box">
                    <div class="location">
                        <span v-if="error">City not found</span>
                        <span v-else-if="!weather.main">Search for a city</span>
                        <span v-else>{{ weather.name }}, {{ weather.sys?.country }}</span>
                    </div>
                    <div class="date">
                        {{ dateBuilder() }}
                    </div>
                </div>
                <div class="weather-box" v-if="typeof weather.main !== 'undefined' && !error">
                    <div class="temp">
                        <span>{{ Math.round(weather.main.temp) }}°C</span>
                    </div>
                    <div class="weather">
                        <span>{{ weather.weather[0]?.main }}</span>
                    </div>
                </div>
                <!-- Display venue information -->
                <div class="venues-wrap" v-if="venues.length">
                    <h2>Nearby Venues</h2>
                    <ul>
                        <li v-for="venue in venues" :key="venue.id">
                            {{ venue.name }} - {{ venue.location.address || 'No address available' }}
                        </li>
                    </ul>
                </div>
            </div>
        </main>
    </div>
</template>

<script>
export default {
    name: "WeatherAppComponent",
    data() {
        return {
            query: '',
            weather: {},
            error: false,
            venues: [],
            foursquareClientId: '3OQ0WSQ0T2ACDNE23D1HZ2V3QEFKMNCNPG5GIQQUY0GUD0KB',
            foursquareClientSecret: 'AO3JMTXCA4GARTTIYOCZJCDLN1D1OULJHEX4GA4QEFFEYBZ4',
            openWeatherMapKey: 'dc78a70f3f9ef9674fe6a80456a810b3',
            foursquareClientToken: '3RRVWEWXZU3SRSQV35WXJRY2H4ZEZUO242HFOL5WZR4G153T',
        };
    },
    computed: {
        backgroundClass() {
            if (typeof this.weather.main !== 'undefined') {
                const weatherType = this.weather.weather[0]?.main.toLowerCase();
                const temp = this.weather.main.temp;

                if (weatherType === 'rain' || weatherType === 'snow') {
                    return weatherType;
                } else if (temp < 10) {
                    return 'cold';
                } else if (temp < 25) {
                    return 'warm';
                } else {
                    return 'sunny';
                }
            }
            return 'sunny';
        }
    },
    methods: {
        fetchWeather(e) {
            if (e.key === 'Enter') {
                const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(this.query)}&units=metric&appid=${this.openWeatherMapKey}`;
                fetch(weatherUrl)
                    .then((res) => {
                        if (!res.ok) {
                            throw new Error('City not found');
                        }
                        return res.json();
                    })
                    .then((results) => {
                        this.setResults(results);
                        this.fetchVenues(); // Fetch venues after weather is fetched
                    })
                    .catch((error) => {
                        this.handleError(error);
                    });
            }
        },
        setResults(results) {
            this.weather = results || {};
            this.error = false;
            console.log(this.weather);
        },
        handleError(error) {
            console.error(error.message);
            this.weather = {};
            this.error = true;
            this.venues = [];
        },
        fetchVenues() {
            if (this.weather.coord) {
                const { lat, lon } = this.weather.coord;
                const venuesUrl = `https://api.foursquare.com/v3/places/search?ll=${lat},${lon}&radius=1000&categories=13001,13007&limit=5`;
                const headers = {
                    'Authorization': `Bearer ${this.foursquareClientToken}` // Ensure this token is correct and not expired
                };

                fetch(venuesUrl, { headers })
                    .then((res) => {
                        if (!res.ok) {
                            throw new Error('Failed to fetch venues');
                        }
                        return res.json();
                    })
                    .then((data) => {
                        this.venues = data.results || [];
                    })
                    .catch((error) => {
                        console.error('Error fetching venues:', error);
                    });
            }
        },
        dateBuilder() {
            const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
            const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
            const d = new Date();
            const day = days[d.getDay()];
            const month = months[d.getMonth()];
            const date = d.getDate();
            const year = d.getFullYear();
            return `${day}, ${month} ${date}, ${year}`;
        }
    }
};
</script>

<style scoped>
#main-app {
    background-size: cover;
    background-position: bottom;
    transition: 0.4s;
}

#main-app.sunny {
    background-image: url("/assets/sunny-bg.jpg");
}

#main-app.cold {
    background-image: url("/assets/cold-bg.jpg");
}

#main-app.warm {
    background-image: url("/assets/warm-bg.jpg");
}

#main-app.rain {
    background-image: url("/assets/rain-bg.jpg");
}

main {
    min-height: 100vh;
    padding: 25px;
    background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}

.location-box .location {
    color: #fff;
    font-size: 32px;
    font-weight: 500;
    text-align: center;
    text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
    color: #fff;
    font-size: 20px;
    font-weight: 300;
    text-align: center;
    font-style: italic;
}

.weather-box {
    text-align: center;
}

.weather-box .temp {
    display: inline-block;
    padding: 10px 25px;
    color: #fff;
    font-size: 100px;
    font-weight: 900;
    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.25);
    border-radius: 16px;
    margin: 30px 0;
    box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
    color: #fff;
    font-size: 48px;
    font-weight: 700;
    font-style: italic;
    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.venue-box {
    margin-top: 20px;
}

.venue-box .venue {
    color: #fff;
    font-size: 20px;
    margin: 10px 0;
}

.venue-box .venue h3 {
    font-size: 24px;
}
</style>
