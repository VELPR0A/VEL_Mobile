# Guia para o Cursor - VEL Mobile

Este documento explica o funcionamento atual do `VEL_Mobile` para orientar alteracoes futuras no Cursor.

## Visao geral

Apesar do nome `VEL_Mobile`, este projeto nao e React Native nem Expo. Ele e um app React com Vite configurado como PWA, voltado para a experiencia do entregador.

O projeto possui telas para:

- login do entregador;
- recuperacao de senha;
- codigo de seguranca;
- lista de entregas;
- detalhe de uma entrega;
- gerenciamento de gastos;
- configuracoes de interface;
- configuracoes de usuario.

No estado atual, o app e majoritariamente estatico. Nao ha chamadas para a `VEL_API`; a unica persistencia real encontrada e o controle financeiro usando `localStorage`.

## Stack principal

- React 18
- Vite 5
- React Router DOM
- styled-components
- CSS Modules
- react-icons
- vite-plugin-pwa

Scripts em `package.json`:

```bash
npm run dev       # servidor local Vite
npm run build     # build de producao
npm run preview   # preview da build
npm run lint      # ESLint
```

## Entrada da aplicacao

Arquivos principais:

```txt
index.html
src/main.jsx
```

`src/main.jsx` cria o React root e registra as rotas usando `BrowserRouter`, `Routes` e `Route`.

## Rotas

Rotas atuais:

```txt
/                         -> Login
/recuperarsenha           -> Recuperar senha
/codigoseguranca          -> Codigo de seguranca
/entregas                 -> Lista de entregas
/entregaespecifica        -> Detalhe de entrega
/gerenciamentogastos      -> Controle financeiro local
/configuracaointerface    -> Configuracoes gerais/interface
/configuracaousuario      -> Configuracoes do usuario
```

Nao existem rotas protegidas. Qualquer tela pode ser aberta diretamente pelo navegador.

## PWA

A configuracao PWA fica em:

```txt
vite.config.js
```

O plugin usado e:

```js
VitePWA
```

Configuracao principal:

```txt
name: VEL para entregadores
short_name: VEL
description: Aplicacao VEL para entregadores
theme_color: #ffffff
icons: pwa-192-192.png, pwa-512-512.png
```

Ao rodar `npm run build`, o projeto gera arquivos de PWA/service worker em `dist`, como:

```txt
manifest.webmanifest
sw.js
registerSW.js
```

Atencao: `vite.config.js` importa `IconsManifest` de `react-icons`, mas esse import nao e usado.

## Estrutura de pastas

```txt
src/
  main.jsx
  Paginas/
    Login/
    RecuperarSenha/
    CodigoSeguranca/
    Entregas/
    EntregaEspecifica/
    GerenciamentoGastos/
    ConfigInterfac/
    ConfigUsuario/
  components/
    Header/
    Footer/
    TelaEntregas/
    TelaEntregaEspecifica/
    Pedido/
    CardEndereco/
    CardPagamento/
    Form/
    Grid/
    GridItem/
    Resume/
    ResumeItem/
    CodeInputs/
  assets/
public/
  android/
  ios/
  windows11/
  pwa-192-192.png
  pwa-512-512.png
```

## Fluxos atuais

### Login

Arquivo:

```txt
src/Paginas/Login/app.jsx
```

O formulario pede email e senha, mas atualmente nao autentica com backend.

Fluxo atual:

1. O usuario preenche email e senha.
2. O submit executa `enviaForm`.
3. O codigo le os campos com `document.querySelector`.
4. Os valores lidos nao sao usados.
5. O app navega diretamente para `/entregas`.

Nao ha token, sessao, `localStorage.User`, validacao real ou chamada para `/loginentregador/usuario`.

### Recuperacao de senha

Arquivo:

```txt
src/Paginas/RecuperarSenha/app.jsx
```

O formulario pede email e navega para `/codigoseguranca`. Nao ha envio real de email nem chamada para API.

### Codigo de seguranca

Arquivos:

```txt
src/Paginas/CodigoSeguranca/app.jsx
src/components/CodeInputs/index.jsx
```

Renderiza seis inputs numericos. O componente controla os valores localmente com `useState`.

O botao "Reenviar Codigo" apenas dispara um `alert` com o codigo digitado. Nao ha validacao real.

### Lista de entregas

Arquivos:

```txt
src/Paginas/Entregas/app.jsx
src/components/TelaEntregas/index.jsx
src/components/Pedido/index.jsx
```

Renderiza uma lista de componentes `Pedido`, mas os dados sao fixos.

Somente o primeiro `Pedido` esta envolvido em um link que navega para:

```txt
/entregaespecifica
```

Nao ha busca de pedidos na API.

### Detalhe de entrega

Arquivos:

```txt
src/Paginas/EntregaEspecifica/app.jsx
src/components/TelaEntregaEspecifica/index.jsx
src/components/CardEndereco/index.jsx
src/components/CardPagamento/index.jsx
```

Mostra:

- header;
- endereco fixo;
- nome fixo do cliente;
- tempo fixo;
- iframe fixo do Google Maps;
- link fixo para o Google Maps;
- pagamento fixo.

Nao ha dados vindos da rota, query string, estado global ou API.

### Gerenciamento de gastos

Arquivo principal:

```txt
src/Paginas/GerenciamentoGastos/App.jsx
```

Componentes:

```txt
src/components/HeaderControle/
src/components/Resume/
src/components/ResumeItem/
src/components/Form/
src/components/Grid/
src/components/GridItem/
```

Este e o modulo mais funcional do app.

Estado persistido:

```txt
localStorage.transactions
```

Fluxo:

1. Ao abrir a tela, le `localStorage.transactions`.
2. Se existir, popula `transactionsList`.
3. Calcula entradas, saidas e total com `useEffect`.
4. O formulario adiciona uma transacao.
5. A transacao e salva no estado e em `localStorage`.
6. A tabela permite deletar transacoes e atualiza o `localStorage`.

Formato atual de uma transacao:

```js
{
  id: number,
  desc: string,
  amount: string,
  expense: boolean
}
```

Pontos de atencao:

- `id` e gerado com `Math.round(Math.random() * 1000)`, entao pode colidir.
- Os radios de entrada/saida alternam estado com `setExpense(!isExpense)`. O ideal e setar explicitamente `false` para entrada e `true` para saida.
- Os valores ficam apenas no navegador do usuario; nao ha sincronizacao com backend.

### Configuracoes de interface

Arquivo:

```txt
src/Paginas/ConfigInterfac/app.jsx
```

Mostra configuracoes visuais:

- notificacoes;
- modo escuro;
- idioma;
- politica de privacidade;
- termos de uso;
- central de ajuda;
- sobre;
- sair.

No estado atual, os toggles e opcoes nao persistem nem executam regras reais.

### Configuracoes de usuario

Arquivo:

```txt
src/Paginas/ConfigUsuario/app.jsx
```

Mostra dados fixos:

- nome;
- email;
- senha;
- telefone;
- conta bancaria;
- CPF;
- CNH.

Os icones de edicao sao visuais. Nao ha formulario funcional, persistencia ou API.

### Footer / navegacao inferior

Arquivo:

```txt
src/components/Footer/index.jsx
```

Navega para:

```txt
/gerenciamentogastos
/entregas
/configuracaointerface
```

## Integracao com API

No estado atual, nao foram encontradas chamadas:

```txt
fetch(...)
axios
https://vel-tnpo.onrender.com
localhost
```

Ou seja: o app mobile ainda nao esta integrado com a `VEL_API`.

Endpoints provaveis para uma integracao futura:

```txt
POST /loginentregador/usuario
GET  /pedido/info/{id_cpf}
GET  /entregador/{idCpf}/comandas
GET  /entregador/id/{id}
```

A API atual tambem possui dados de entregador, comandas e pedidos que podem sustentar este PWA.

## Estilos

O projeto mistura:

- styled-components em arquivos `.ts` e `styles.js`;
- CSS Modules em `style.module.css`;
- estilos inline.

Ao editar, siga o padrao do componente atual:

- se o componente usa `style.module.css`, continue com CSS Module;
- se usa `styled-components`, continue com styled-components.

## Assets

Ha assets em:

```txt
src/assets/
public/
```

`public/` possui muitos icones gerados para PWA em Android, iOS e Windows.

## Validacao executada

As dependencias foram instaladas com:

```bash
npm install
```

O npm reportou vulnerabilidades:

```txt
22 vulnerabilities
```

O build foi executado e passou:

```bash
npm run build
```

Resultado:

```txt
vite build OK
PWA generated
```

O lint foi executado e falhou:

```bash
npm run lint
```

Foram encontrados 39 erros.

## Problemas criticos identificados

### 1. Mobile nao esta integrado ao backend

Nao ha login real, busca de entregas, detalhe dinamico de entrega, perfil real ou sincronizacao financeira.

### 2. Login apenas navega

`src/Paginas/Login/app.jsx` le email/senha, mas nao usa esses valores. O submit apenas navega para `/entregas`.

### 3. Recuperacao de senha e codigo sao apenas interface

Nao existe chamada de envio de codigo, verificacao de codigo ou reset de senha.

### 4. Entregas sao mockadas

`TelaEntregas` renderiza varios componentes `Pedido` estaticos. O detalhe de entrega tambem usa dados fixos.

### 5. Mapa e rota sao fixos

`CardEndereco` tem iframe e link fixos do Google Maps. Nao usa endereco real da entrega.

### 6. Financeiro usa apenas localStorage

O controle financeiro funciona localmente, mas nao sincroniza com a API. Se limpar o navegador, os dados somem.

### 7. Possivel bug nos radios de entrada/saida

O formulario financeiro usa `setExpense(!isExpense)` para os dois radios. Isso pode alternar para um estado incorreto. Preferir:

```js
onChange={() => setExpense(false)} // entrada
onChange={() => setExpense(true)}  // saida
```

### 8. IDs financeiros podem colidir

IDs sao gerados com random de 0 a 1000. Para dados locais, usar `crypto.randomUUID()` seria mais seguro.

### 9. Lint falha

Principais tipos de erro:

- imports `React` nao usados;
- props sem `prop-types`;
- variaveis nao usadas;
- `navigate` declarado e nao usado;
- import `IconsManifest` nao usado em `vite.config.js`.

### 10. Encoding quebrado

Varios textos aparecem como:

```txt
CÃ³digo
ConfiguraÃ§Ãµes
SÃ£o Paulo
descriÃ§Ã£o
```

Padronizar arquivos para UTF-8 antes de mexer muito em copy/textos.

### 11. Uso de `document.querySelector`

Login e recuperacao de senha usam `document.querySelector` para ler inputs. Em React, prefira `useState`, `useRef` ou `FormData`.

### 12. `node_modules` e `dist`

`node_modules` e `dist` sao gerados localmente e devem continuar ignorados pelo git.

## Prioridades recomendadas

1. Integrar login com `POST /loginentregador/usuario`.
2. Guardar identificador do entregador autenticado em estado/localStorage.
3. Criar protecao simples de rota para telas internas.
4. Buscar entregas reais pela API.
5. Passar o pedido selecionado para a tela de detalhe.
6. Remover dados fixos de entrega, mapa, pagamento e perfil.
7. Corrigir radios e IDs do controle financeiro.
8. Decidir se financeiro deve continuar local ou sincronizar com backend.
9. Corrigir encoding para UTF-8.
10. Limpar erros principais do lint.

## Comandos uteis para investigacao

Listar rotas:

```bash
rg "Route path" src/main.jsx
```

Encontrar persistencia local:

```bash
rg "localStorage|sessionStorage" src
```

Encontrar chamadas de API:

```bash
rg "fetch\\(|axios|onrender|localhost" src
```

Encontrar textos/inputs lidos via DOM:

```bash
rg "document\\.querySelector|getElementById" src
```

Encontrar navegacao:

```bash
rg "useNavigate|navigate\\(" src
```

## Estado atual resumido

O `VEL_Mobile` e um PWA React para entregadores. Ele compila e gera PWA, mas ainda nao esta conectado ao backend. As telas de login, recuperacao, entregas, detalhe de entrega e perfil sao principalmente prototipos estaticos. O controle financeiro e funcional localmente via `localStorage`, mas nao sincroniza com a API. O projeto tem debitos de lint, encoding e alguns bugs pequenos de estado.
