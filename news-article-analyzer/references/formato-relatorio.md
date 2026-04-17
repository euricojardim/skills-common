# Formato do relatório editorial — DIÁRIO

Estrutura padrão dos três artefactos de output da Fase 3.

## Markdown — `relatorio_YYYY-MM-DD.md`

```markdown
# Relatório Editorial DIÁRIO — {data}

> Período analisado: {periodo} · Artigos: {n_artigos} · Gerado: {timestamp}

## Resumo executivo

{2-3 parágrafos com as conclusões mais salientes do dia. Esta é a única secção
que muitos leitores vão ler — tem de ser auto-contida.}

---

## 1. Top 10 artigos do período

| # | Título | Categoria | Pageviews | Engagement | Trend Score |
|---|--------|-----------|-----------|------------|-------------|
| 1 | [...](...) | ... | 12.450 | 2m22s | 87,3 |
| 2 | ... | ... | ... | ... | ... |

## 2. Distribuição da audiência

### Por categoria editorial

{Tabela ou descrição em prosa das categorias com mais tracção, comparando com
média móvel dos 7 dias.}

### Por fonte de tráfego

{Análise da dependência de Facebook vs. pesquisa orgânica vs. directo.
Sinalizar concentrações preocupantes — ex.: "75% do tráfego de Política
Regional veio de Facebook hoje, contra 45% de média semanal".}

## 3. Temas dominantes

{3-5 clusters temáticos identificados, cada um com:
- Nome do cluster
- Artigos que o compõem
- Volume agregado de tráfego
- Breve observação sobre o porquê deste tema ter ganho tracção}

## 4. Mapa de entidades

### Pessoas mais mencionadas
{Top 10, com cargo e número de artigos}

### Locais mais referidos
{Top 10, com tipo administrativo}

### Organizações em destaque
{Top 5}

## 5. Padrões de interesse

{Observações qualitativas baseadas no cruzamento de extracção semântica com
métricas. Cada observação como bullet de 2-3 frases.}

## 6. Lacunas e oportunidades

### Sub-cobertura
{Temas com sinal de procura (pesquisas, social) mas pouca cobertura editorial.}

### Follow-ups sugeridos
{Lista de artigos com `potencial_followup = alta`, com razão.}

## 7. Anomalias

{Padrões fora da norma histórica, positivos ou negativos.}

---

## Deduções editoriais

### {Título da dedução 1}

**Observação:** {facto}

**Interpretação** (confiança {alta|média|baixa}): {leitura}

**Sugestão:** {acção concreta}

**Evidência:** {links ou métricas}

---

### {Título da dedução 2}
...

---

*Relatório gerado automaticamente pela skill `news-article-analyzer`.
Todas as métricas são rastreáveis aos dados de origem (GA4 + CMS).
Para questões metodológicas, consultar [...].*
```

## HTML — `relatorio_YYYY-MM-DD.html`

Mesmo conteúdo do Markdown, com:

- Layout responsivo simples (max-width 800px, fonte serifada para corpo)
- Tabelas com alternância de cor (zebra)
- Trend scores e pageviews destacados visualmente
- Links clicáveis para os artigos
- Cabeçalho com logo do DIÁRIO (placeholder se ainda não disponível)
- Pé de página com email de contacto da equipa que mantém o Paperclip

Estilo visual sóbrio — **não** dashboard com cores fortes; é um relatório editorial. Tipografia importa mais que infografia.

## JSON — `dados_YYYY-MM-DD.json`

Conforme definido em `schema-extracao.md`, secção "Fase 3". Este ficheiro é a fonte de verdade — Markdown e HTML são derivados dele. Permite:

- Reconstruir o relatório se o template mudar
- Alimentar dashboards Metabase futuros
- Análise histórica longitudinal

## Convenções editoriais para o relatório

- **Tom institucional, não conversacional.** "O artigo X registou..." em vez de "Vê só, o artigo X teve..."
- **Números formatados em pt-PT.** Separador de milhares com ponto, decimal com vírgula. "12.450 pageviews" e "trend_score 87,3".
- **Datas no formato pt-PT.** "16 de Abril de 2026" no corpo; ISO 8601 (`2026-04-16`) em metadados.
- **Citar incertezas explicitamente.** "Os dados sugerem..." em vez de "Os dados provam..." quando a confiança é média ou baixa.
- **Evitar jargão técnico desnecessário.** O relatório é lido pela direcção editorial, não pela equipa de IT. "Pageviews" e "engagement" podem ficar; "bounce rate" deve ser traduzido para "taxa de saída". "trend_score" pode ficar mas com nota de rodapé na primeira ocorrência.
- **Anonimizar leitores.** Nunca incluir métricas que possam identificar utilizadores individuais.

## Distribuição

O relatório deve poder ser:

1. Lido no terminal (Markdown puro)
2. Enviado por email à direcção editorial (HTML)
3. Versionado em Git (Markdown + JSON)
4. Ingerido por dashboards futuros (JSON)

A skill produz os três ficheiros; a distribuição (envio de email, push para repo, etc.) é responsabilidade do orquestrador (OpenClaw, n8n, ou cron).
