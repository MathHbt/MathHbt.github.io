---
sidebar_position: 3
sidebar_label: "Mini-Project #1 - API (serveur)"
---

# Mini-Project #1 - API (Phoenix Framework - Elixir - Rest API)

**Objectif :** Réaliser une API REST pour gérer une pointeuse

**Contraintes :** Utilisation de Phoenix Framework - Routes et schémas imposés - Base de données PostgreSQL

## Initalisation du projet

Initalisation et accès au projet :
```
mix phx.new api --app api --no-html --no-assets
cd api/
```

Installation des dépendances :
```
mix deps.get
```

Execution des migrations / Mise en place de la base de données :
```
mix ecto.reset
```

Lancer le projet :
```
mix phx.server
```

Le projet est désormais accessible à l'url [http://localhost:4000](http://localhost:4000)

## Création des schémas

La commande permettant de créer un schéma JSON est la suivante :
```
mix phx.gen.json [context] [schema_name] [table_name] [field:type, ...]
```

```json
users = {
  "username": "string",
  "email": "string"
}

clocks = {
  "time": "datetime",
  "status": "boolean",
  "user": "users"
}

workingtimes = {
  "start": "datetime",
  "end": "datetime",
  "user": "users"
}
```

Nous générons les schémas ci-dessus avec les commandes suivantes :
```
mix phx.gen.json ApiWeb User users username:string email:string
mix phx.gen.json ApiWeb Clock clocks time:datetime status:boolean user:references:users
mix phx.gen.json ApiWeb WorkingTime workingtimes start:datetime end:datetime user:references:users
```

## Définition du router

Le router permet de rediriger les appels vers le bon Controller (Architechture MVC), la ligne suivante permet de 'bind' une route à un controller :

```elixir
resources "/workingtimes", WorkingTimesController, only: [:update, :delete]
```

Le fichier "api/lib/api_web/router.ex" :

```elixir
defmodule ApiWeb.Router do
  use ApiWeb, :router

  pipeline :api do
    plug :accepts, ["json"] # incoming type accepted
  end

  scope "/api", ApiWeb do
    pipe_through :api

    # Links /users/* calls to UsersController file
    resources "/users", UsersController, only: [:index, :show, :create, :update, :delete] 
    resources "/workingtimes", WorkingTimesController, only: [:update, :delete]
    resources "/workingtimes/:userID", WorkingTimesController, only: [:index, :show, :create]
    resources "/clocks/:userID", ClocksController, only: [:index, :create]
  end

  [...]
end

```

## Customisation des routes

Les routes par sont par défaut pré-générées, mais dans notre cas plusieurs modifications ont été apportées pour s'adapter au sujet.

### Filtrage du résultat pour matcher un paramètre d'URL

Pour l'insertion d'une clock il faut le user_id, nous avons surchargé la fonction de base index() qui permet de récuperer l'ensemble des Clock pour qu'elle puisse traiter les cas avec/sans userID.

Nous rajoutons au dessus notre fonction appelée si le parametre d'url userID est renseigné :

```elixir
# endpoint : {base_url}/clocks/:userID, method : GET
def index(conn, %{"userID" => userID}) do
  userID = String.to_integer(userID)
  clocks = WebApi.list_clocks()
  filtered_users = Enum.filter(clocks, fn clock -> clock.user == userID end)
  render(conn, :index, clocks: filtered_users)
end

# endpoint : {base_url}/clocks, method : GET
def index(conn, _params) do
  clocks = WebApi.list_clocks()
  render(conn, :index, clocks: clocks)
end
```

### Ajout d'un paramètre d'URL à un body pour insertion

Lors de l'insertion d'une Clock, on précise dans l'url le user_id associé, il faut rajouter ce champ au body pour pouvoir l'envoyer et l'insérer dans la base de données

Le body étant de type Map, on peut utiliser les fonctions fournies par cette classe :

```elixir
# Ajoute le champ "user": ${userID} à l'objet clocks_params
clocks_params = Map.put(clocks_params, "user", userID)
```

La fonction create() complète :

```elixir
def create(conn, %{"clocks" => clocks_params, "userID" => userID}) do
  clocks_params = Map.put(clocks_params, "user", userID)
  with {:ok, %Clocks{} = clocks} <- WebApi.create_clocks(clocks_params) do
    conn
    |> put_status(:created)
    |> put_resp_header("location", ~p"/api/clocks/#{clocks}")
    |> render(:show, clocks: clocks)
  end
end
```

## CORS (Cross Origin Resource Sharing)

  - [Tutoriel Phoenix officiel](https://hexdocs.pm/cors_plug/readme.html)

### Ajout de la dépendance cors_plug

Dans le fichier *mix.exs*

```elixir
def deps do
  # ...
  {:cors_plug, "~> 3.0"},
  #...
end
```

```sh
# Fetch new cors dependency
$ mix deps.get
```

### Mise en place de CORSPlug

Dans le fichier *your_app/lib/api_web/endpoint.ex*

```elixir
defmodule ApiWeb.Endpoint do
  use Phoenix.Endpoint, otp_app: :api

  # [...]

  plug CORSPlug # => Enables CORS server-side
  plug ApiWeb.Router
end
```

### Configuration de CORSPlug

```elixir
plug CORSPlug, origin: ["http://origin1.com", "http://origin2.com"]
```

## API references

### ✓ Users API

#### Récuperer tous les utilisateurs

##### Requête HTTP

```http
  GET http://localhost:4000/api/users
```

| Paramètre | Type technique | Type de paramètre | Description                                |
| :-------- | :------------- | :---------------- | :----------------------------------------- |
| email     | `string`       | URL               | **Optionnel**. L'email de l'utilisateur    |
| username  | `string`       | URL               | **Optionnel**. L'username de l'utilisateur |

##### Réponse HTTP

| Statut | Raison |
| :----- | :----- |
| 200 OK | Succès |

``` javascript
  HTTP/1.1 200
  Content-Type: application/json

{
  "data": [
      {
          "id": 34,
          "username": "test",
          "email": "test@test.fr"
      },
      {
          "id": 35,
          "username": "test",
          "email": "test@test.fr"
      }
  ]
}
```

#### Récuperer un utilisateur

##### Requête HTTP

```http
  GET  http://localhost:4000/api/users/:userID
```

| Paramètre | Type technique  | Type de paramètre | Description                                    |
| :-------- | :-------------- | :---------------- | :--------------------------------------------- |
| userID    | `integer - uid` | URL               | **Obligatoire**. Id technique de l'utilisateur |

##### Réponse HTTP

| Statut        | Raison            |
| :------------ | :---------------- |
| 200 OK        | Succès            |
| 404 Not Found | userID inexistant |

```javascript
  HTTP/1.1 200
  Content-Type: application/json

{
  "data": {
      "id": 34,
      "username": "test",
      "email": "test@test.fr"
  }
}
```

```javascript
  HTTP/1.1 404
  Content-Type: text/html
```

#### Créer un utilisateur

##### Requête HTTP

```http
  POST  http://localhost:4000/api/users
```

| Paramètre | Type technique | Type de paramètre | Description                                              |
| :-------- | :------------- | :---------------- | :------------------------------------------------------- |
| username  | `string`       | Body              | **Obligatoire**. Nom d'utilisateur du nouvel utilisateur |
| email     | `string`       | Body              | **Obligatoire**. Adresse email du nouvel utilisateur     |

##### Réponse HTTP

| Statut          | Raison                        |
| :-------------- | :---------------------------- |
| 201 Created     | Création réussie              |
| 400 Bad Request | username et/ou email manquant |

```javascript
  HTTP/1.1 201
  Content-Type: application/json

{
  "data": {
      "id": 37,
      "username": "test",
      "email": "romain.rosso@epitech.eu"
  }
}
```

```javascript
  HTTP/1.1 400
  Content-Type: text/html
```

#### Modifier un utilisateur

##### Requête HTTP

```http
  PUT  http://localhost:4000/api/users/:userID
```

| Paramètre | Type technique  | Type de paramètre | Description                                               |
| :-------- | :-------------- | :---------------- | :-------------------------------------------------------- |
| userID    | `integer - uid` | URL               | **Obligatoire**. Id technique de l'utilisateur à modifier |
| username  | `string`        | Body              | **Obligatoire**. Nouveau nom d'utilisateur                |
| email     | `string`        | Body              | **Obligatoire**. Nouvelle adresse email                   |

##### Réponse HTTP

| Statut          | Raison                        |
| :-------------- | :---------------------------- |
| 201 Created     | Création réussie              |
| 400 Bad Request | username et/ou email manquant |

```javascript
  HTTP/1.1 201
  Content-Type: application/json

{
  "data": {
      "id": 37,
      "username": "test",
      "email": "romain.rosso@epitech.eu"
  }
}
```

```javascript
  HTTP/1.1 400
  Content-Type: text/html
```


#### Supprimer un utilisateur

```http
  DELETE  http://localhost:4000/api/users/:userID
```

| Paramètre | Type technique  | Type de paramètre | Description                                                |
| :-------- | :-------------- | :---------------- | :--------------------------------------------------------- |
| userID    | `integer - uid` | URL               | **Obligatoire**. Id technique de l'utilisateur à supprimer |

##### Réponse HTTP

| Statut          | Raison           |
| :-------------- | :--------------- |
| 200 OK          | Création réussie |
| 400 Bad Request | Body manquant    |

```javascript
  HTTP/1.1 200
  Content-Type: application/json

{
  "data": {
      "id": 37,
      "username": "test",
      "email": "romain.rosso@epitech.eu"
  }
}
```

```javascript
  HTTP/1.1 400
  Content-Type: text/html
```

### ✓ WorkingTime API

#### Récuperer tous les workingtimes

```http
  GET http://localhost:4000/api/workingtimes/:userID?start=XXX&end=YYY
```

| Paramètre | Type technique                     | Type de paramètre | Description                                                       |
| :-------- | :--------------------------------- | :---------------- | :---------------------------------------------------------------- |
| userID    | `string`                           | URL               | **Obligatoire**. L'identifiant technique de l'utilisateur associé |
| start     | `datetime - (format : dd-MM-yyyy)` | URL               | **Optionnel**. La date à partir de laquelle chercher              |
| end       | `datetime - (format: dd-MM-yyyy)`  | URL               | **Optionnel**. La date avant laquelle chercher                    |

#### Récuperer un workingtime

```http
  GET  http://localhost:4000/api/workingtimes/:userID/:id
```

| Paramètre | Type technique  | Type de paramètre | Description                                              |
| :-------- | :-------------- | :---------------- | :------------------------------------------------------- |
| userID    | `integer - uid` | URL               | **Obligatoire**. Id technique de l'utilisateur associé   |
| id        | `integer - uid` | URL               | **Obligatoire**. Id technique du workingtime à récuperer |

#### Créer un workingtime

```http
  POST  http://localhost:4000/api/workingtimes/:userID
```

| Paramètre | Type technique                     | Type de paramètre | Description                                                     |
| :-------- | :--------------------------------- | :---------------- | :-------------------------------------------------------------- |
| userID    | `string`                           | URL               | **Obligatoire**. Identifiant technique de l'utilisateur associé |
| start     | `datetime - (format : dd-MM-yyyy)` | Body              | **Obligatoire**. Date de début                                  |
| end       | `datetime - (format : dd-MM-yyyy)` | Body              | **Obligatoire**. Date de fin                                    |

#### Modifier un workingtime

```http
  PUT  http://localhost:4000/api/workingtimes/:id
```

| Paramètre | Type technique                     | Type de paramètre | Description                                             |
| :-------- | :--------------------------------- | :---------------- | :------------------------------------------------------ |
| id        | `integer - uid`                    | URL               | **Obligatoire**. Id technique du workingtime à modifier |
| start     | `datetime - (format : dd-MM-yyyy)` | Body              | **Obligatoire**. Date de début                          |
| end       | `datetime - (format : dd-MM-yyyy)` | Body              | **Obligatoire**. Date de fin                            |
| user      | `integer - uid`                    | Body              | **Obligatoire**. Utilisateur associé                    |

#### Supprimer un workingtime

```http
  DELETE  http://localhost:4000/api/workingtimes/:id
```

| Paramètre | Type technique  | Type de paramètre | Description                                              |
| :-------- | :-------------- | :---------------- | :------------------------------------------------------- |
| id        | `integer - uid` | URL               | **Obligatoire**. Id technique du workingtime à supprimer |

### ✓ Clock API

#### Récuperer les clocks

```http
  GET http://localhost:4000/api/clocks/:userID
```

| Paramètre | Type technique | Type de paramètre | Description                                                       |
| :-------- | :------------- | :---------------- | :---------------------------------------------------------------- |
| userID    | `string`       | URL               | **Obligatoire**. L'identifiant technique de l'utilisateur associé |

#### Créer une clock

```http
  POST http://localhost:4000/api/clocks/:userID
```

| Paramètre | Type technique                       | Type de paramètre | Description                                                       |
| :-------- | :----------------------------------- | :---------------- | :---------------------------------------------------------------- |
| userID    | `string`                             | URL               | **Obligatoire**. L'identifiant technique de l'utilisateur associé |
| time      | `datetime - (format : "dd-MM-yyyy")` | Body              | **Obligatoire**. L'heure de la clock                              |
| status    | `boolean`                            | Body              | **Obligatoire**. Le status de la clock                            |