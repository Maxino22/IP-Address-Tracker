<script setup>
import { onMounted, ref } from "vue";
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import axios from "axios";

const queryIP = ref("");
const IpInfo = ref(null);
const usersIP = ref("");

let mymap;

onMounted(() => {
  mymap = leaflet.map("map").setView([-1.286389, 36.817223], 13);
  const accessToken =
    "pk.eyJ1IjoibWF4aW5vMjIiLCJhIjoiY2xjM3J2b28wMW5pcTN3cDlkdTdxbXN6aiJ9.yRpc5Bwp6o_cLwhTcTyvyg";

  leaflet
    .tileLayer(
      `https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${accessToken}`,
      {
        maxZoom: 18,
        id: "mapbox/street-v11",
        tileSize: 512,
        zoomOffset: -1,
        accessToken:
          "pk.eyJ1IjoibWF4aW5vMjIiLCJhIjoiY2xjM3J2b28wMW5pcTN3cDlkdTdxbXN6aiJ9.yRpc5Bwp6o_cLwhTcTyvyg",
        attribution:
          '© <a href="https://www.mapbox.com/feedback/">Mapbox</a> © <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      }
    )
    .addTo(mymap);
  // getting & setting users ip
  function getUserIP() {
    var VisitorAPI = function (t, e, a) {
      var s = new XMLHttpRequest();
      (s.onreadystatechange = function () {
        var t;
        s.readyState === XMLHttpRequest.DONE &&
          (200 === (t = JSON.parse(s.responseText)).status
            ? e(t.data)
            : a(t.status, t.result));
      }),
        s.open("GET", "https://api.visitorapi.com/api/?pid=" + t),
        s.send(null);
    };
    VisitorAPI(
      "nqQecoOuuWfYjXKVGHXF",
      function (data) {
        usersIP.value = data.ipAddress;
        console.log(usersIP.value);
      },
      function (errorCode, errorMessage) {
        console.log(errorCode, errorMessage);
      }
    );
  }

  getUserIP();
});

async function getIPInfo() {
  try {
    const data = await axios.post("https://flask-web-ip.onrender.com/api", {
      ip_address: queryIP.value === "" ? usersIP.value : queryIP.value,
    });

    const result = data.data;
    IpInfo.value = {
      address: result.ip,
      state: result.location.region,
      timezone: result.location.timezone,
      isp: result.isp,
      lat: result.location.lat,
      lng: result.location.lng,
    };
    leaflet.marker([IpInfo.value.lat, IpInfo.value.lng]).addTo(mymap);
    mymap.setView([IpInfo.value.lat, IpInfo.value.lng], 13);
  } catch (err) {
    console.log(err);
  }
}
</script>

<template>
  <main>
    <div class="flex flex-col h-screen max-h-screen">
      <!-- search/results -->
      <div
        class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
      >
        <!-- serch input -->
        <div class="w-full max-w-screen-sm">
          <h1 class="text-white text-center text-3xl pb-4">
            IP Adress Tracker
          </h1>
          <div class="flex">
            <input
              v-model="queryIP"
              class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
              type="text"
              placeholder="Search for any IP address or leave empty to get your  ip info"
            />
            <i
              @click="getIPInfo"
              class="fas fa-chevron-right cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center"
            ></i>
          </div>
        </div>
        <!-- Ip Info-->
        <IPInfo v-if="IpInfo" :ipInfo="IpInfo" />
      </div>
      <!-- map -->
      <div id="map" class="h-full z-10"></div>
    </div>
  </main>
</template>
