# Schemas de extracção — formatos JSON

Schemas detalhados para os artefactos produzidos em cada fase do pipeline.

## Fase 1 — Output do GA4

```json
{
  "metadata": {
    "periodo_inicio": "2026-04-16T00:00:00+00:00",
    "periodo_fim": "2026-04-17T00:00:00+00:00",
    "property_id": "...",
    "data_geracao": "2026-04-17T09:00:00+00:00"
  },
  "top_artigos": [
    {
      "url": "https://www.dnoticias.pt/2026/4/16/exemplo-titulo/",
      "slug": "exemplo-titulo",
      "titulo_ga4": "Título conforme registado pelo GA4",
      "metricas": {
        "pageviews": 12450,
        "users": 8200,
        "engaged_sessions": 5100,
        "avg_engagement_time_s": 142,
        "scroll_depth_avg": 68,
        "bounce_rate": 0.31
      },
      "fontes_trafego": {
        "organic_search": 0.45,
        "social_facebook": 0.32,
        "direct": 0.15,
        "referral": 0.05,
        "outros": 0.03
      },
      "trend_score": 87.3,
      "comparacao_7d": {
        "pageviews_vs_media": 2.4,
        "engagement_vs_media": 1.1
      }
    }
  ],
  "agregados": {
    "total_pageviews": 245000,
    "total_users": 89000,
    "por_categoria": {
      "Política Regional": 78000,
      "Desporto": 56000,
      "...": "..."
    },
    "por_fonte": {
      "organic_search": 0.42,
      "social_facebook": 0.35,
      "direct": 0.18,
      "outros": 0.05
    }
  }
}
```

## Fase 2 — Output da análise semântica

Por artigo:

```json
{
  "url": "https://www.dnoticias.pt/2026/4/16/exemplo-titulo/",
  "data_publicacao": "2026-04-16T08:30:00+00:00",
  "autor": "Nome do Jornalista",
  "extracao": {
    "titulo": "Título exacto do artigo",
    "lead": "Primeiro parágrafo ou subtítulo",
    "tipo_artigo": "noticia",
    "tema_principal": "Obras na ER101 entre Calheta e Ponta do Sol",
    "sub_temas": [
      "infraestruturas rodoviárias",
      "circulação alternativa",
      "investimento público regional"
    ],
    "contexto": "As obras inserem-se no plano plurianual de requalificação da rede viária da RAM. A ER101 é a principal artéria do oeste da Madeira, com tráfego médio diário de cerca de 8.000 veículos.",
    "pessoas": [
      {
        "nome": "Pedro Fino",
        "papel": "Secretário Regional das Infraestruturas",
        "relevancia": "central"
      }
    ],
    "locais": [
      {"nome": "ER101", "tipo": "via_rodoviaria", "regiao": "Madeira"},
      {"nome": "Calheta", "tipo": "concelho", "regiao": "Madeira"},
      {"nome": "Ponta do Sol", "tipo": "concelho", "regiao": "Madeira"}
    ],
    "organizacoes": [
      {"nome": "Vialitoral", "tipo": "empresa_publica"},
      {"nome": "Secretaria Regional das Infraestruturas", "tipo": "orgao_governo_regional"}
    ],
    "categorias_editoriais": ["Madeira", "Sociedade"],
    "etiquetas_sugeridas": [
      "obras-publicas",
      "er101",
      "calheta",
      "ponta-do-sol",
      "infraestruturas"
    ],
    "tom": "informativo",
    "relevancia_regional": 5,
    "angulo_narrativo": "Impacto das obras no quotidiano dos residentes do oeste da Madeira",
    "potencial_followup": "alta",
    "razao_followup": "Tópico de interesse continuado, com cronograma de obras a desenrolar-se ao longo de meses; oportunidade para acompanhar evolução e recolher reacções de utentes da via."
  },
  "notas_extracao": []
}
```

## Fase 3 — Estrutura de dados do relatório

```json
{
  "metadata": {
    "data_relatorio": "2026-04-17",
    "periodo_analisado": "2026-04-16",
    "artigos_analisados": 20,
    "modelos_usados": {
      "extracao": "claude-sonnet-4-6",
      "sintese": "claude-opus-4-7"
    }
  },
  "quantitativo": {
    "top_10": [...],
    "distribuicao_categorias": [...],
    "distribuicao_fontes": [...],
    "comparacao_historica": [...],
    "performance_autores": [...],
    "performance_tipos": [...]
  },
  "qualitativo": {
    "temas_dominantes": [
      {
        "cluster": "Política regional — eleições autárquicas",
        "artigos": ["url1", "url2", "url3"],
        "pageviews_total": 34000,
        "observacoes": "..."
      }
    ],
    "mapa_entidades": {
      "pessoas_top": [{"nome": "...", "frequencia": 8, "categorias": [...]}],
      "locais_top": [{"nome": "...", "frequencia": 12, "tipo": "concelho"}],
      "organizacoes_top": [...]
    },
    "padroes_interesse": [...],
    "lacunas_cobertura": [...],
    "oportunidades_followup": [...],
    "anomalias": [...]
  },
  "deducoes": [
    {
      "id": "ded-001",
      "observacao": "...",
      "interpretacao": "...",
      "confianca": "alta|media|baixa",
      "sugestao": "...",
      "evidencia": ["url1", "metric_x"]
    }
  ]
}
```

## Tipos controlados

### `tipo` em `locais`

- `freguesia` — divisão administrativa mais pequena
- `concelho` — município
- `regiao` — RAM, Continente, Açores
- `cidade` — quando relevante distinguir cidade dentro de concelho
- `pais` — para conteúdo internacional
- `via_rodoviaria` — ER, VR, autoestrada, etc.
- `instituicao_localizada` — hospital, universidade, escola, porto, aeroporto
- `area_natural` — pico, vereda, levada, parque
- `bairro` — divisão informal de cidade

### `tipo` em `organizacoes`

- `governo_regional`
- `orgao_governo_regional`
- `governo_nacional`
- `autarquia`
- `partido_politico`
- `empresa_privada`
- `empresa_publica`
- `instituicao_publica`
- `associacao`
- `clube_desportivo`
- `escola`
- `instituicao_religiosa`
- `sindicato`
- `media`
- `internacional` — UE, ONU, etc.

### `relevancia` em `pessoas`

- `central` — protagonista do artigo, fonte primária citada
- `secundaria` — fonte adicional, mencionada com contexto
- `mencao` — apenas referida, sem desenvolvimento
