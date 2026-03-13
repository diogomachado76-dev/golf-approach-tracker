# CLAUDE.md

## Visão Geral do Projeto

**Golf Training Tracker Pro** é um aplicativo web de página única para acompanhar sessões de treino de approach no golfe. O jogador registra tacadas de approach (acertou o green, errou, acertou o buraco), faz a fase de putting, e depois visualiza os resultados da sessão com estatísticas e gráficos históricos. Toda a interface é em **português brasileiro**.

## Estrutura do Repositório

```
golf-approach-tracker/
├── index.html          # Aplicação principal (HTML + CSS + JS, ~1260 linhas)
├── index (5).html      # Versão antiga / backup (upload via GitHub)
├── index (6).html      # Versão antiga / backup (upload via GitHub)
└── CLAUDE.md           # Este arquivo
```

Este projeto **não tem dependências locais nem processo de build**. Tudo está em um único arquivo `index.html` com CSS e JavaScript inline. A única dependência externa é o **Chart.js v2**, carregado via CDN (tag `<script>`).

## Arquitetura

### Aplicação em Arquivo Único (`index.html`)

O app é organizado em três seções dentro de um único arquivo HTML:

1. **`<style>`** — Todo o CSS (~690 linhas). Usa gradientes CSS, layouts com flexbox/grid e design responsivo. Sem pré-processador ou framework.
2. **HTML body** — Seções semânticas para configuração, rastreamento, resultados e modal de histórico.
3. **`<script>`** — Todo o JavaScript (~560 linhas). JS puro (vanilla), sem framework.

### Fluxo da Aplicação

1. **Fase de Configuração** (`#configSection`) — O usuário define data, número de rodadas, bolas por rodada, distância (jardas) e tipo de taco.
2. **Fase de Approach** (`#trackingSection`) — O usuário registra cada tacada como:
   - `green` — Bola caiu no green
   - `miss` — Bola errou o green
   - `hole` — Bola entrou no buraco
3. **Fase de Putting** (`#puttingSection`) — Para cada green acertado, o usuário registra se o putt foi convertido ou errado.
4. **Fase de Resultados** (`#resultsSection`) — Resumo por rodada com % de green e % de putting.
5. **Modal de Histórico** (`#historyModal`) — Estatísticas agregadas, gráfico de linha (% green ao longo do tempo) e gráfico de barras (média de % green por taco).

### Gerenciamento de Estado

- O objeto global `session` armazena todo o estado em tempo de execução (rodada atual, tacadas, contadores, fase).
- A propriedade `phase` alterna entre `'approach'` e `'putting'`.

### Persistência de Dados

- Chave no **localStorage**: `golfSessions` — Array JSON com todos os objetos de sessão salvos.
- Cada sessão armazena: data, timestamp, distância, taco, contadores por rodada e dados detalhados das rodadas.

### Dependências Externas

- **Chart.js v2** via CDN (`https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.9.4/Chart.min.js`) — Usado para gráficos de linha e barras no modal de histórico.

## Referência das Funções Principais

| Função | Descrição |
|---|---|
| `startSession()` | Valida os campos de configuração e inicializa uma nova sessão de treino |
| `startNewRound()` | Reseta a interface e o estado para uma nova rodada dentro da sessão |
| `recordShot(type)` | Registra uma tacada de approach (`green`, `miss` ou `hole`) |
| `startPuttingPhase()` | Faz a transição do approach para o putting; pula se nenhum green foi acertado |
| `recordPutt(made)` | Registra o resultado de um putt (booleano: acertou/errou) |
| `finishRound()` | Salva os dados da rodada; avança para a próxima ou exibe resultados |
| `undoLastAction()` | Desfaz o último registro de tacada ou putt |
| `showResults()` | Exibe o resumo por rodada e aciona `saveSession()` |
| `saveSession()` | Persiste os dados da sessão no localStorage |
| `showHistory()` | Abre o modal de histórico com estatísticas, gráficos e lista de sessões |
| `clearHistory()` | Limpa todas as sessões salvas do localStorage (com confirmação) |

## Fluxo de Desenvolvimento

### Executando Localmente

Não é necessário nenhum passo de build. Basta abrir o `index.html` diretamente no navegador:

```bash
# Usando o servidor embutido do Python:
python3 -m http.server 8000

# Ou simplesmente abrir o arquivo:
open index.html        # macOS
xdg-open index.html    # Linux
```

### Testes

Não há testes automatizados. Todos os testes são feitos manualmente pelo navegador.

### Fazendo Alterações

- Todas as mudanças de código são feitas no `index.html`. Não existe sistema de módulos nem pipeline de build.
- O CSS fica no topo dentro de um bloco `<style>`; o JS fica na parte inferior dentro de um bloco `<script>`.
- Os arquivos `index (5).html` e `index (6).html` são versões antigas e em geral não devem ser modificados.

## Convenções

- **Idioma**: Todo o texto da interface é em português brasileiro. Manter todas as strings voltadas ao usuário em pt-BR.
- **Estilização**: CSS inline usando gradientes customizados (`linear-gradient(135deg, #667eea, #764ba2)` como tema principal). Sem framework CSS.
- **JavaScript**: Apenas JS puro (vanilla). Sem transpilação. Manipulação do DOM via `document.getElementById()`. Sem módulos ou imports.
- **Formato dos dados**: Todas as medidas em jardas. Tipos de taco incluem variantes de wedge (PW, GW, SW, LW) e ferros (9i até 5i).
- **Chart.js**: Usa a API da v2 (sintaxe legada `xAxes`/`yAxes`, não v3+). Manter o código de gráficos compatível com a v2.

## Armadilhas Comuns

- As instâncias dos gráficos (`lineChart`, `barChart`) devem ser destruídas antes de recriá-las para evitar erros de reutilização do canvas.
- O objeto `session` é global e mutável — tenha cuidado com o estado ao adicionar funcionalidades.
- O `localStorage` não tem sistema de migração; mudanças no schema de `golfSessions` podem quebrar dados existentes do usuário.
