---
sidebar_position: 4
sidebar_label: "Mini-Project #2 - Frontend (client)"
---

# Mini-Project #2 - Frontend (Vue.js - chart.js - Vuetify)

**Objectif :** Réaliser une vue permettant la rétribution des données sous la forme d'un dashboard de graphiques

**Contraintes :** Utilisation de Vue.js - Une seule vue dynamiquement remplie - Réalisation de plusieurs graphiques

## Initialisation du projet

```sh
vue create frontend
cd frontend/
```

## Installation / Lancement du projet

```sh
yarn install
yarn dev

$ vite

  VITE v4.5.0  ready in 420 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help
```

## Charte graphique avec Balsamiq

- Police d'écriture : **Figtree**
- Tailles de police d'écriture :
    - Titres : **Figtree Semibold 600**
    - Menus et sous-titres : **Figtree Regular 400**
    - Corps de texte : **Figtree Light 300**
- Couleurs principales : **<font color="#6AA84F">#6AA84F</font>** et **<font color="#A8A54F">#A8A54F</font>**
- Couleurs secondaires : **<font color="#346F3A">#346F3A</font>** et **<font color="#C1E36A">#C1E36A</font>**

## Routage avec vue-router

[vue-router documentation](https://router.vuejs.org/)

### Installation

```sh
# Yarn :
yarn add vue-router@4

# NPM :
npm install vue-router@4
```

### Déclaration des routes

Dans le fichier *router.ts* :

```typescript
interface Route {
    path: string;
    name: string;
    component: any;
    props?: boolean;
}

const routes: Route[] = [
  { path: '/', name: "home", component: Home },
  { path: '/users', name: "users", component: UserDashboard },
  { path: '/workingTime/:userid', component: WorkingTime, props: true },
  { path: '/workingTime/:userid/:workingtimeid', component: WorkingTime, props: true },
  { path: '/workingtimes/:userid', component: Workingtimes },
  { path: '/chartManager/:userid', component: chartManager, props: true },
  { path: '/login', name: "login", component: Login },
  { path: '/register', name: "register", component: Register },
  { path: '/clock/:userID', component: Clock, props:true },
  { path: '/:pathMatch(.*)*', name: 'not-found', component: PageNotFound },
];

const router = createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

Dans le fichier *main.js :

```typescript
import router from './router';

createApp(App).use(router).mount('#app')
```

### Injection des composants dans App.vue

Dans le fichier *App.vue* :

La balise **```<router-view> </router-view>```** contiendra le composant rendu par le router

```vue
<script setup>
    import Navbar from "../src/components/Navbar.vue";
</script>

<template>
  <div id="navbar">
    <Navbar />
  </div>
  <div id="main">
    <Suspense>
      <router-view></router-view>
    </Suspense>
  </div>
</template>
```

### Paramètres d'URL

Soit la route précedemment déclarée **```{ path: '/workingTime/:userid', component: WorkingTime, props: true }```**, l'attribut props indique qu'il au router qu'une ou plusieurs props doivent etre extraitent du path.

Dans le composant **WorkingTime**

```javascript
export default {
  name: "WorkingTime",
  props: {
    userid: String,
    workingtimeid: String,
  },
  methods: {
    // [...]
  },
  async setup(props) {
    // [...]

    return {
        userid,
        workingtimeid
    }
  }
}

// userid and workingtimeid are accessibles anywhere within the current component
```

### Redirection

Avec vue-router la vue App.vue est dynamique, les redirections se font grace à la fonction **push** fournie par vue-router

```javascript
// ```{ path: '/workingTime/:userid', component: WorkingTime, props: true }```

router.push({ `/workingTime/${userid}` });
```

## Affichage des graphiques avec chart-js

### Pourquoi chart-js ?

Nous avons choisi d'utiliser chart-js pour 3 raisons majeures :

- Le sujet conseille chart-js
- Sur [npmjs.com](https://www.npmjs.com/), la librairie est parmi les plus populaires pour la génération de graphiques. Par extension, il y a donc beaucoup de ressources en ligne disponibles
- Certains membres du groupe ont déjà utilisé chart-js dans un autre contexte, la prise en main sera donc accelerée pour l'ensemble du groupe

### Installation

```sh
# Yarn :
yarn add chart.js

# NPM :
npm install chart.js
```

Dans le fichier *index.html* :

```html
<script src="path/to/chartjs/dist/chart.umd.js"></script>
```

### Utilisation

Créons un fichier *HoursPerMonth* contenu dans *@src/components/Charts/*

Il est nécessaire de 's'abonner' au composant chart-js afin de pouvoir rendre au client un graphique :

```javascript
import { BarElement, CategoryScale, Chart as ChartJS, Legend, LinearScale, Title, Tooltip } from 'chart.js'
import { Bar } from 'vue-chartjs'

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)
```

Les options permettent de gérer certains aspects du graphiques de manières simple, ici nous allons rendre le graphique responsive en quelques lignes :

```javascript
const options = {
    responsive: true,
    preserveAspectRatio: true
}
```

Nous créons une variable **data** qui permettra de constituer la donnée nécessaire à l'affichage : 

```javascript
const data = {
  labels: [
    'Monday',
    'Tuesday',
    'Wednesday',
    'Thursday',
    'Friday',
    'Saturday',
    'Sunday',
  ],
  datasets: [
    {
      label: 'Data One',
      backgroundColor: '#f87979',
      data: [40, 20, 12, 39, 10, 40, 39]
    }
  ]
}
```

Puis nous rendons le graphique de cette façon :

```javascript
export default {
  name: "HoursPerMonthChart",
  components: {
    Bar
  },
  data() {
    return {
      data: data,
      options: options
    };
  },
}

<template>
    <Bar :data="data" :options="options" />
</template>
```

## Intégration rapide avec Vuetify (Librairie CSS)

### Installation

```sh
# Yarn :
yarn add vuetify

# NPM :
npm install vuetify

# pnpm :
pnpm i vuetify
```

Dans le fichier *main.js* :

```javascript
// Vuetify
import 'vuetify/styles'
import { createVuetify } from 'vuetify'
import * as components from 'vuetify/components'
import * as directives from 'vuetify/directives'

const vuetify = createVuetify({
  components,
  directives,
  // ssr: true, -> If server-side rendering is needed, we don't
})

createApp(App).use(router).use(vuetify).mount('#app')
```

### Utilisation

[Documentation officielle](https://vuetifyjs.com/en/)

Une fois la librairie installée, les composants peuvent être utilisés directement dans les composants Vue :

```javascript
/* Submit button example */
<v-btn type="submit" block @click="submit" color="success" class="mt-2">Se connecter</v-btn>

/* Table display example */
<v-table v-if="renderComponent">
    <thead>
        <tr>
            <th class="text-left">
                Nom d'utilisateur
            </th>
            <th class="text-left">
                Email
            </th>
            <th class="text-right">
                Action (Supprimer)
            </th>
        </tr>
    </thead>
    <tbody v-if="userList" :key="userList">
        <tr v-for="item in userList" :key="item.id">
            <td>{{ item.username || "Non renseigné" }}</td>
            <td>{{ item.email || "Non renseigné" }}</td>
            <td>
                <v-container>
                    <v-row justify="end">
                        <v-col cols="auto" class="delete-icon">
                            <v-btn block @click="deleteUser(item.id)" class="m-2" id="working-time-form">
                                <svg-icon type="mdi" :path="pictures.delete"></svg-icon>
                            </v-btn>
                        </v-col>
                    </v-row>
                </v-container>
            </td>
        </tr>
    </tbody>
</v-table>

/* [...] */
```