<template>
  <section class="search">
    <input type="number" v-model="inputnomor" class="input" placeholder="masukan nomor surah"/>
    <button @click="lihat" class="btn">lihat</button>
  </section>

  <section class="surah">
    <div class="judul">
      <h1 v-if="namaSurah" class="judul">{{ namaSurah.name_sample }}</h1>
    </div>
    <div class="suara">
      <p v-if="audio">
        <audio controls class="suara"><source :src="audio.audio_url" type="audio/mpeg" /> your browser does not support the audio element</audio>
      </p>
    </div>
    <div class="bis">
      <p class="bismillah">بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم</p>
    </div>
  </section>

  <section class="hasil">
    <div class="hasill">
      <ul class="lista">
        <li class="ayat" v-for="ayat in ayats" :key="ayat.id">
          {{ ayat.text_utsmani }} {{ ayat.text }}
        </li>
      </ul>
    </div>
  </section>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      ayats:[],arti:[],audio:null,namaSurah:null,myinput:"",
    };
  },
  mounted() {
    this.lihat()
  },
  methods: {
    async lihat() {
      let nomor = this.inputnomor;
      let ayat = 'https://api.quran.com/api/v4/quran/verses/uthmani?chapter_number='+ nomor;
      let arti = "https://api.quran.com/api/v4/quran/translations/33?chapter_number=" +nomor;
      let judul = "https://api.quran.com/api/v4/chapters?language=id";
      let suara = "https://api.quran.com/api/v4/chapter_recitations/5?language=en";

      if (nomor <=0 || nomor > 114) {
        alert("masukkan nomor sesuai dengan nomor surah yang ada di Al-Quran");
      }else{
        const reqAyat = axios.get(ayat);
        const reqArti = axios.get(arti);
        const reqJudul = axios.get(judul);
        const reqSuara = axios.get(suara);

        axios.all([reqAyat, reqArti, reqJudul, reqSuara]).then(axios.spread((...response) => {
          let responseAyat = response[0];
          const responseArti = response[1];
          const responseJudul = response[2];
          const responseSuara = response[3];

          const a = responseAyat.data.verses;
          const b = responseArti.data.translations;

          const gabung = (a, b) => {
            const res = [];

            for (let i = 0; i < a.length + b.length; i++) {
              if (i % 2 === 0) {
                res.push(a[i / 2]);
              } else {
                res.push(b[(i - 1) / 2]);
              }
            }
            return res;
          };

          this.ayats = gabung(a,b);
          this.audio =
              responseSuara.data.audio_files[nomor - 1];
          this.namaSurah =
              responseJudul.data.chapters[nomor - 1];
        }))
      };
    }
  },
};

</script>

<style></style>