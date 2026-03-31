# COMANDOS NO CMD

Execute os comandos abaixo para instalar as dependências do projeto:

```
npm install express cors
npm install sqlite3 sqlite
npm install dotenv
npm install bcrypt
npm install jsonwebtoken
```

---

# ARRUMANDO PASTAS

Estrutura mínima de diretórios e arquivos usados neste projeto:

- Crie a pasta `public/` na raiz e adicione os arquivos estáticos:

```
public/
  script.js
  index.html
  style.css
```

- Crie a pasta `src/` com as subpastas abaixo:

```
src/
  database/
    sqlite.js
  middlewares/
    auth.js
  models/
    Cliente.js
    Pedido.js
    Pizza.js
    Usuario.js
    
  routes/
    index.js
```

- Arquivos exemplares:
  - `src/database/sqlite.js`
  - `src/middlewares/auth.js`
  - `src/models/Cliente.js`, `src/models/Pedido.js`, `src/models/Pizza.js`, `src/models/Usuario.js`

- Arquivo do banco (não versionado): `pizzaria.db`

# EXPLICANDO ARQUIVOS

- `index.html` — arquivo front-end (página estática servida em `/`).
- `public/script.js` — lógica do front-end (requisições para a API, manipulação do DOM).
- `public/style.css` — estilos do front-end.
- `index.js` — ponto de entrada do servidor: carrega variáveis de ambiente, configura middlewares, aguarda `ready` do banco e monta as rotas em `/api`.
- `package.json` — lista de dependências e scripts úteis (`start`, `dev`, `seed`).
- `seed.js` — script para popular o banco com dados iniciais .
- `pizzaria.db` — arquivo SQLite gerado por `sql.js` (não versionar).

- `src/database/sqlite.js` — inicializa `sql.js`, cria o arquivo de DB (`DB_PATH`) se necessário, cria as tabelas e exporta `ready` (Promise) e os helpers `query`, `get`, `run`, `salvar`.
  - Observação: `run()` grava alterações e chama `salvar()` para persistir o arquivo `pizzaria.db`.

- `src/middlewares/auth.js` — middleware JWT: extrai `Authorization: Bearer <token>`, valida com `process.env.JWT_SECRET` e popula `req.usuario` com o payload.

- `src/models/Usuario.js` — modelo de usuários. Métodos: `findAll`, `findByEmail`, `findById`, `create`, `update`, `delete`, `verificarSenha`. Usa `bcryptjs` para hashing e formata a saída (ex.: `ativo` → booleano).

- `src/models/Pizza.js` — modelo de pizzas: campos como `nome`, `ingredientes`, `precos` (JSON). Implementa operações CRUD via os helpers do DB.

- `src/models/Cliente.js` — modelo de clientes: CRUD básico e busca por critérios (ex.: `busca` query em rotas).

- `src/models/Pedido.js` — modelo de pedidos: cria pedidos com itens, calcula totais, expõe `create`, `findAll` (aceita filtros), `findById`, `updateStatus`, `delete`.

- `src/routes/index.js` — define as rotas REST principais (`/auth/login`, `/pizzas`, `/clientes`, `/pedidos`, `/usuarios`), aplica `auth` middleware nas rotas protegidas e checa `req.usuario.perfil` para permissões (ex.: somente `Administrador` pode criar/ler `/usuarios`).

- `readme.md` — este arquivo: instruções rápidas sobre instalação e estrutura do projeto.

# EPLICANDO SITE

### **_TELA DE LOGIN_**

## Modelos (models)

---
Abaixo estão os modelos presentes em src/models/ com uma breve descrição do que cada um expõe e quais operações é esperado que realizem.
---

### Usuario (src/models/Usuario.js)
---
- Campos típicos:
  - id / _id
  - nome
  - email
  - senha (hash)
  - perfil (e.g. "Administrador", "Garcom", "Atendente")
  - ativo (boolean)
- O que faz / métodos esperados:
  - findAll(filters) — listar usuários (com paginação / filtros).
  - findById(id) — retornar usuário por id.
  - findByEmail(email) — buscar usuário pelo email (usado no login).
  - create(data) — criar novo usuário; deve hashear a senha com bcrypt antes de salvar.
  - update(id, data) — atualizar campos (não retornar hash de senha nas respostas).
  - delete(id) — desativar ou remover usuário.
  - verificarSenha(senha, hash) — comparar senha com hash (bcrypt).
- Observações:
  - Não devolver o campo senha nas respostas da API.
  - Perfis controlam permissões nas rotas (ex.: só Administrador pode gerenciar usuários).
---

### Pizza (src/models/Pizza.js)
---
- Campos típicos:
  - id / _id
  - nome
  - descricao / ingredientes
  - precos (objeto ou JSON com chaves: pequena, media, grande)
  - categoria (opcional)
  - disponivel (boolean)
- O que faz / métodos esperados:
  - findAll(filters) — listar pizzas (filtro por nome, categoria, disponibilidade).
  - findById(id) — retornar pizza.
  - create(data) — criar pizza (armazenar preços como JSON).
  - update(id, data) — atualizar pizza.
  - delete(id) — remover pizza.
- Observações:
  - precos deve ser um objeto JSON para facilitar diferentes tamanhos/preços.
  - Validar campos obrigatórios (nome, pelo menos um preço).
---
### Cliente (src/models/Cliente.js)
---
- Campos típicos:
  - id / _id
  - nome
  - telefone
  - endereco (rua, numero, complemento, bairro, cidade)
  - observacoes
- O que faz / métodos esperados:
  - findAll({ busca }) — listar clientes; permitir busca por nome/telefone.
  - findById(id) — retornar cliente.
  - create(data) — criar cliente.
  - update(id, data) — atualizar cliente.
  - delete(id) — remover ou marcar como inativo.
- Observações:
  - Permitir pesquisa incremental no front-end (debounce) utilizando a rota de listagem com query string.
---

### Pedido (src/models/Pedido.js)

---
- Campos típicos:
  - id / _id
  - clienteId (ou dados básicos do cliente)
  - mesa (número, opcional)
  - itens: [ { pizzaId, nome, tamanho, preco, quantidade } ]
  - subtotal, taxa_entrega, total
  - forma_pagamento (dinheiro, cartão, pix, etc.)
  - troco (quando forma_pagamento == dinheiro)
  - status (recebido, em_preparo, saiu_entrega, entregue, cancelado)
  - observacoes
  - criado_em, atualizado_em
- O que faz / métodos esperados:
  - create(pedidoData) — criar pedido; calcular subtotal e total (somar itens, adicionar taxa).
  - findAll(filters) — listar pedidos (filtros por status, cliente, período, mesa).
  - findById(id) — retornar pedido com itens e dados do cliente.
  - updateStatus(id, novoStatus) — alterar status do pedido.
  - update(id, data) — editar pedido (se permitido).
  - delete(id) — cancelar ou remover pedido.
- Observações:
  - Ao criar pedido, gravar snapshot dos itens (nome, tamanho, preço) para manter histórico.
  - Validar estoque/preços conforme necessário (se houver).
  - Permitir fechamento de mesas (conjunto de pedidos) e gerar totais para pagamento.

---
