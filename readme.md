COMANDOS NO CMD

- npm install express cors
- npm install sqlite3 sqlite
- npm install dotenv
- npm install bcrypt
- npm install jsonwebtoken

ARRUMANDOS PASTAS
- criando pasta public na pasta principal e adicionando os arquivos script.js, index.html, style.css
   |-public/
     |
     |- script.js
     |- index.html
     |- style.css

- criando pasta src  
- criando pastas database, middlewares, models, routes - dentro da pasta src
     -src/
       |
       |-database/
       |   |
       |   |-sqlite.js
       |
       |-middlewares/
       |  |
       |  |-auth.js
       |
       |-models/
       |  |
       |  |-Cliente.js
       |  |-Pedido.js
       |  |-Pizza.js
       |  |-Usuario.js
       | 
       |-routes/
         |
         |-index.js

- database - arquivo; sqlite.js
- middlewares - arquivo; auth.js
- models - arquivos; Cliente.js, Pedido.js, Pizza.js, Usuario.js
- criando arquivo sql não versionado, arquivo: pizzaria.db