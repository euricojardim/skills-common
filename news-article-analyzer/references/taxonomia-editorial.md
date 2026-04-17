# Taxonomia editorial do DIÁRIO

> **NOTA:** este ficheiro deve ser validado e actualizado pelo Eurico/equipa editorial. Os valores abaixo são uma proposta inicial baseada na estrutura típica de jornais regionais portugueses. Substituir pela taxonomia real do CMS antes de pôr em produção.

## Categorias principais

A skill deve usar **exclusivamente** os valores desta lista no campo `categorias_editoriais`. Um artigo pode ter mais que uma categoria (ex.: "Política Regional" + "Madeira").

### Geografia / âmbito

- `Madeira` — temas circunscritos à RAM (default para conteúdo regional)
- `Porto Santo` — temas específicos da ilha de Porto Santo
- `País` — temas nacionais portugueses sem ângulo regional dominante
- `Mundo` — temas internacionais

### Editoriais

- `Política Regional` — Governo Regional, ALRAM, autarquias da Madeira
- `Política Nacional` — AR, Governo, partidos a nível nacional
- `Economia` — empresas, mercados, indicadores económicos
- `Sociedade` — temas sociais, demografia, comportamento
- `Educação` — ensino básico, secundário, superior, formação
- `Saúde` — SESARAM, hospitais, saúde pública, medicina
- `Justiça` — tribunais, processos judiciais, criminalidade
- `Ambiente` — clima, biodiversidade, sustentabilidade, ordenamento
- `Cultura` — artes, espectáculos, património, eventos culturais
- `Desporto` — todas as modalidades, com sub-tags por desporto
- `Turismo` — sector turístico, hotelaria, fluxos
- `Tecnologia` — inovação, digital, ciência aplicada
- `Religião` — Igreja, eventos religiosos, pastoral
- `Obituário` — necrologia, falecimentos com relevância pública
- `Opinião` — colunas, editoriais, artigos de opinião assinados
- `Multimédia` — galerias, vídeo, podcast, infografias

### Sub-categorias de Desporto (mais granulares)

- `Futebol` (Marítimo, Nacional, União, Camacha, etc.)
- `Andebol`
- `Basquetebol`
- `Voleibol`
- `Ténis`
- `Atletismo`
- `Vela`
- `Surf`
- `Rali` (Rali Vinho da Madeira)
- `Outros desportos`

## Tipos de artigo

Lista controlada para o campo `tipo_artigo`:

| Tipo | Definição | Sinais identificadores |
|---|---|---|
| `noticia` | Relato factual de acontecimento recente, estrutura inverted pyramid | Lead com 5W, citações de fontes, sem opinião do autor |
| `breve` | Notícia curta (<200 palavras) sem desenvolvimento | Geralmente sem foto destacada, parágrafo único ou dois |
| `reportagem` | Peça aprofundada com trabalho de terreno, múltiplas fontes | Extensão >800 palavras, narrativa, descrições de cenário |
| `entrevista` | Diálogo Q&A ou narrativa baseada em entrevista a uma figura | Estrutura pergunta-resposta ou citações longas dominantes |
| `opiniao` | Artigo assinado expressando posição do autor | Tom argumentativo, primeira pessoa, ausência de fontes externas |
| `analise` | Interpretação contextualizada de evento ou tendência | Mistura factos com leitura, frequentemente assinada por jornalista sénior |
| `factcheck` | Verificação de afirmação pública | Estrutura "afirmação → verificação → veredicto" |
| `obituario` | Texto biográfico póstumo | Tempos verbais no passado, biografia condensada |
| `editorial` | Posição institucional do jornal, geralmente sem assinatura | Voz colectiva, página de opinião |
| `comunicado` | Reprodução de press release com edição mínima | Linguagem corporativa, sem trabalho jornalístico aparente |

## Etiquetas (tags) — orientação

Tags são livres mas devem seguir convenções:

- **Minúsculas, sem acentos quando ambíguo** — `eleicoes-autarquicas-2025` não `Eleições Autárquicas 2025`
- **Singulares quando aplicável** — `escola` não `escolas`
- **Hierárquicas com hífen** — `futebol-maritimo`, `futebol-nacional`
- **Eventos com ano** — `festa-da-flor-2026`, `rali-vinho-madeira-2026`
- **Pessoas com cargo opcional** — `miguel-albuquerque`, `pedro-calado-camara-funchal`

## Notas para a skill

- Quando um artigo não encaixar claramente em nenhuma categoria, atribuir a mais próxima e sinalizar no campo `notas_extracao` da Fase 2.
- Sempre que o tipo de artigo for ambíguo (ex.: "reportagem ou notícia longa?"), prevalecer a categoria mais conservadora.
- Evitar inflação de relevância — um comunicado da Câmara não é uma reportagem só porque tem 600 palavras.
