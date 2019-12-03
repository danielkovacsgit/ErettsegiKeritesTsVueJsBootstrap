<template>
  <div id="app" class="ff">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <font-awesome-icon icon="vuejs" class="mr-1" />
      <b-navbar-brand href="#">Linkek</b-navbar-brand>
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item href="kerites.txt" download="file.txt">Kerítés.txt</b-nav-item>
          <b-nav-item href="Kerítés_fel.pdf" target="_blank">Kerítés feladat</b-nav-item>
          <b-nav-item href="Kerítés_jav.pdf" target="_blank">Javítási útmutató</b-nav-item>
        </b-navbar-nav>
        <!-- Right aligned nav items -->
        <b-navbar-nav class="ml-auto">
          <b-navbar-nav right>
            <b-nav-item href="https://github.com/danielkovacsgit/ErettsegiKeritesTsVueJsBootstrap" target="_blank"
              >Forrás</b-nav-item
            >
            <b-nav-item href="https://github.com/nitslaszlo/JedlikVueJsStarter" target="_blank">SDK</b-nav-item>
          </b-navbar-nav>
          <!-- <b-nav-item-dropdown text="Lang" right>
            <b-dropdown-item href="#">English</b-dropdown-item>
            <b-dropdown-item href="#">Deutsch</b-dropdown-item>
            <b-dropdown-item href="#">Magyar</b-dropdown-item>
          </b-nav-item-dropdown> -->
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>

    <button type="button" class="btn btn-outline-primary btn-sm mx-auto m-3 btn">
      <TxtReader title="Kérem töltse fel a forrás (kerites.txt) állományt!" @load="txtSorai = $event" />
    </button>

    <div v-if="mutat" id="megoldas" class="ff">
      <div class="p-3 mb-2 bg-info text-white rounded">
        <p>2. feladat<br />Az eladott telkek száma: {{ telkek.length }}</p>
        <p>
          3. feladat<br />A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket<br />
          Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
        </p>
        <p>4. feladat<br />A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}</p>
        <p>
          5. feladat<br />Adjon meg egy házszámot!
          <input v-model="hazszamInputStr" type="number" min="1" max="117" /><br />
          A kerírés színe / állapota: {{ keritesSzineAllapota }}<br />
          Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
        </p>
      </div>
      <p v-if="mutat">
        <button id="save" class="btn rounded btn-primary">
          <font-awesome-icon icon="save" class="mr-1" />
          <TxtWriter title="Fájl mentése" :content="utcakep" filename="utcakep.txt" />
        </button>
      </p>
    </div>
    <!-- Megoldás DIV -->

    <!-- Nem a feladat része : -->

    <!-- -->
      <div v-if="mutat" id="egyebek" class="">
        <pre>
          <div id="utcakep.txt" class="overflow-auto m-3">
  utcakep.txt fájl:
  {{ utcakep }}
          </div>
          <div id="kerites.txt">
  kerites.txt fájl:
  {{ txtSorai }}
          </div>
        </pre>
      </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import Telek from "./telek";
import TxtReader from "./components/TxtReader.vue";
import TxtWriter from "./components/TxtWriter.vue";

// eslint-disable-next-line
@Component({ components: { TxtReader, TxtWriter } })
export default class App extends Vue {
  private telkek: Telek[] = [];
  private txtSorai: string = ""; // Watch végett nem lehet ékezetes azonosító! (pl.: forrás)!
  private mutat: boolean = false;
  // 5. feladat:
  private hazszamInputStr: string = "83";
  // 6. feladat:
  private utcakep: string = "";

  @Watch("txtSorai", { immediate: true, deep: true })
  private haForrasValtozik(val: string) {
    if (val !== "") this.feldolgoz();
  }

  private feldolgoz(): void {
    try {
      Telek.hazszamReset(); // statikus mezők visszaállítása
      this.txtSorai.split("\n").forEach(i => {
        const aktSor: string = i.trim();
        if (aktSor.length > 0) this.telkek.push(new Telek(aktSor));
      });
      // 6. utcakep generalasa
      let sor1: string = "";
      let sor2: string = "";
      for (const i of this.telkek.filter(x => x.paratlanOldali)) {
        sor1 += "".padEnd(i.telekSzelessege, i.keritesSzine);
        sor2 += i.hazSzama.toString().padEnd(i.telekSzelessege, " ");
      }
      this.utcakep = `${sor1}\n${sor2}\n`;

      this.mutat = true;
    } catch (error) {
      this.mutat = false;
      this.telkek = [];
      this.txtSorai = "";
      window.alert("Hibás forrás!");
    }
  }

  private get utolsoTelek(): Telek {
    return this.telkek[this.telkek.length - 1];
  }

  private get ugyanOlyanSzinuKerites(): number {
    let elozoTelek: Telek; // induláskor undefined értékű
    // elozoKerites! -> igaz, ha értéke nem null vagy undefined
    for (const aktTelek of this.telkek.filter(x => x.paratlanOldali)) {
      if (
        elozoTelek! &&
        aktTelek.keritesSzine !== "#" &&
        aktTelek.keritesSzine !== ":" &&
        aktTelek.keritesSzine === elozoTelek!.keritesSzine
      ) {
        return elozoTelek!.hazSzama;
      } else elozoTelek = aktTelek; // ha még undefined az ElozoTelek
    }
    return -1; // id a feladatkiírás szerint már nem juthat el
  }

  private get keritesSzineAllapota(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    if (keresettTelek.length !== 0) {
      return keresettTelek[0].keritesSzine;
    } else {
      return "Nincs ilyen házszám!"; // ez nem volt feladat, de így szép
    }
  }

  private get lehetsegesFestesiSzin(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    const balSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput + 2);
    const jobbSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput - 2);
    let lehetsegesSzinek: string[] = ["A", "B", "C", "D"];
    if (keresettTelek.length !== 0) {
      // ha van telek, aminek a kerítését festeni kell
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== keresettTelek[0].keritesSzine);
    } else return "Nincs ilyen házszám!";
    // ha van bal szomszéd:
    if (balSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== balSzomszedTelek[0].keritesSzine);
    }
    // ha van jobb szomszéd:
    if (jobbSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== jobbSzomszedTelek[0].keritesSzine);
    }
    return lehetsegesSzinek[0];
  }
}
</script>

<style>
#app {
  font-family: Courier;
}

#megoldas {
  /* background-color: lightgrey;*/
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 500px;
}

a {
  text-decoration: none;
  padding-left: 10px;
}

input[type="number"] {
  background-color: lightgray;
}

pre {
  font-size: 1.1em;
  margin: 0;
}

#save {
  border: none;
  background: #404040;
  color: #ffffff !important;
  font-weight: 100;
  padding: 20px;
  text-transform: uppercase;
  border-radius: 6px;
  display: inline-block;
  transition: all 0.3s ease 0s;
}
#save:hover {
  color: #404040 !important;
  font-weight: 700 !important;
  letter-spacing: 3px;
  background: none;
  box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  -webkit-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  -moz-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  transition: all 0.3s ease 0s;
}
#utcakep.txt{
  overflow-y: scroll;
}
.ff{
  font-family: Tahoma, Geneva, Verdana, sans-serif
}
</style>
