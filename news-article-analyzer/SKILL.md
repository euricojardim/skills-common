---
name: news-article-analyzer
description: Analisar artigos noticiosos do DIÁRIO (DNoticias) e cruzá-los com dados do Google Analytics 4 para produzir relatórios editoriais accionáveis. Usar sempre que o utilizador pedir análise de artigos, identificação de temas, extracção de entidades (pessoas, locais, organizações), categorização editorial, deteção de tópicos em tendência (trending), análise de performance de conteúdo, ou geração de relatórios diários/semanais para a redacção. Activar também quando mencionar "Paperclip", "análise de artigos", "trending", "top artigos", "relatório editorial", ou pedir para cruzar dados de GA4 com conteúdo editorial.
---

# News Article Analyzer — DIÁRIO

Análise semântica e quantitativa de artigos noticiosos do DNoticias, cruzando extracção de entidades com métricas do Google Analytics 4 para produzir relatórios editoriais com insights accionáveis.

## Quando usar esta skill

Activar sempre que o utilizador pedir:

- Análise de um conjunto de artigos publicados (identificar temas, entidades, categorias)
- Identificação de tópicos em tendência cruzando conteúdo com analytics
- Relatório editorial diário/semanal para a redacção ou administração
- Detecção de lacunas de cobertura ou oportunidades de follow-up
- Análise de performance de conteúdo por secção, autor, ou tipo de peça

## Princípios fundamentais

1. **Português europeu obrigatório.** Toda a análise, output e relatórios em pt-PT. Usar "motores de pesquisa" em vez de "motores de busca". Quando o utilizador indicar, usar pré-Acordo Ortográfico.
2. **Provenance.** Cada métrica ou afirmação no relatório final deve ser rastreável até à query GA4 ou ao artigo original. Sem isto, o relatório não é defensável editorialmente.
3. **Contexto regional primeiro.** O DIÁRIO é um jornal regional madeirense. Distinguir explicitamente conteúdo regional (Madeira/Porto Santo), nacional (Portugal), e internacional. A relevância regional pesa mais que volume bruto.
4. **Insights accionáveis, não descritivos.** Um relatório que diz "a peça X teve 12 mil visitas" é inútil. Um que diz "a peça X teve 12 mil visitas e o tema relacionado Y está sub-coberto — sugerimos follow-up" é accionável.
5. **Honestidade epistémica.** Quando os dados são insuficientes para uma conclusão, dizer isso. Não inventar padrões.

## Pipeline de análise

A skill executa três fases sequenciais. Cada fase produz um artefacto que alimenta a próxima.

### Fase 1 — Recolha de dados analíticos (GA4)

**Input:** período de análise (default: últimas 24h), property GA4 do dnoticias.pt.

**Tools usadas:** GA4 MCP server (`run_report`, `run_realtime_report`).

**Queries a executar:**

1. **Top páginas por pageviews** — dimensões: `pagePath`, `pageTitle`; métricas: `screenPageViews`, `totalUsers`, `userEngagementDuration`, `engagedSessions`. Filtrar para URLs de artigos (excluir homepage, páginas de secção, etc.).
2. **Distribuição por fonte de tráfego** — dimensões: `pagePath`, `sessionSource`, `sessionMedium`. Para identificar dependência de Facebook vs. pesquisa orgânica vs. directo.
3. **Engagement profundo** — métricas: `averageSessionDuration`, `eventCount` para `scroll` ou eventos custom de leitura.
4. **Comparação histórica** — mesma query para os 7 dias anteriores, para calcular médias móveis e detectar anomalias.

**Cálculo do `trend_score`:**

```
trend_score = (0.4 × normalize(pageviews))
            + (0.3 × normalize(engagement_time))
            + (0.2 × normalize(velocity_pageviews_per_hour))
            + (0.1 × normalize(scroll_depth))
```

`velocity` privilegia artigos recentes; sem este factor, evergreen content domina sempre.

**Output:** lista ordenada de top N artigos (default N=20) com URL, título, métricas e trend_score.

Ver `references/schema-extracao.md` para o formato JSON exacto.

### Fase 2 — Análise semântica dos artigos

**Input:** lista de URLs da Fase 1.

**Para cada artigo:**

1. **Fetch do conteúdo** — via endpoint do CMS Django (preferencial) ou scraping do HTML público. Extrair título, lead, corpo, autor, data de publicação, categorias e tags já atribuídas pelo CMS.

2. **Extracção semântica estruturada** — produzir o seguinte JSON para cada artigo:

```json
{
  "url": "https://www.dnoticias.pt/...",
  "titulo": "...",
  "tipo_artigo": "noticia | reportagem | opiniao | entrevista | analise | breve | factcheck",
  "tema_principal": "frase curta (5-10 palavras)",
  "sub_temas": ["..."],
  "contexto": "2-3 frases que situem o artigo num enquadramento mais amplo",
  "pessoas": [
    {"nome": "Miguel Albuquerque", "papel": "Presidente do Governo Regional", "relevancia": "central|secundaria|mencao"}
  ],
  "locais": [
    {"nome": "Câmara de Lobos", "tipo": "concelho", "regiao": "Madeira"}
  ],
  "organizacoes": [
    {"nome": "PSD-Madeira", "tipo": "partido_politico"}
  ],
  "categorias_editoriais": ["Política Regional", "Madeira"],
  "etiquetas_sugeridas": ["eleições", "autarquia", "campanha"],
  "tom": "informativo | opiniao | investigacao | breaking | celebracao",
  "relevancia_regional": 1,
  "angulo_narrativo": "frase que descreve o ângulo escolhido pelo jornalista",
  "potencial_followup": "alta|media|baixa",
  "razao_followup": "..."
}
```

`relevancia_regional` numa escala 1-5: 1 = puramente nacional/internacional sem ângulo regional; 5 = exclusivamente regional ou com forte ângulo regional adicionado.

**Critérios de classificação:**

- Para `tipo_artigo`, ver `references/schema-extracao.md` com definições e exemplos de cada tipo.
- Para `categorias_editoriais`, usar **exclusivamente** valores da taxonomia em `references/taxonomia-editorial.md`. Não inventar categorias.
- Para `locais`, validar contra `references/geografia-madeira.md` para a hierarquia administrativa correcta da RAM.
- Para `pessoas`, identificar papel/cargo no momento da publicação (ex.: "Presidente da Câmara do Funchal" — não "ex-Presidente").

### Fase 3 — Síntese e relatório

**Input:** dados das Fases 1 e 2.

**Análises a produzir:**

#### Quantitativas

- **Top 10 artigos** por trend_score com tabela de métricas
- **Distribuição por categoria editorial** — onde está a audiência hoje?
- **Distribuição por fonte de tráfego** — agregada e por categoria (ex.: Política depende muito de Facebook? Desporto vem maioritariamente de pesquisa?)
- **Comparação com média móvel 7 dias** — categorias que tiveram pico ou queda atípica
- **Performance por autor** — top autores do período por engagement médio (não apenas volume)
- **Performance por tipo de artigo** — reportagens vs. notícias breves vs. opinião

#### Qualitativas

- **Temas dominantes do dia** — agrupamento dos `tema_principal` extraídos, com 3-5 clusters
- **Mapa de entidades** — pessoas e locais mais mencionados, com frequência
- **Padrões de interesse** — observações como "artigos com locais do interior da Madeira tiveram 2× mais engagement médio" ou "peças de opinião sobre saúde mantiveram-se no top mais tempo que notícias breves"
- **Lacunas de cobertura** — temas com tráfego elevado em pesquisa (via GA4 search terms se disponível) mas pouca cobertura editorial recente
- **Oportunidades de follow-up** — peças com `potencial_followup = alta` e métricas a corroborar
- **Anomalias** — qualquer padrão que se desvie significativamente da norma histórica

#### Deduções

A última secção do relatório deve conter 3-5 deduções editoriais explícitas. Cada dedução tem:

- **Observação** (o quê) — facto baseado nos dados
- **Interpretação** (porquê) — leitura plausível com nível de confiança
- **Sugestão** (e agora?) — acção concreta para a redacção

Exemplo:

> **Observação:** A peça sobre obras na ER101 (Calheta) gerou 8.400 pageviews e 3min42s de engagement médio, valores 3× acima da média de notícias regionais.
>
> **Interpretação** (confiança média): O tema toca directamente quotidiano dos leitores do oeste da Madeira. O engagement profundo sugere interesse genuíno, não tráfego acidental de social media.
>
> **Sugestão:** Considerar peça de seguimento com cronograma das obras e impacto nas alternativas de circulação. Possível ângulo adicional: declarações da Câmara da Calheta.

Ver `references/formato-relatorio.md` para a estrutura completa do relatório (Markdown + HTML + JSON).

## Formatos de output

A skill produz três artefactos em paralelo:

- **`relatorio_YYYY-MM-DD.md`** — versão para terminal e arquivo Git
- **`relatorio_YYYY-MM-DD.html`** — versão para envio por email à direcção editorial
- **`dados_YYYY-MM-DD.json`** — dados estruturados para alimentar dashboards Metabase futuros

## Boas práticas operacionais

- **Não inventar métricas.** Se a Data API não retorna um valor, marcar como "indisponível" — nunca interpolar ou estimar sem o dizer.
- **Validar URLs.** Confirmar que cada URL devolvida pelo GA4 corresponde a um artigo activo no CMS antes de tentar fetch. URLs orfãs (artigos eliminados, redirecionamentos) devem ser sinalizadas no relatório.
- **Limites de chamada ao Claude.** Para análise semântica, processar artigos em batches de 5-10 para controlar custos e tempo. Usar Sonnet por default; reservar Opus para peças longas (>3000 palavras) ou de análise complexa.
- **Cache.** Se o mesmo artigo aparecer em execuções consecutivas (ex.: evergreen na semana), reutilizar a análise semântica anterior — só re-correr a Fase 1 (GA4).
- **Privacidade.** Não persistir dados pessoais identificáveis de leitores que possam vir do GA4 (ex.: utilizadores específicos). Apenas agregados.

## Quando NÃO usar esta skill

- Análise de artigos individuais sem componente analítica (use análise simples conversacional)
- Pesquisa académica de longo prazo sobre conteúdo (precisa de outra metodologia)
- Análise de redes sociais isoladamente (use ferramentas dedicadas)
- Geração de novos artigos ou edição (não é o âmbito)

## Ficheiros de referência

- `references/taxonomia-editorial.md` — categorias e secções oficiais do DIÁRIO
- `references/geografia-madeira.md` — hierarquia administrativa da RAM e topónimos
- `references/schema-extracao.md` — schemas JSON detalhados para inputs e outputs
- `references/formato-relatorio.md` — estrutura e templates dos relatórios finais

Consultar estes ficheiros quando necessário, não carregar todos por defeito.
