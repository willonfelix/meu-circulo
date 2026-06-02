# Especificação do App — Meu Círculo

## 1. Visão Geral

**Meu Círculo** é um PWA (*Progressive Web App*) para gerenciamento de círculo social. Permite cadastrar amigos, controlar aniversários, preferências de presente, nível de proximidade, e registrar eventos/festas com controle de orçamento e convidados.

**Público-alvo**: Pessoas que desejam organizar e manter sua vida social, com lembretes de aniversários e planejamento de eventos.

**Plataforma**: Navegador (desktop e mobile), instalável como app na tela inicial.

---

## 2. Funcionalidades

### 2.1 Amigos
| Funcionalidade | Descrição |
|---|---|
| Cadastro | Nome, nascimento, gênero, estilo/personalidade, preferências de presente |
| Filhos | Número e gênero dos filhos |
| Último Contato | Data do último contato para monitoramento |
| Proximidade | Classificação: "Melhor Amigo", "Próximo" ou "Conhecido" |
| Editar | Atualizar qualquer campo do amigo |
| Excluir | Remover amigo com confirmação |
| Alerta Aniversário | Banner automático para aniversários nos próximos 7 dias (com "É Hoje!" animado) |

### 2.2 Festas
| Funcionalidade | Descrição |
|---|---|
| Cadastro | Nome do evento, tipo, status, data, horário, local, orçamento |
| Convidados | Selecionar múltiplos amigos cadastrados como convidados |
| Observações | Campo livre para notas (ideias de presente, lembranças, etc.) |
| Editar | Atualizar qualquer campo da festa |
| Excluir | Remover festa com confirmação |
| Ordenação | Festas listadas por data crescente |

### 2.3 Dashboard (Sempre Visível)
- **Card "Amigos"**: Total de amigos cadastrados
- **Card "Filhos (Total)"**: Soma de filhos de todos os amigos
- **Card "Festas este Mês"**: Quantidade de eventos no mês corrente
- **Gráfico "Gênero"**: Doughnut — distribuição por gênero
- **Gráfico "Faixa Etária"**: Barras — faixas 18-25, 26-35, 36-50, 50+
- **Gráfico "Eventos/Mês"**: Barras — eventos por mês (Jan–Dez)

### 2.4 PWA
- Instalável na tela inicial (manifest.json + service worker)
- Funciona offline (cache via service worker)
- Responsivo (mobile-first com Tailwind CSS)

---

## 3. Arquitetura

### 3.1 Estrutura de Arquivos
```
/
├── index.html          — Aplicação completa (HTML + CSS + JS)
├── manifest.json       — Configuração PWA
├── ws.js               — Service Worker (cache offline)
├── icon-192.png        — Ícone 192x192
├── icon-512.png        — Ícone 512x512
├── 1f465.svg           — Ícone auxiliar (não utilizado no app)
├── Leia-me-MD          — Instruções iniciais de deploy
└── ESPECIFICACAO.md    — Este documento
```

### 3.2 Dependências Externas
| Biblioteca | CDN | Uso |
|---|---|---|
| Tailwind CSS | `cdn.tailwindcss.com` | Estilização utilitária |
| Chart.js | `cdn.jsdelivr.net/npm/chart.js` | Gráficos do dashboard |
| Font Awesome | `cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0` | Ícones |

### 3.3 Armazenamento
**localStorage** — duas chaves:

#### `meusAmigosPWA`
```json
[
  {
    "id": 1717000000000,
    "nome": "João Silva",
    "nascimento": "1990-05-15",
    "genero": "Masculino",
    "estilo": "O Aventuroso",
    "preferencias": "Livros de ficção, cerveja artesanal",
    "numFilhos": 2,
    "generoFilhos": "1 menino, 1 menina",
    "ultimoContato": "2026-05-01",
    "proximidade": "Melhor Amigo"
  }
]
```

#### `festasPWA`
```json
[
  {
    "id": 1717000000001,
    "nome": "Aniversário do João",
    "tipo": "Aniversário",
    "status": "Confirmada",
    "data": "2026-05-15",
    "horario": "19:00",
    "local": "Casa do João",
    "orcamento": 200.00,
    "convidados": [1717000000000, 1717000000002],
    "observacoes": "Levar vinho"
  }
]
```

---

## 4. Interface do Usuário

### 4.1 Header
- Título "Meu Círculo" com ícone
- Botão "Novo Amigo" (sempre visível, abre modal de amigo)

### 4.2 Abas de Navegação
- **Amigos** (ativa por padrão) — borda inferior azul
- **Festas** — borda inferior rosa quando ativa

### 4.3 Seção Amigos
- Lista de cards ordenados por proximidade do aniversário
- Cada card: nome, idade, estilo, proximidade, preferências, filhos, último contato
- Badge "É Hoje!" (pulsante) ou "Em N dias" para aniversários próximos
- Borda lateral vermelha para "Melhor Amigo", azul para demais

### 4.4 Seção Festas
- Header com título "Suas Festas" + botão "Nova Festa"
- Lista de cards ordenados por data crescente
- Cada card: nome, tipo, status (com cor e ícone), data, horário, local, orçamento, convidados, observações
- Borda lateral rosa

### 4.5 Modais
Ambos os modais são `bottom-sheet` em mobile (arredondados em cima) e `centered` em desktop.

#### Modal Amigo
Campos: Nome*, Nascimento*, Gênero, Estilo, Preferências, Nº Filhos, Gênero Filhos, Último Contato, Proximidade

#### Modal Festa
Campos: Nome do Evento*, Tipo, Status, Data*, Horário, Local, Orçamento, Convidados (checkboxes), Observações

---

## 5. Regras de Negócio

### 5.1 Amigos
- Nome e Nascimento são obrigatórios
- ID gerado via `Date.now()` se novo, ou reutilizado na edição
- Lista ordenada por quem faz aniversário primeiro (dias até o próximo aniversário)
- Alerta de aniversário: calcula diferença em dias entre hoje e o próximo aniversário

### 5.2 Festas
- Nome e Data são obrigatórios
- Convidados são armazenados como array de IDs de amigos
- Se um amigo for excluído, o nome exibido na festa será "Desconhecido"
- Orçamento é opcional, exibido como `R$ 0,00` se vazio
- Status controla cor e ícone: Planejada (📋 amarelo), Confirmada (✅ verde), Realizada (🎊 cinza)

### 5.3 Dashboard
- "Festas este Mês" conta apenas eventos da lista `festasPWA` cujo mês da data coincida com o mês atual
- Gráfico "Eventos/Mês" usa dados de `festasPWA`

---

## 6. API JavaScript

### 6.1 Núcleo
| Função | Descrição |
|---|---|
| `renderizarTudo()` | Re-renderiza lista, alertas, resumo, gráficos, festas e checkboxes |
| `mudarTab(tab)` | Alterna entre abas "amigos" e "festas" |

### 6.2 Amigos
| Função | Descrição |
|---|---|
| `salvarAmigo(e)` | Cria ou atualiza amigo, persiste no localStorage |
| `editarAmigo(id)` | Preenche modal com dados do amigo para edição |
| `excluirAmigo(id)` | Remove amigo com confirmação |
| `renderizarLista()` | Renderiza cards de amigos ordenados |
| `renderizarAlertas()` | Mostra/esconde banner de aniversários próximos |

### 6.3 Festas
| Função | Descrição |
|---|---|
| `salvarFesta(e)` | Cria ou atualiza festa, persiste no localStorage |
| `editarFesta(id)` | Preenche modal com dados da festa para edição |
| `excluirFesta(id)` | Remove festa com confirmação |
| `renderizarFestas()` | Renderiza cards de festas ordenados por data |
| `popularConvidadosCheckbox(selecionados)` | Popula checkboxes de convidados no formulário |

### 6.4 Utilitários
| Função | Descrição |
|---|---|
| `calcularIdade(dataNasc)` | Retorna idade a partir da data de nascimento |
| `diasParaAniversario(dataNasc)` | Retorna dias até o próximo aniversário |
| `formatarData(dataStr)` | Converte `YYYY-MM-DD` para `DD/MM/YYYY` |
| `criarGrafico(canvasId, tipo, labels, dados, cores, titulo)` | Renderiza gráfico via Chart.js |

### 6.5 Modais
| Função | Descrição |
|---|---|
| `abrirModal()` / `fecharModal()` | Abre/fecha modal de amigo |
| `abrirModalFesta()` / `fecharModalFesta()` | Abre/fecha modal de festa |

---

## 7. Service Worker (ws.js)

Estratégia de cache: **Cache First** para assets estáticos (CDNs e arquivos locais).

| Evento | Ação |
|---|---|
| `install` | Pré-cacheia HTML, manifest, Tailwind, Chart.js, Font Awesome |
| `fetch` | Retorna do cache se disponível, senão busca na rede |
| `activate` | Remove caches de versões anteriores |

**Nota**: O app registra `./sw.js`, mas o arquivo real é `ws.js` — corrigir o registro ou renomear o arquivo.

---

## 8. Possíveis Melhorias Futuras

- [ ] Exportar/importar dados (JSON)
- [ ] Notificações push para lembretes de aniversário
- [ ] Tema escuro
- [ ] Upload de foto do amigo
- [ ] Cálculo automático de total de gastos com festas
- [ ] Calendário visual mensal
- [ ] Filtro de amigos por proximidade / estilo
- [ ] Histórico de presentes dados
- [ ] Sincronização em nuvem (Firebase / Supabase)
